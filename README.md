import pandas as pd

def process_lineage_to_dataframe(file_path):
    data = []
    with open(file_path, "r") as f:
        for line_num, line in enumerate(f, start=1):
            line = line.strip()
            if not line:
                continue  # skip empty lines

            # Split lineage by '->' and strip spaces
            job_ids = [job.strip() for job in line.split("->")]

            for step, job in enumerate(job_ids, start=1):
                data.append({
                    "Lineage": line_num,
                    "Step": step,
                    "JobID": job
                })

    return pd.DataFrame(data)


# Example usage with your TXT file
file_path = "/mnt/data/lineage.txt"  # placeholder, user should upload file here
df = process_lineage_to_dataframe(file_path)

import caas_jupyter_tools
caas_jupyter_tools.display_dataframe_to_user("Processed Lineage", df)
