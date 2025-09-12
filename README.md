import pandas as pd

def summarize_by_db(combined: pd.DataFrame, output: str = "db_summary.xlsx"):
    # Ensure column names are normalized
    cols = {c.lower(): c for c in combined.columns}
    size_gb_col = cols.get("size_in_gb")
    size_tb_col = cols.get("size_in_tb")
    size_pb_col = cols.get("size_in_pb")
    table_col   = cols.get("table_name")

    if not table_col:
        raise ValueError("No 'table_name' column found!")

    # Extract db from "db.table"
    combined["database"] = combined[table_col].astype(str).str.split(".").str[0]

    # Group by db and sum sizes
    summary = combined.groupby("database", as_index=False)[[size_gb_col, size_tb_col, size_pb_col]].sum()

    # Save output
    if output.endswith(".csv"):
        summary.to_csv(output, index=False)
    else:
        summary.to_excel(output, index=False)

    print(f"✅ Database summary saved to {output}")
    return summary


# Example usage after merging:
# combined = merge_excels("C:/path/to/folder", "combined.xlsx")
# summary = summarize_by_db(combined, "db_summary.xlsx")
