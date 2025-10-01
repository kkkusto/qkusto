SELECT COUNT(CASE WHEN order_date BETWEEN '2023-01-01' AND '2023-12-31' THEN 1 END) count_2023,
       COUNT(CASE WHEN order_date BETWEEN '2024-01-01' AND '2024-12-31' THEN 1 END) count_2024,
       COUNT(CASE WHEN order_date BETWEEN '2025-01-01' AND '2025-12-31' THEN 1 END) count_2025
FROM customer_orders;


import pandas as pd

# Input Excel and output shell script
excel_file = "input.xlsx"
output_sh = "run_counts.sh"

# Read Excel
df = pd.read_excel(excel_file)

queries = []

for _, row in df.iterrows():
    if str(row["legacy system"]).strip().lower() == "hadoop":
        table = row["TableName"]   # column C
        date_col = row["DateColumn"]  # column K

        query = f"""
hive -e "SELECT COUNT(CASE WHEN YEAR({date_col}) = 2023 THEN 1 END) count_2023,
               COUNT(CASE WHEN YEAR({date_col}) = 2024 THEN 1 END) count_2024,
               COUNT(CASE WHEN YEAR({date_col}) = 2025 THEN 1 END) count_2025
        FROM {table};" >> results.csv
"""
        queries.append(query)

# Write shell script
with open(output_sh, "w") as f:
    f.write("#!/bin/bash\n")
    f.write('echo "count_2023,count_2024,count_2025" > results.csv\n')
    for q in queries:
        f.write(q + "\n")

print(f"✅ Shell script created: {output_sh}")
