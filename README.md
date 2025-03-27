import pandas as pd

# Load CSV
df = pd.read_csv('your_file.csv')

# Make sure table_name is string
df['table_name'] = df['table_name'].astype(str)

# 1. Starts with a number
starts_with_number = df[df['table_name'].str.match(r'^\d')][['table_name']].drop_duplicates()
starts_with_number.to_csv('tables_start_with_number.csv', index=False)

# 2. Contains 'tmp' or 'temp'
has_tmp_or_temp = df[df['table_name'].str.contains(r'\b(tmp|temp)\b', case=False, regex=True)][['table_name']].drop_duplicates()
has_tmp_or_temp.to_csv('tables_with_tmp_or_temp.csv', index=False)

# 3. Contains 'backup', 'bkp', or 'bkup'
has_backup_variants = df[df['table_name'].str.contains(r'\b(backup|bkp|bkup)\b', case=False, regex=True)][['table_name']].drop_duplicates()
has_backup_variants.to_csv('tables_with_backup_variants.csv', index=False)

# 4. Length <= 3
len_leq_3 = df[df['table_name'].str.len() <= 3][['table_name']].drop_duplicates()
len_leq_3.to_csv('tables_with_len_leq_3.csv', index=False)

# 5. Ends with a number
ends_with_number = df[df['table_name'].str.match(r'.*\d$')][['table_name']].drop_duplicates()
ends_with_number.to_csv('tables_ending_with_number.csv', index=False)

# 6. Starts with exactly 3 digits
starts_with_3_digits = df[df['table_name'].str.match(r'^\d{3}\D') | df['table_name'].str.fullmatch(r'^\d{3}')][['table_name']].drop_duplicates()
starts_with_3_digits.to_csv('tables_start_with_3_digits.csv', index=False)
