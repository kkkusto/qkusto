# ---- DFS-based topo sort with robust handling ----
def topo_sort_dfs(jobs):
    """
    Topological sort using DFS.
    Handles missing After/Rel targets with warnings.
    """
    id2job = {job["JobID"]: job for job in jobs}
    graph = defaultdict(list)  # adjacency list

    for job in jobs:
        jid = job["JobID"]
        for after_id in job["After"]:
            if after_id in id2job:
                graph[after_id].append(jid)
            else:
                print(f"⚠️  Warning: {jid} has unknown After {after_id}", file=sys.stderr)
        for rel_id in job["Rel"]:
            if rel_id in id2job:
                graph[jid].append(rel_id)
            else:
                print(f"⚠️  Warning: {jid} has unknown Rel {rel_id}", file=sys.stderr)

    # DFS topo sort
    visited = {}
    ordering = []
    def visit(node):
        if node in visited:
            if visited[node] == 1:
                raise RuntimeError(f"Cycle detected at job {node}")
            return
        visited[node] = 1
        for nbr in graph[node]:
            visit(nbr)
        visited[node] = 2
        ordering.append(node)

    # only traverse jobs we have data for
    for jid in id2job:
        if jid not in visited:
            visit(jid)

    ordering.reverse()  # because post-order DFS
    return [id2job[jid] for jid in ordering]
