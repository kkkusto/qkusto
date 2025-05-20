import re
import csv

INPUT  = "jobs.txt"
OUTPUT = "wrapper_input.csv"

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
    """
    From one job-block, build a dict with these fields:
      EventName, JobType, JobID, ScriptName, After (list), Rel (list)
    """
    info = {
        "EventName":  "",
        "JobType":    "",
        "JobID":      "",
        "ScriptName": "",
        "After":      [],
        "Rel":        []
    }

    # state machines for multi-line fields
    collecting = None   # either "SCRIPT", "AFTER" or "REL"
    buf = ""

    # header → JobType and JobID
    m = re.match(r"^(AIX_JOB|LINUX_JOB)\s+(\S+)", block[0])
    info["JobType"], info["JobID"] = m.groups()

    for line in block[1:]:
        # —— SCRIPTNAME (may span lines ending in '+') ——
        if collecting == "SCRIPT":
            part = line.strip().lstrip("+")
            buf += part
            if not line.strip().endswith("+"):
                info["ScriptName"] = buf
                info["EventName"]  = buf.split("/")[-1]
                collecting = None
            continue

        if line.startswith("SCRIPTNAME"):
            _, tail = line.split(None,1)
            if tail.endswith("+"):
                collecting = "SCRIPT"
                buf = tail.rstrip("+")
            else:
                info["ScriptName"] = tail
                info["EventName"]  = tail.split("/")[-1]
            continue

        # —— AFTER / REL (may span lines inside parentheses) ——
        if collecting in ("AFTER","REL"):
            part = line.strip().lstrip("+")
            buf += part
            # end if we see ')'
            if ")" in line:
                ids = split_parenthesized(buf)
                info[collecting].extend(ids)
                collecting = None
            continue

        if line.startswith("AFTER") or line.startswith("REL"):
            keyword, rest = line.split(None,1)
            rest = rest.strip()
            if rest.startswith("("):
                # multi-line parenthesized
                collecting = keyword
                buf = rest.lstrip("+")
                if ")" in rest:
                    # all on one line
                    ids = split_parenthesized(buf)
                    info[keyword].extend(ids)
                    collecting = None
            else:
                # single ID
                info[keyword].append(rest)
            continue

        # ignore everything else

    # flatten lists to comma-joined strings
    return {
        "EventName":  info["EventName"],
        "JobType":    info["JobType"],
        "JobID":      info["JobID"],
        "ScriptName": info["ScriptName"],
        "After":      ",".join(info["AFTER"]),
        "Rel":        ",".join(info["REL"])
    }

def main():
    blocks = list(parse_blocks(INPUT))
    rows   = [extract(b) for b in blocks]

    with open(OUTPUT, "w", newline="") as csvfile:
        writer = csv.DictWriter(
            csvfile,
            fieldnames=["EventName","JobType","JobID","ScriptName","After","Rel"]
        )
        writer.writeheader()
        writer.writerows(rows)

    print(f"✔ Wrote {len(rows)} rows to {OUTPUT}")

if __name__ == "__main__":
    main()
