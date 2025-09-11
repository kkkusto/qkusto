import pandas as pd

# --- Step 1: Load data ---
usecase_df = pd.read_excel("usecase_tables.xlsx")   # has p1/p2/p3 column
all_tables_df = pd.read_excel("all_tables.xlsx")    # 1453 master list

# Normalize text
for col in ["Database", "Table"]:
    usecase_df[col] = usecase_df[col].str.strip().str.lower()
    all_tables_df[col] = all_tables_df[col].str.strip().str.lower()

# Unique key
usecase_df["db_table"] = usecase_df["Database"] + "." + usecase_df["Table"]
all_tables_df["db_table"] = all_tables_df["Database"] + "." + all_tables_df["Table"]

# --- Step 2: Normalize priorities ---
usecase_df["priority"] = usecase_df["p1/p2/p3"].str.strip().str.upper()

# --- Step 3: Collect priorities per table ---
priority_map = (
    usecase_df.groupby("db_table")["priority"]
    .apply(set)  # each table may have multiple tags
    .to_dict()
)

# --- Step 4: Conflict resolution function ---
def resolve_priority(priorities):
    if not priorities:
        return "Not_Associated"
    if "P1" in priorities:
        return "P1"
    if "P2.1" in priorities and "P3" in priorities:
        return "P2.1"
    if "P2.2" in priorities and "P3" in priorities:
        return "P2.2"
    # If only one priority or non-conflicting multiple
    return sorted(priorities)[0]   # stable pick

# --- Step 5: Assign priorities to all 1453 tables ---
all_tables_df["priority_raw"] = all_tables_df["db_table"].map(priority_map)
all_tables_df["Final_Priority"] = all_tables_df["priority_raw"].apply(resolve_priority)

# --- Step 6: Save output ---
all_tables_df.drop(columns=["priority_raw"], inplace=True)
all_tables_df.to_excel("all_tables_with_priority.xlsx", index=False)

print("✅ Tagged priorities saved to all_tables_with_priority.xlsx")
