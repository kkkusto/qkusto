import pandas as pd
import re

def clean_jobid(jobid: str) -> str:
    """Clean up JobID by removing unwanted characters."""
    return re.sub(r'[^A-Za-z0-9_]', '', jobid.strip())

def process_lineage_excel(lineage_excel, runsheet_excel, output_file="processed_lineage.xlsx"):
    # Load lineage Excel
    lineage_df = pd.read_excel(lineage_excel)
    runsheet_df = pd.read_excel(runsheet_excel)

    data = []
    for idx, row in lineage_df.iterrows():
        use_case = row["UseCase"]
        lineage_cell = str(row["Lineage"])

        if pd.isna(lineage_cell) or lineage_cell.strip() == "":
            continue

        # Extract Main Job ID
        main_job_match = re.search(r'Main Job ID[: ]*([A-Za-z0-9_]+)', lineage_cell)
        main_job_id = clean_jobid(main_job_match.group(1)) if main_job_match else None

        # Split multiple rows inside a lineage cell
        lineage_lines = [l.strip() for l in lineage_cell.splitlines() if l.strip()]

        # Track seen jobs per use case
        seen_jobs = set()

        for lineage_idx, lineage in enumerate(lineage_lines, start=1):
            # Skip "Main Job ID" text line
            if "Main Job ID" in lineage:
                continue

            job_ids = [clean_jobid(job) for job in lineage.split("->") if job.strip()]

            step = 1
            for job in job_ids:
                sub_jobs = job.split("_")
                for sub_step, sub_job in enumerate(sub_jobs, start=1):
                    sub_job = clean_jobid(sub_job)

                    # Skip if it is the main job ID (will add later at the end)
                    if sub_job == main_job_id:
                        continue
                    if sub_job in seen_jobs:
                        continue
                    seen_jobs.add(sub_job)

                    data.append({
                        "UseCase": use_case,
                        "MainJobID": main_job_id,
                        "LineageGroup": lineage_idx,
                        "Step": step,
                        "SubStep": sub_step,
                        "JobID": sub_job
                    })
                step += 1

        # Finally: add the Main Job ID once at the end
        if main_job_id and main_job_id not in seen_jobs:
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
