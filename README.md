import pandas as pd

# --- Step 1: Load data ---
usecase_df = pd.read_excel("usecase_tables.xlsx")   # contains p1/p2/p3 info
all_tables_df = pd.read_excel("all_tables.xlsx")    # 1453 master list

# Ensure consistent case and strip spaces
usecase_df["Database"] = usecase_df["Database"].str.strip().str.lower()
usecase_df["Table"] = usecase_df["Table"].str.strip().str.lower()
all_tables_df["Database"] = all_tables_df["Database"].str.strip().str.lower()
all_tables_df["Table"] = all_tables_df["Table"].str.strip().str.lower()

# Create a unique identifier Database.Table
usecase_df["db_table"] = usecase_df["Database"] + "." + usecase_df["Table"]
all_tables_df["db_table"] = all_tables_df["Database"] + "." + all_tables_df["Table"]

# --- Step 2: Normalize p2.1, p2.2 → p2 ---
usecase_df["priority"] = usecase_df["p1/p2/p3"].str.lower().replace({
    "p2.1": "p2",
    "p2.2": "p2"
})

# --- Step 3: Split by priority ---
p1_tables = set(usecase_df.loc[usecase_df["priority"] == "p1", "db_table"])
p2_tables = set(usecase_df.loc[usecase_df["priority"] == "p2", "db_table"])
p3_tables = set(usecase_df.loc[usecase_df["priority"] == "p3", "db_table"])
all_tables = set(all_tables_df["db_table"])

# --- Step 4: Deduplication logic ---
p1_unique = p1_tables
p2_unique = p2_tables - p1_tables
p3_unique = p3_tables - p1_tables - p2_tables
not_associated = all_tables - (p1_unique | p2_unique | p3_unique)

# --- Step 5: Convert to DataFrames ---
def to_df(db_table_set, label):
    data = [x.split(".", 1) for x in sorted(db_table_set)]
    return pd.DataFrame(data, columns=["Database", "Table"])

df_p1 = to_df(p1_unique, "P1")
df_p2 = to_df(p2_unique, "P2")
df_p3 = to_df(p3_unique, "P3")
df_not = to_df(not_associated, "Not_Associated")

# --- Step 6: Add counts summary ---
summary = pd.DataFrame({
    "Bucket": ["P1", "P2", "P3", "Not_Associated", "Total_All_Tables"],
    "Count": [len(df_p1), len(df_p2), len(df_p3), len(df_not), len(all_tables)]
})

# --- Step 7: Save results to Excel ---
with pd.ExcelWriter("final_table_classification.xlsx", engine="openpyxl") as writer:
    df_p1.to_excel(writer, sheet_name="P1_Unique", index=False)
    df_p2.to_excel(writer, sheet_name="P2_Unique", index=False)
    df_p3.to_excel(writer, sheet_name="P3_Unique", index=False)
    df_not.to_excel(writer, sheet_name="Not_Associated", index=False)
    summary.to_excel(writer, sheet_name="Summary", index=False)

print("✅ Final classification saved to final_table_classification.xlsx")
