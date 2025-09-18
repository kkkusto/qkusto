#!/usr/bin/env python3
"""
lineage_to_sequence.py

Parse lineage text blocks like:

  
    ...

and emit a CSV with rows: Step No.,Job ID,Main Job

Rules implemented:
- Respect dependencies A -> B (A before B).
- If multiple jobs appear on the same side separated by underscores and ALL
  tokens are job-like (e.g., RFT\d{5}), they are treated as a group that shares
  the same step number in the output (e.g., "RFT62101_RFT62171").
- Underscores inside a single job name (like "RFT62125_Talend") do NOT split.
- Deduplicate jobs and keep a stable, LHS-first biased order when ties exist.
- The "Main Job ID" is expected to be the terminal job; the sort ensures it
  appears at the end when dependencies are complete.
- Multiple lineage blocks in a single file/STDIN are supported.

Usage:
    python lineage_to_sequence.py -i lineage.txt -o job_execution_sequences.txt --append
    # or read from STDIN:
    cat lineage.txt | python lineage_to_sequence.py -o job_execution_sequences.txt --append
"""

from __future__ import annotations
import argparse
import os
import re
import sys
from collections import defaultdict, deque
from typing import Dict, Iterable, List, Tuple, Set

MAIN_RE = re.compile(r"Main\s*Job\s*ID[:\s]*([A-Za-z0-9_]+)", re.IGNORECASE)
EDGE_RE = re.compile(r"([A-Za-z0-9_]+)\s*->\s*([A-Za-z0-9_]+)")
# Group pattern: one or more RFT##### joined with underscores
GROUP_RE = re.compile(r"^(?:RFT\d{5})(?:_RFT\d{5})+$")
# Individual job id pattern
JOB_ID_RE = re.compile(r"^RFT\d{5}(?:_[A-Za-z0-9]+)?$")

def clean_token(s: str) -> str:
    s = s.strip().strip('"').strip("'").strip()
    # remove stray single-character punctuation-only tokens
    if len(s) == 1 and s in {',', ':', '*', '?'}:
        return ""
    return s

def split_compound(job: str) -> List[str]:
    """
    Return a list of job IDs.
    Only split when ALL parts are strict RFT#####: "RFT12345_RFT54321".
    Do not split single job ids that merely contain underscores (e.g., "..._Talend").
    """
    job = clean_token(job)
    if not job:
        return []
    if GROUP_RE.match(job):
        return job.split("_")
    return [job]

def normalize_line(line: str) -> str:
    # remove invisible quoting/extra commas
    return line.strip().replace("“", '"').replace("”", '"').replace("’", "'")

def parse_blocks(text: str) -> List[Tuple[str, List[Tuple[str, str]]]]:
    """
    Parse the input text into a list of (main_job, edges) blocks.
    edges is a list of (lhs, rhs) raw strings (possibly group tokens present).
    """
    blocks: List[Tuple[str, List[Tuple[str, str]]]] = []
    current_main: str | None = None
    current_edges: List[Tuple[str, str]] = []

    for raw in text.splitlines():
        line = normalize_line(raw)
        if not line:
            continue

        m = MAIN_RE.search(line)
        if m:
            # flush previous
            if current_main and current_edges:
                blocks.append((current_main, current_edges))
            current_main = clean_token(m.group(1))
            current_edges = []
            continue

        if "->" in line:
            m2 = EDGE_RE.search(line)
            if m2:
                lhs = clean_token(m2.group(1))
                rhs = clean_token(m2.group(2))
                if lhs and rhs:
                    current_edges.append((lhs, rhs))

    # flush last
    if current_main and current_edges:
        blocks.append((current_main, current_edges))

    return blocks

