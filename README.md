import pandas as pd
from pathlib import Path

def merge_excels(folder: str):
    folder_path = Path(folder)
    files = list(folder_path.glob("*.xlsx"))

    all_frames = []
    error_frames = []

    for file in files:
        try:
            xls = pd.ExcelFile(file)
            for sheet in xls.sheet_names:
                df = pd.read_excel(file, sheet_name=sheet, engine="openpyxl")
                df = df.dropna(how="all")
                if df.empty:
                    continue

                # Normalize column names
                df.columns = [c.strip().lower() for c in df.columns]

                # Ensure required columns exist
                if not {"table_name", "size_in_gb", "size_in_tb", "size_in_pb", "comments"} <= set(df.columns):
                    continue

                # Split into error and clean frames
                error_df = df[df["comments"].astype(str).str.contains("error", case=False, na=False)].copy()
                clean_df = df[~df.index.isin(error_df.index)].copy()

                # Add lineage info
                error_df["source_file"] = file.name
                error_df["sheet_name"] = sheet
                clean_df["source_file"] = file.name
                clean_df["sheet_name"] = sheet

                all_frames.append(clean_df)
                if not error_df.empty:
                    error_frames.append(error_df)

        except Exception as e:
            print(f"[WARN] Skipping {file}: {e}")

    if not all_frames:
        raise SystemExit("❌ No valid data found.")

    combined_clean = pd.concat(all_frames, ignore_index=True).drop_duplicates()
    combined_error = pd.concat(error_frames, ignore_index=True).drop_duplicates() if error_frames else pd.DataFrame()

    return combined_clean, combined_error


def summarize(combined_clean: pd.DataFrame, combined_error: pd.DataFrame, output: str = "summary.xlsx"):
    # Extract db from "db.table"
    combined_clean["database"] = combined_clean["table_name"].astype(str).str.split(".").str[0]
    if not combined_error.empty:
        combined_error["database"] = combined_error["table_name"].astype(str).str.split(".").str[0]

    # Group by db for sizes
    size_summary = combined_clean.groupby("database", as_index=False)[["size_in_gb", "size_in_tb", "size_in_pb"]].sum()

    # Error counts per db
    if not combined_error.empty:
        error_counts = combined_error.groupby("database").size().reset_index(name="error_table_count")
    else:
        error_counts = pd.DataFrame(columns=["database", "error_table_count"])

    # Merge results
    final_summary = pd.merge(size_summary, error_counts, on="database", how="left").fillna(0)

    # Save everything to Excel
    with pd.ExcelWriter(output) as writer:
        combined_clean.to_excel(writer, sheet_name="Clean_Data", index=False)
        if not combined_error.empty:
            combined_error.to_excel(writer, sheet_name="Error_Tables", index=False)
        final_summary.to_excel(writer, sheet_name="DB_Summary", index=False)

    print(f"✅ Summary written to {output}")
    return final_summary


# Example usage
# clean, errors = merge_excels("C:/path/to/folder")
# summary = summarize(clean, errors, "db_summary.xlsx")
