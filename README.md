import re
import csv
import sys
from collections import defaultdict, deque

INPUT  = "jobs.txt"
OUTPUT = "wrapper_input.csv"

def parse_blocks(path):
    with open(path) as f:
        buf = []
        for raw in f:
            line = raw.rstrip()
            if re.match(r"^(AIX_JOB|LINUX_JOB)\b", line):
                buf = [line]
            elif buf:
                buf.append(line)
                if line == "ENDJOB":
                    yield buf
                    buf = []

def split_parenthesized(s):
    return [tok.strip()
            for tok in s.strip().lstrip("(").rstrip(")").split(",")
            if tok.strip()]

def extract(block):
    info = {"EventName": "", "JobType": "", "JobID": "",
            "ScriptName": "", "Dependencies": ""}
    deps = []
    # states for multi-line collection
    collecting_deps = False
    deps_buf = ""
    collecting_script = False
    script_buf = ""

    # first line → JobType, JobID
    m = re.match(r"^(AIX_JOB|LINUX_JOB)\s+(\S+)", block[0])
    if not m:
        raise RuntimeError("Malformed header: " + block[0])
    info["JobType"], info["JobID"] = m.groups()

    for line in block[1:]:
        # —— SCRIPTNAME (multi-line safe) ——
        if collecting_script:
            part = line.strip().lstrip("+")
            script_buf += part
            if not line.strip().endswith("+"):
                info["ScriptName"] = script_buf
                info["EventName"]  = script_buf.split("/")[-1]
                collecting_script = False
            continue

        if line.startswith("SCRIPTNAME"):
            tail = line.split(None,1)[1]
            if tail.endswith("+"):
                collecting_script = True
                script_buf = tail.rstrip("+")
            else:
                info["ScriptName"] = tail
                info["EventName"]  = tail.split("/")[-1]
            continue

        # —— DEPENDENCIES (AFTER & REL, multi-line safe) ——
        if collecting_deps:
            part = line.strip().lstrip("+")
            deps_buf += part
            if ")" in line:
                deps += split_parenthesized(deps_buf)
                collecting_deps = False
            continue

        if line.startswith(("AFTER","REL")):
            keyword, rest = line.split(None,1)
            rest = rest.strip()
            if rest.startswith("("):
                collecting_deps = True
                deps_buf = rest.lstrip("+")
                if ")" in rest:
                    collecting_deps = False
                    deps += split_parenthesized(deps_buf)
            else:
                deps.append(rest)
            continue

    info["Dependencies"] = deps  # store as a list for topo-sort
    return info

def topo_sort(jobs):
    # build mapping JobID→job and graph
    id2job = {j["JobID"]: j for j in jobs}
    graph = defaultdict(list)   # dep → [job, job, ...]
    indegree = {jid:0 for jid in id2job}

    # populate edges
    for j in jobs:
        for dep in j["Dependencies"]:
            if dep not in indegree:
                # missing job: warn but ignore
                print(f"⚠️  Warning: {j['JobID']} depends on unknown {dep}", file=sys.stderr)
                continue
            graph[dep].append(j["JobID"])
            indegree[j["JobID"]] += 1

    # Kahn’s algorithm
    queue = deque([jid for jid,deg in indegree.items() if deg==0])
    ordered = []
    while queue:
        u = queue.popleft()
        ordered.append(u)
        for v in graph[u]:
            indegree[v] -= 1
            if indegree[v] == 0:
                queue.append(v)

    if len(ordered) != len(id2job):
        raise RuntimeError("Cycle detected or missing nodes in dependency graph")

    # map back to job dicts
    return [id2job[jid] for jid in ordered]

def main():
    blocks = list(parse_blocks(INPUT))
    jobs   = [extract(b) for b in blocks]
    sorted_jobs = topo_sort(jobs)

    # write CSV
    with open(OUTPUT, "w", newline="") as csvfile:
        writer = csv.DictWriter(
            csvfile,
            fieldnames=["EventName","JobType","JobID","ScriptName","Dependencies"]
        )
        writer.writeheader()
        for j in sorted_jobs:
            row = {
                "EventName":  j["EventName"],
                "JobType":    j["JobType"],
                "JobID":      j["JobID"],
                "ScriptName": j["ScriptName"],
                # join back dependencies into comma-list:
                "Dependencies": ",".join(j["Dependencies"])
            }
            writer.writerow(row)

    print(f"✔ Wrote {len(sorted_jobs)} rows (topologically sorted) to {OUTPUT}")

if __name__ == "__main__":
    main()
