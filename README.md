def topo_sort(jobs):
    """
    Topologically sort jobs where:
      - After means this job depends on others (A → this)
      - Rel means others depend on this job (this → R)
    """
    id2job = {job["JobID"]: job for job in jobs}
    graph = defaultdict(list)
    indegree = {jid: 0 for jid in id2job}

    for job in jobs:
        # Handle AFTER: for each 'A' in After, edge A → this
        for after_id in job["After"]:
            if after_id not in id2job:
                print(f"⚠️  Warning: {job['JobID']} has unknown After {after_id}", file=sys.stderr)
                continue
            graph[after_id].append(job["JobID"])
            indegree[job["JobID"]] += 1
        # Handle REL: for each 'R' in Rel, edge this → R
        for rel_id in job["Rel"]:
            if rel_id not in id2job:
                print(f"⚠️  Warning: {job['JobID']} has unknown Rel {rel_id}", file=sys.stderr)
                continue
            graph[job["JobID"]].append(rel_id)
            indegree[rel_id] += 1

    # Kahn's algorithm
    queue = deque([jid for jid, deg in indegree.items() if deg == 0])
    ordered = []
    while queue:
        u = queue.popleft()
        ordered.append(u)
        for v in graph[u]:
            indegree[v] -= 1
            if indegree[v] == 0:
                queue.append(v)

    if len(ordered) != len(id2job):
        raise RuntimeError("Cycle detected in dependency graph")

    return [id2job[jid] for jid in ordered]
