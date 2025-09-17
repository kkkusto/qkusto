import pandas as pd

def expand_jobids_with_info(input_csv, output_csv):
    df = pd.read_csv(input_csv)
    expanded_rows = []

    for _, row in df.iterrows():
        step = row["Step Number"]
        job_ids = str(row["Job ID"]).split("_")  # split by underscore

        for part in job_ids:
            part = part.strip()
            if not part:
                continue

            if part.startswith("RFT"):
                expanded_rows.append({"Step Number": step, "Job ID": part, "Info": ""})
            else:
                # Attach to last added row as Info
                if expanded_rows:
                    expanded_rows[-1]["Info"] = part
                else:
                    expanded_rows.append({"Step Number": step, "Job ID": "", "Info": part})

    expanded_df = pd.DataFrame(expanded_rows)

    # Save
    expanded_df.to_csv(output_csv, index=False)
    print(f"Expanded Job IDs with Info saved to {output_csv}")
    return expanded_df

# Example usage:
# expand_jobids_with_info("job_sequence_resolved.csv", "job_sequence_expanded.csv")
