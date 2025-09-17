import pandas as pd
import re
from collections import defaultdict, deque

def parse_lineage(lineage_text, output_csv):
    graph = defaultdict(list)
    indegree = defaultdict(int)
    nodes = set()

    # Parse dependencies
    for line in lineage_text.strip().splitlines():
        if "->" not in line or "Lineage" in line or "Main Job ID" in line:
            continue
        lhs, rhs = line.split("->")
        lhs = lhs.strip()
        rhs_parts = re.split(r"[_ ,]", rhs.strip())
        rhs_parts = [r for r in rhs_parts if r]
        for rhs in rhs_parts:
            graph[lhs].append(rhs)
            indegree[rhs] += 1
            nodes.add(lhs)
            nodes.add(rhs)

    # Initialize queue with jobs having no dependencies before them
    queue = deque([n for n in nodes if indegree[n] == 0])
    ordered = []

    while queue:
        node = queue.popleft()
        ordered.append(node)
        for neighbor in graph[node]:
            indegree[neighbor] -= 1
            if indegree[neighbor] == 0:
                queue.append(neighbor)

    # Check for cycle (if not all nodes covered)
    if len(ordered) < len(nodes):
        print("⚠️ Warning: Cycle detected in dependencies. Some jobs may depend on each other.")
        # Append remaining nodes (to not lose them)
        remaining = [n for n in nodes if n not in ordered]
        ordered.extend(remaining)

    # Build DataFrame
    df = pd.DataFrame({
        "Step Number": range(1, len(ordered) + 1),
        "Job ID": ordered
    })

    df.to_csv(output_csv, index=False)
    print(f"Saved sequence to {output_csv}")
    return df
lineage_text = """

"""
df = parse_lineage(lineage_text, "job_sequence_resolved3.csv")
