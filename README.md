import re
import csv

INPUT  = "jobs.txt"           # your raw file
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
    """Turn '(A,B,C)' → ['A','B','C'] (stripping whitespace)."""
    return [tok.strip() 
            for tok in s.strip().lstrip("(").rstrip(")").split(",") 
            if tok.strip()]

def extract(block):
    info = {
        "EventName":   "",
        "JobType":     "",
        "JobID":       "",
        "ScriptName":  "",
        "Dependencies":"",
    }
    deps = []
    
    # State for multi-line collections
    collecting_deps = False
    deps_buf = ""
    deps_prefix = None    # either "AFTER" or "REL"
    
    collecting_script = False
    script_buf = ""
    
    # Parse the very first line: AIX_JOB or LINUX_JOB + ID
    m = re.match(r"^(AIX_JOB|LINUX_JOB)\s+(\S+)", block[0])
    info["JobType"], info["JobID"] = m.groups()

    for line in block[1:]:
        # ——— SCRIPTNAME handling ——————————————————————
        if collecting_script:
            part = line.strip().lstrip("+")
            script_buf += part
            if not line.strip().endswith("+"):
                # done collecting
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

        # ——— DEPENDENCY handling FOR AFTER and REL ——————————
        # if already collecting deps (multi-line):
        if collecting_deps:
            part = line.strip().lstrip("+")
            deps_buf += part
            if ")" in line:
                # finish collecting
                deps += split_parenthesized(deps_buf)
                collecting_deps = False
            continue

        # start of AFTER or REL
        if line.startswith("AFTER") or line.startswith("REL"):
            keyword, rest = line.split(None,1)
            rest = rest.strip()
            # parenthesized multi-line?
            if rest.startswith("("):
                deps_prefix = keyword
                collecting_deps = True
                deps_buf = rest.lstrip("+")
                # maybe it closes on the same line:
                if ")" in rest:
                    collecting_deps = False
                    deps += split_parenthesized(deps_buf)
            else:
                # single ID
                deps.append(rest)
            continue

        # everything else we ignore in this CSV

    info["Dependencies"] = ",".join(deps)
    return info

def main():
    blocks = list(parse_blocks(INPUT))
    jobs   = [extract(b) for b in blocks]
    with open(OUTPUT, "w", newline="") as csvfile:
        writer = csv.DictWriter(
            csvfile,
            fieldnames=["EventName","JobType","JobID","ScriptName","Dependencies"]
        )
        writer.writeheader()
        writer.writerows(jobs)
    print(f"✔ Wrote {len(jobs)} rows to {OUTPUT}")

if __name__ == "__main__":
    main()
