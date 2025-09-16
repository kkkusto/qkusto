import pandas as pd
import re

def clean_jobid(jobid: str) -> str:
    """Remove unwanted chars from job IDs."""
    return re.sub(r'[^A-Za-z0-9_]', '', jobid.strip())

def process_lineage_txt(input_txt, output_excel="processed_lineage.xlsx"):
    data = []

    with open(input_txt, "r") as f:
        lineage_cell = f.read()

    # Extract "Main Job ID"
    main_job_match = re.search(r'Main Job ID[: ]*([A-Za-z0-9_]+)', lineage_cell)
    main_job_id = main_job_match.group(1) if main_job_match else None

    # Split into multiple lineage rows if needed
    lineage_lines = [l.strip() for l in lineage_cell.splitlines() if l.strip()]

    seen_jobs = set()  # Track duplicates

    for lineage_idx, lineage in enumerate(lineage_lines, start=1):
        if "Main Job ID" in lineage:
            continue  # Skip that line

        # Split by ->
        job_ids = [clean_jobid(job) for job in lineage.split("->") if job.strip()]

        step = 1
        for job in job_ids:
            sub_jobs = job.split("_")

            # Case 1: RFTCxxxx_Label → one job with label
            if len(sub_jobs) == 2 and sub_jobs[0].startswith("RFTC") and not sub_jobs[1].startswith("RFTC"):
                jobid = f"{sub_jobs[0]} ({sub_jobs[1]})"
                if jobid not in seen_jobs:
                    data.append({
                        "MainJobID": main_job_id,
                        "LineageGroup": lineage_idx,
                        "Step": step,
                        "SubStep": 1,
                        "JobID": jobid
                    })
                    seen_jobs.add(jobid)

            else:
                # Case 2: multiple JobIDs
                for sub_step, token in enumerate(sub_jobs, start=1):
                    token = clean_jobid(token)
                    if not token or token in seen_jobs:
                        continue
                    data.append({
                        "MainJobID": main_job_id,
                        "LineageGroup": lineage_idx,
                        "Step": step,
                        "SubStep": sub_step,
                        "JobID": token
                    })
                    seen_jobs.add(token)

            step += 1

    df = pd.DataFrame(data)
    df.to_excel(output_excel, index=False)
    print(f"Processed lineage saved to {output_excel}")
    return df

# Example usage:
# process_lineage_txt("lineage.txt", "output.xlsx")
            data.append({
                "UseCase": use_case,
                "MainJobID": main_job_id,
                "LineageGroup": lineage_idx,  # last group processed
                "Step": step,
                "SubStep": 1,
                "JobID": main_job_id
            })

    expanded_df = pd.DataFrame(data)

    # Merge with runsheet
    merged_df = pd.merge(expanded_df, runsheet_df, on="JobID", how="left")

    # Save
    merged_df.to_excel(output_file, index=False)
    print(f"Processed lineage saved to {output_file}")
    return merged_df
    return merged_df
