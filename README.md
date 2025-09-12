import pandas as pd
from pathlib import Path

def merge_excels(folder: str, output: str = "combined.xlsx"):
    folder_path = Path(folder)
    files = list(folder_path.glob("*.xlsx"))

    all_frames = []

    for file in files:
        try:
            # read all sheets
            xls = pd.ExcelFile(file)
            for sheet in xls.sheet_names:
                df = pd.read_excel(file, sheet_name=sheet, engine="openpyxl")

                # drop empty rows
                df = df.dropna(how="all")
                if df.empty:
                    continue

                # remove rows where 'comments' column has "Error"
                if "comments" in (c.lower() for c in df.columns):
                    # find the actual column name matching 'comments'
                    colname = next(c for c in df.columns if c.lower() == "comments")
                    df = df[~df[colname].astype(str).str.contains("error", case=False, na=False)]

                # keep only needed columns
                keep_cols = [c for c in df.columns if c.lower() in {"table_name", "size", "size in gb", "comments"}]
                df = df[keep_cols]

                # add source info
                df.insert(0, "source_file", file.name)
                df.insert(1, "sheet_name", sheet)

                all_frames.append(df)

        except Exception as e:
            print(f"[WARN] Skipping {file}: {e}")

    if not all_frames:
        print("No data found.")
        return

    combined = pd.concat(all_frames, ignore_index=True)

    # save output
    if output.endswith(".csv"):
        combined.to_csv(output, index=False)
    else:
        combined.to_excel(output, index=False)

    print(f"✅ Combined {len(all_frames)} frames from {len(files)} files into {output}")

# Example usage:
# merge_excels("C:/path/to/folder", "combined.xlsx")
