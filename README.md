import re
import csv
from collections import defaultdict, deque

INPUT = "jobs.txt"           # your raw file
OUTPUT = "wrapper_input.csv" # the “wrapper” input

def parse_blocks(path):
    """Yield lists of lines, one per job‐block."""
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

def extract(block):
    """
    Turn one block into a dict with:
      - EventName, JobType, JobID, ScriptName (full path),
      - deps: a Python list of dependency IDs
    """
    info = {
        "EventName":    "",
        "JobType":      "",
        "JobID":        "",
        "ScriptName":   "",
        "deps":         []
    }
    # first line: job type & ID
    m = re.match(r"^(AIX_JOB|LINUX_JOB)\s+(\S+)", block[0])
    info["JobType"], info["JobID"] = m.groups()
    for line in block:
        if line.startswith("SCRIPTNAME"):
            script = line.split(None,1)[1]
            info["ScriptName"] = script
            info["EventName"] = script.split("/")[-1]
        elif line.startswith("AFTER") or line.startswith("REL"):
            # grab the single token after
            info["deps"].append(line.split()[1])
    return info

def topo_sort(jobs):
    """
    Kahn’s algorithm: input is list of job‐dicts with .JobID and .deps
    Returns list of JobIDs in valid order, or raises if cycle detected.
    """
    # map id→job, build graph edges and indegree
    jobs_by_id = {j["JobID"]: j for j in jobs}
    graph = defaultdict(list)      # parent → list of children
    indegree = defaultdict(int)    # child → # of unmet deps

    # initialize
    for job in jobs:
        jid = job["JobID"]
        indegree.setdefault(jid, 0)
    # populate
    for job in jobs:
        for dep in job["deps"]:
            if dep not in jobs_by_id:
                # unknown dependency — you can warn or ignore
                continue
            graph[dep].append(job["JobID"])
            indegree[job["JobID"]] += 1

    # start with everything that has zero indegree
    queue = deque([jid for jid, d in indegree.items() if d == 0])
    sorted_list = []

    while queue:
        cur = queue.popleft()
        sorted_list.append(cur)
        for child in graph[cur]:
            indegree[child] -= 1
            if indegree[child] == 0:
                queue.append(child)

    if len(sorted_list) != len(jobs_by_id):
        raise RuntimeError("Dependency cycle detected or missing jobs")

    return sorted_list

def main():
    # 1. parse & extract
    raw_blocks = list(parse_blocks(INPUT))
    jobs        = [extract(b) for b in raw_blocks]

    # 2. topological sort
    order = topo_sort(jobs)

    # 3. write CSV in that order
    with open(OUTPUT, "w", newline="") as out:
        writer = csv.writer(out)
        writer.writerow(["EventName","JobType","JobID","ScriptName","Dependencies"])
        for jid in order:
            job = next(j for j in jobs if j["JobID"] == jid)
            deps_str = ",".join(job["deps"])
            writer.writerow([
                job["EventName"],
                job["JobType"],
                job["JobID"],
                job["ScriptName"],
                deps_str
            ])

    print(f"→ Wrote {len(order)} jobs (in dependency order) to {OUTPUT}")

if __name__ == "__main__":
    main()
