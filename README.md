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

# Regex to capture hadoop fs commands
HADOOP_FS_PATTERN = re.compile(r"(hadoop\s+fs\s+(-\S+)(.*))")

def parse_hadoop_command(line):
    """Extract command, source, and target from a Hadoop fs line."""
    match = HADOOP_FS_PATTERN.search(line)
    if not match:
        return None
    
    full_cmd = match.group(1).strip()
    cmd_flag = match.group(2).strip()
    args = match.group(3).strip().split()
    
    category = COMMAND_CATEGORY.get(cmd_flag, "other")
    
    source, target = None, None
    if cmd_flag in ["-put", "-copyFromLocal", "-mv", "-cp"]:
        if len(args) >= 2:
            source, target = args[0], args[1]
    elif cmd_flag in ["-get", "-copyToLocal"]:
        if len(args) >= 2:
            source, target = args[0], args[1]
    elif cmd_flag in ["-rm", "-cat", "-text", "-ls"]:
        if len(args) >= 1:
            target = args[0]
    elif cmd_flag == "-mkdir":
        if len(args) >= 1:
            target = args[0]
    
    return {
        "command": cmd_flag,
        "statement": full_cmd,
        "category": category,
        "target": target,
        "source": source
    }

def process_sh_files(folder_path, output_csv="hadoop_fs_commands.csv"):
    records = []
    
    for root, _, files in os.walk(folder_path):
        for file in files:
            if file.endswith(".sh"):
                file_path = os.path.join(root, file)
                with open(file_path, "r", encoding="utf-8") as f:
                    for line in f:
                        parsed = parse_hadoop_command(line)
                        if parsed:
                            parsed["source_filename"] = file
                            parsed["foldername"] = os.path.basename(root)
                            records.append(parsed)
    
    df = pd.DataFrame(records, columns=[
        "command", "statement", "category", 
        "target", "source", "source_filename", "foldername"
    ])
    
    df.to_csv(output_csv, index=False)
    print(f"Saved results to {output_csv}")
    return df

# Example usage
if __name__ == "__main__":
    folder_to_scan = "./scripts"  # change this to your folder path
    df = process_sh_files(folder_to_scan)
    print(df.head())
