import pandas as pd

def process_lineage_with_runsheet(lineage_file, runsheet_file, output_file="processed_lineage.xlsx"):
    # Load runsheet Excel
    runsheet_df = pd.read_excel(runsheet_file)
    
    data = []
    with open(lineage_file, "r") as f:
        for line_num, line in enumerate(f, start=1):
            line = line.strip()
            if not line:
                continue

            job_ids = [job.strip() for job in line.split("->")]

            step = 1
            for job in job_ids:
                # If multiple jobs are combined with '_'
                sub_jobs = job.split("_")
                for sub_step, sub_job in enumerate(sub_jobs, start=1):
                    data.append({
                        "Lineage": line_num,
                        "Step": step,
                        "SubStep": sub_step,
                        "JobID": sub_job
                    })
                step += 1

    lineage_df = pd.DataFrame(data)

    # Merge with runsheet (must contain JobID column)
    merged_df = pd.merge(lineage_df, runsheet_df, on="JobID", how="left")

    # Save output
    merged_df.to_excel(output_file, index=False)
    print(f"Processed lineage saved to {output_file}")
    return merged_df
