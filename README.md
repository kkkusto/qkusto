import pandas as pd

# Read the CSV file
df = pd.read_csv('input.csv')  # Replace 'input.csv' with your actual CSV file path

# Extract the column
tables = df['TableToAssess'].dropna().unique()

# Open (or create) the output script file
with open('esp_identification.sh', 'w') as script_file:
    for table in tables:
        script_file.write(f'echo "{table}"
')
        script_file.write(f'grep -ir "{table}" * | awk -F\':\' '{{print $1}}' | uniq > {table}.txt\n')
