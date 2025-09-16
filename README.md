import pandas as pd

def process_lineage_excel(lineage_excel, runsheet_excel, output_file="processed_lineage.xlsx"):
    # Load lineage Excel
    lineage_df = pd.read_excel(lineage_excel)

    # Load runsheet Excel
    runsheet_df = pd.read_excel(runsheet_excel)

    data = []
    for idx, row in lineage_df.iterrows():
        use_case = row["UseCase"]
        lineage = row["Lineage"]

        if pd.isna(lineage):
            continue

        job_ids = [job.strip() for job in lineage.split("->")]

        step = 1
        for job in job_ids:
            sub_jobs = job.split("_")
            for sub_step, sub_job in enumerate(sub_jobs, start=1):
                data.append({
                    "UseCase": use_case,
                    "Step": step,
                    "SubStep": sub_step,
                    "JobID": sub_job
                })
            step += 1

    expanded_df = pd.DataFrame(data)

    # Merge with runsheet details (assuming JobID is the join key)
    merged_df = pd.merge(expanded_df, runsheet_df, on="JobID", how="left")

    # Save final result
    merged_df.to_excel(output_file, index=False)
    print(f"Processed lineage saved to {output_file}")
    return merged_df
