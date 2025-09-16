import pandas as pd

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

        # Track seen JobIDs per UseCase
        seen_jobs = set()

        # Split multiple rows in a single cell (\n-separated)
        lineage_lines = [l.strip() for l in lineage_cell.splitlines() if l.strip()]

        for lineage_idx, lineage in enumerate(lineage_lines, start=1):
            job_ids = [job.strip() for job in lineage.split("->")]

            step = 1
            for job in job_ids:
                sub_jobs = job.split("_")
                for sub_step, sub_job in enumerate(sub_jobs, start=1):
                    if sub_job in seen_jobs:
                        continue  # skip duplicates for this UseCase
                    seen_jobs.add(sub_job)

                    data.append({
                        "UseCase": use_case,
                        "LineageGroup": lineage_idx,
                        "Step": step,
                        "SubStep": sub_step,
                        "JobID": sub_job
                    })
                step += 1

    expanded_df = pd.DataFrame(data)

    # Merge with runsheet details (assuming JobID is the join key)
    merged_df = pd.merge(expanded_df, runsheet_df, on="JobID", how="left")

    # Save to Excel
    merged_df.to_excel(output_file, index=False)
    print(f"Processed lineage saved to {output_file}")
    return merged_df
