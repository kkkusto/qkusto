import pandas as pd
from collections import defaultdict

# Input files
MIGRATED_SCHEMA_CSV = "databricks_schema.csv"
TABLE_MAPPING_CSV = "table_mapping.csv"
HADOOP_SCHEMA_TXT = "hadoop_schemas.txt"

# Load Databricks schema
df_migrated = pd.read_csv(MIGRATED_SCHEMA_CSV)
migrated_dict = defaultdict(set)
for _, row in df_migrated.iterrows():
    migrated_dict[row['table_name'].strip().lower()].add(row['col_name'].strip().lower())

# Load Hadoop to Databricks mapping
df_mapping = pd.read_csv(TABLE_MAPPING_CSV)
mapping = [(row['table_name'].strip().lower(), row['phase2_name'].strip().lower()) for _, row in df_mapping.iterrows()]

# Parse Hadoop schemas
def parse_hadoop_schema(file_path):
    schema_dict = defaultdict(set)
    current_table = None
    with open(file_path, 'r') as f:
        for line in f:
            line = line.strip()
            if line.startswith("TABLE:"):
                current_table = line.split(":", 1)[1].strip().lower()
            elif current_table and line:
                schema_dict[current_table].add(line.lower())
    return schema_dict

hadoop_dict = parse_hadoop_schema(HADOOP_SCHEMA_TXT)

# Compare based on mapping
results = []

for hadoop_table, databricks_table in mapping:
    hadoop_cols = hadoop_dict.get(hadoop_table, set())
    databricks_cols = migrated_dict.get(databricks_table, set())

    missing_in_databricks = hadoop_cols - databricks_cols
    extra_in_databricks = databricks_cols - hadoop_cols
    matched = hadoop_cols & databricks_cols

    results.append({
        "Hadoop Table": hadoop_table,
        "Databricks Table": databricks_table,
        "Missing in Databricks": ", ".join(sorted(missing_in_databricks)) or "None",
        "Extra in Databricks": ", ".join(sorted(extra_in_databricks)) or "None",
        "Matched Columns": ", ".join(sorted(matched)) or "None"
    })

# Export to CSV
df_result = pd.DataFrame(results)
df_result.to_csv("schema_comparison_report.csv", index=False)
print("Comparison report saved to schema_comparison_report.csv")
