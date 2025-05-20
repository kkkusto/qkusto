import re
import csv

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
    """Turn one block into a dict with the five fields we need."""
    info = {
        "EventName": "",
        "JobType":  "",
        "JobID":    "",
        "ScriptName": "",
        "Dependencies": "",
    }
    deps = []
    # first line holds the type & ID
    m = re.match(r"^(AIX_JOB|LINUX_JOB)\s+(\S+)", block[0])
    info["JobType"], info["JobID"] = m.groups()
    for line in block:
        if line.startswith("SCRIPTNAME"):
            # grab everything after the keyword
            script = line.split(None,1)[1]
            info["ScriptName"] = script
            # event name is just the filename
            info["EventName"] = script.split("/")[-1]
        elif line.startswith("AFTER"):
            deps.append(line.split()[1])
        elif line.startswith("REL"):
            deps.append(line.split()[1])
    info["Dependencies"] = ",".join(deps)
    return info

def main():
    jobs = [extract(b) for b in parse_blocks(INPUT)]
    with open(OUTPUT, "w", newline="") as out:
        w = csv.DictWriter(out,
            fieldnames=["EventName","JobType","JobID","ScriptName","Dependencies"])
        w.writeheader()
        w.writerows(jobs)
    print(f"→ wrote {len(jobs)} rows into {OUTPUT}")

if __name__ == "__main__":
    main()
