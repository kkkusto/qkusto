import pandas as pd
import re
from collections import defaultdict, deque

def parse_lineage(lineage_text, output_csv):
    graph = defaultdict(list)
    indegree = defaultdict(int)
    nodes = set()
    appearance_order = {}  # track first appearance order
    order_counter = 0

    # Parse dependencies in order
    for line in lineage_text.strip().splitlines():
        if "->" not in line or "Lineage" in line or "Main Job ID" in line:
            continue
        lhs, rhs = line.split("->")
        lhs = lhs.strip()
        rhs_parts = re.split(r"[_ ,]", rhs.strip())
        rhs_parts = [r for r in rhs_parts if r]

        if lhs not in appearance_order:
            appearance_order[lhs] = order_counter
            order_counter += 1

        for rhs in rhs_parts:
            graph[lhs].append(rhs)
            indegree[rhs] += 1
            nodes.add(lhs)
            nodes.add(rhs)
            if rhs not in appearance_order:
                appearance_order[rhs] = order_counter
                order_counter += 1

    # Initialize queue with jobs that have no incoming edges
    queue = deque(sorted([n for n in nodes if indegree[n] == 0],
                         key=lambda x: appearance_order[x]))
    ordered = []

    while queue:
        node = queue.popleft()
        ordered.append(node)

        for neighbor in sorted(graph[node], key=lambda x: appearance_order[x]):
            indegree[neighbor] -= 1
            if indegree[neighbor] == 0:
                queue.append(neighbor)

    # Detect cycles
    if len(ordered) < len(nodes):
        print("⚠️ Warning: Cycle detected in dependencies.")
        # Add the missing nodes in original order
        for n in sorted(nodes, key=lambda x: appearance_order[x]):
            if n not in ordered:
                ordered.append(n)

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
