import pandas as pd

# Load the two CSV files
df1 = pd.read_csv("file1.csv")
df2 = pd.read_csv("file2.csv")

# Merge the dataframes on two columns (e.g., 'ColumnA' and 'ColumnB')
merged_df = df1.merge(df2, on=['ColumnA', 'ColumnB'], how='inner')  # Change 'inner' to 'left', 'right', or 'outer' if needed

# Save the merged dataframe to a new CSV file
merged_df.to_csv("merged_output.csv", index=False)

# Display the first few rows of the merged dataframe
print(merged_df.head())
