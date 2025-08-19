import os
import re
import pandas as pd

# Map command categories
COMMAND_CATEGORY = {
    "-ls": "list",
    "-cat": "read",
    "-text": "read",
    "-get": "download",
    "-copyToLocal": "download",
    "-put": "upload",
    "-copyFromLocal": "upload",
    "-mv": "move",
    "-cp": "copy",
    "-rm": "delete",
    "-mkdir": "mkdir"
}

# Regex to capture hadoop fs or hdfs dfs commands
HADOOP_FS_PATTERN = re.compile(r"((?:hadoop|hdfs)\s+(?:fs|dfs)\s+(-\S+)(.*))")

def parse_hadoop_command(line):
    """Extract command, source, target, and extra flags from a Hadoop fs/dfs line."""
    match = HADOOP_FS_PATTERN.search(line)
    if not match:
        return None

    full_cmd = match.group(1).strip()
    cmd_flag = match.group(2).strip()
    args = match.group(3).strip().split()

    category = COMMAND_CATEGORY.get(cmd_flag, "other")

    source, target = None, None
    extra_flags = []

    # Separate extra flags (those starting with -)
    non_flag_args = []
    for arg in args:
        if arg.startswith("-"):
            extra_flags.append(arg)
        else:
            non_flag_args.append(arg)

    # Decide source/target based on command type
    if cmd_flag in ["-put", "-copyFromLocal", "-mv", "-cp"]:
        if len(non_flag_args) >= 2:
            source, target = non_flag_args[0], non_flag_args[1]
    elif cmd_flag in ["-get", "-copyToLocal"]:
        if len(non_flag_args) >= 2:
            source, target = non_flag_args[0], non_flag_args[1]
    elif cmd_flag in ["-rm", "-cat", "-text", "-ls"]:
        if len(non_flag_args) >= 1:
            target = non_flag_args[0]
    elif cmd_flag == "-mkdir":
        if len(non_flag_args) >= 1:
            target = non_flag_args[0]

    return {
        "command": cmd_flag,
        "statement": full_cmd,
        "category": category,
        "target": target,
        "source": source,
        "extra_flags": " ".join(extra_flags) if extra_flags else None
    }

def read_file_with_continuation(filepath):
    """Read file and merge lines with backslash continuations. Ignore comment lines."""
    merged_lines = []
    buffer = ""
    with open(filepath, "r", encoding="utf-8") as f:
        for line in f:
            line = line.strip()
            if not line or line.startswith("#"):  # ignore empty and comment lines
                continue
            if line.endswith("\\"):
                buffer += line[:-1] + " "  # remove backslash, add space
            else:
                buffer += line
                merged_lines.append(buffer)
                buffer = ""
    if buffer:
        merged_lines.append(buffer)
    return merged_lines

def process_sh_files(folder_path, output_csv="hadoop_fs_commands.csv"):
    records = []

    for root, _, files in os.walk(folder_path):
        for file in files:
            if file.endswith(".sh"):
                file_path = os.path.join(root, file)
                lines = read_file_with_continuation(file_path)
                for line in lines:
                    parsed = parse_hadoop_command(line)
                    if parsed:
                        parsed["source_filename"] = file
                        parsed["foldername"] = os.path.basename(root)
                        records.append(parsed)

    df = pd.DataFrame(records, columns=[
        "command", "statement", "category", 
        "target", "source", "extra_flags", 
        "source_filename", "foldername"
    ])

    df.to_csv(output_csv, index=False)
    print(f"Saved results to {output_csv}")
    return df

# Example usage
if __name__ == "__main__":
    folder_to_scan = "./scripts"  # change this to your folder path
    df = process_sh_files(folder_to_scan)
    print(df.head())
