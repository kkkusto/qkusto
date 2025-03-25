import pandas as pd

# Read the CSVs
left_df = pd.read_csv('left.csv')
right_df = pd.read_csv('right.csv')

# Clean: Remove rows where 'id' is NaN or empty string
left_df = left_df[left_df['id'].notna() & (left_df['id'].astype(str).str.strip() != '')]
right_df = right_df[right_df['id'].notna() & (right_df['id'].astype(str).str.strip() != '')]

# Optional: Convert to consistent type (e.g., int, str) for joining
left_df['id'] = left_df['id'].astype(str)
right_df['id'] = right_df['id'].astype(str)

# Perform left join
merged_df = pd.merge(left_df, right_df, on='id', how='left')

print(merged_df)