def build_graph(edges: List[Tuple[str, str]]) -> Tuple[Dict[str, Set[str]], Dict[str, int], Dict[str, List[str]], Dict[str, int]]:
    """
    Build a DAG from edges that may contain group tokens.
    Returns:
        graph: adjacency list (node -> set(neighbors))
        indeg: indegree map
        group_members: mapping of group-node-key -> list of job ids OR single-node -> [job]
        first_seen: stable ordering index for nodes
    Nodes are either:
        - single job ids (e.g., "RFT62125_Talend")
        - group nodes with key like "GROUP:RFT62101|RFT62171"
    """
    graph: Dict[str, Set[str]] = defaultdict(set)
    indeg: Dict[str, int] = defaultdict(int)
    group_members: Dict[str, List[str]] = {}
    first_seen: Dict[str, int] = {}
    order = 0

    def node_for_side(token: str) -> str:
        nonlocal order
        parts = split_compound(token)
        if not parts:
            return ""
        if len(parts) == 1:
            key = parts[0]
            if key not in group_members:
                group_members[key] = [key]
                if key not in first_seen:
                    first_seen[key] = order; order += 1
            return key
        else:
            gkey = "GROUP:" + "|".join(parts)
            if gkey not in group_members:
                group_members[gkey] = parts[:]  # members
                if gkey not in first_seen:
                    first_seen[gkey] = order; order += 1
            return gkey

    # collect nodes + edges
    for lhs_raw, rhs_raw in edges:
        lhs_key = node_for_side(lhs_raw)
        rhs_key = node_for_side(rhs_raw)
        if not lhs_key or not rhs_key:
            continue
        if rhs_key not in graph[lhs_key]:
            graph[lhs_key].add(rhs_key)
            indeg[rhs_key] += 1
            # ensure nodes exist in indeg map
            indeg.setdefault(lhs_key, indeg.get(lhs_key, 0))
            indeg.setdefault(rhs_key, indeg.get(rhs_key, 0))
            # give first_seen for nodes touched only as RHS
            first_seen.setdefault(rhs_key, len(first_seen))

    # ensure isolated nodes are present in indeg
    for n in group_members.keys():
        indeg.setdefault(n, indeg.get(n, 0))

    return graph, indeg, group_members, first_seen

def topo_steps(graph: Dict[str, Set[str]], indeg: Dict[str, int], group_members: Dict[str, List[str]], first_seen: Dict[str, int]) -> List[str]:
    """
    Kahn's algorithm with stable ordering by `first_seen`.
    Returns a list of "output tokens", where group nodes are rendered as
    underscore-joined members (e.g., "RFT62101_RFT62171").
    """
    # Priority by first_seen
    q = deque(sorted((n for n, d in indeg.items() if d == 0), key=lambda n: first_seen.get(n, 10**9)))
    result: List[str] = []
    visited: Set[str] = set()

    while q:
        n = q.popleft()
        if n in visited:
            continue
        visited.add(n)

        members = group_members.get(n, [n])
        token = "_".join(members) if n.startswith("GROUP:") else members[0]
        result.append(token)

        for m in sorted(graph.get(n, []), key=lambda x: first_seen.get(x, 10**9)):
            indeg[m] -= 1
            if indeg[m] == 0:
                q.append(m)

    # If there were cycles (shouldn't be), append remaining nodes in stable order
    remaining = [n for n in group_members.keys() if n not in visited]
    for n in sorted(remaining, key=lambda x: first_seen.get(x, 10**9)):
        members = group_members.get(n, [n])
        token = "_".join(members) if n.startswith("GROUP:") else members[0]
        if token not in result:
            result.append(token)

    return result

def block_to_steps(edges: List[Tuple[str, str]], main_job: str) -> List[Tuple[int, str]]:
    graph, indeg, groups, first_seen = build_graph(edges)
    order_tokens = topo_steps(graph, indeg, groups, first_seen)

    # move main_job to the end if it appears (while preserving relative order of others)
    if main_job in order_tokens:
        order_tokens = [t for t in order_tokens if t != main_job] + [main_job]
    else:
        # if main_job wasn't part of the edges, append it for completeness
        order_tokens.append(main_job)

    # assign step numbers; groups already condensed into a single token
    steps = [(i + 1, token) for i, token in enumerate(order_tokens)]
    return steps

def write_csv_rows(rows: Iterable[Tuple[int, str, str]], out_path: str, append: bool) -> None:
    exists = os.path.exists(out_path)
    mode = "a" if append else "w"
    with open(out_path, mode, encoding="utf-8") as f:
        if not append or not exists:
            f.write("Step No.,Job ID,Main Job\n")
        for step, job, main in rows:
            f.write(f"{step},{job},{main}\n")

def main():
    ap = argparse.ArgumentParser(description="Convert lineage text to step-numbered sequences.")
    ap.add_argument("-i", "--input", help="Input text file (defaults to STDIN).")
    ap.add_argument("-o", "--output", default="job_execution_sequences.txt", help="Output CSV-like text path.")
    ap.add_argument("--append", action="store_true", help="Append to output instead of overwriting.")
    args = ap.parse_args()

    if args.input:
        with open(args.input, "r", encoding="utf-8") as fh:
            text = fh.read()
    else:
        text = sys.stdin.read()

    blocks = parse_blocks(text)
    if not blocks:
        print("No lineage blocks found.", file=sys.stderr)
        sys.exit(1)

    out_rows: List[Tuple[int, str, str]] = []
    for main_job, edges in blocks:
        steps = block_to_steps(edges, main_job)
        out_rows.extend([(s, j, main_job) for s, j in steps])

    write_csv_rows(out_rows, args.output, args.append)

if __name__ == "__main__":
    main()
    
