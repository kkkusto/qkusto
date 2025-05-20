import re
import csv

INPUT  = "jobs.txt"           # your full job-definition file
OUTPUT = "wrapper_input.csv"  # the CSV your wrapper expects

def parse_blocks(path):
    """Yield each job block (from AIX_JOB or LINUX_JOB down to ENDJOB)."""
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
    """
    Given a string like "(PAR1,PAR2,PAR3)",
    strip parentheses and return ['PAR1','PAR2','PAR3'].
    """
    return [tok.strip()
            for tok in s.strip().lstrip("(").rstrip(")").split(",")
            if tok.strip()]

def extract(block):
    """
    From one job-block, build a dict of:
      EventName, JobType, JobID, ScriptName, Dependencies
    """
    info = {
        "EventName":   "",
        "JobType":     "",
        "JobID":       "",
        "ScriptName":  "",
        "Dependencies":"",
    }
    deps = []
    # to accumulate multi-line parenthesized deps
    collecting = False
    collect_buf = ""

    # first line has the type and ID
    m = re.match(r"^(AIX_JOB|LINUX_JOB)\s+(\S+)", block[0])
    info["JobType"], info["JobID"] = m.groups()

    for line in block[1:]:
        # SCRIPTNAME → full path and file name
        if line.startswith("SCRIPTNAME"):
            script = line.split(None,1)[1]
            info["ScriptName"] = script
            info["EventName"]  = script.split("/")[-1]

        # single-line REL
        elif line.startswith("REL"):
            rel = line.split(None,1)[1].strip()
            deps.append(rel)

        # start of AFTER (could be parenthesized or single)
        elif line.startswith("AFTER"):
            tail = line.split(None,1)[1].strip()
            if tail.startswith("("):
                # begin collecting until “)” appears
                collecting = True
                collect_buf = tail.lstrip("+")
                if ")" in tail:
                    collecting = False
                    deps += split_parenthesized(collect_buf)
            else:
                # simple single job
                deps.append(tail)

        # continuation of a multi-line AFTER
        elif collecting:
            part = line.strip().lstrip("+")
            collect_buf += part
            if ")" in line:
                collecting = False
                deps += split_parenthesized(collect_buf)

    info["Dependencies"] = ",".join(deps)
    return info

def main():
    jobs = [extract(b) for b in parse_blocks(INPUT)]
    with open(OUTPUT, "w", newline="") as csvfile:
        w = csv.DictWriter(csvfile,
            fieldnames=["EventName","JobType","JobID","ScriptName","Dependencies"])
        w.writeheader()
        w.writerows(jobs)
    print(f"✔ Wrote {len(jobs)} rows to {OUTPUT}")

if __name__ == "__main__":
    main()
