import pandas as pd

# Load CSV
df = pd.read_csv('your_file.csv')

# Make sure table_name is a string
df['table_name'] = df['table_name'].astype(str)

# 1. Tables starting with a number
starts_with_number = df[df['table_name'].str.match(r'^\d')]

# 2. Tables containing 'tmp' or 'temp' (case-insensitive)
has_tmp_or_temp = df[df['table_name'].str.contains(r'\b(tmp|temp)\b', case=False, regex=True)]

# 3. Tables containing 'backup', 'bkp', or 'bkup' (case-insensitive)
has_backup_variants = df[df['table_name'].str.contains(r'\b(backup|bkp|bkup)\b', case=False, regex=True)]

# Print results
print("Tables starting with a number:")
print(starts_with_number)

print("\nTables with 'tmp' or 'temp':")
print(has_tmp_or_temp)

print("\nTables with 'backup', 'bkp', or 'bkup':")
print(has_backup_variants)
