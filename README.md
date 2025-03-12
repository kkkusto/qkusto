import pandas as pd

# Load the CSV file
df = pd.read_csv("file.csv")

# Split 'ColumnA' by '.' and create new columns 'A' and 'B'
df[['A', 'B']] = df['ColumnA'].astype(str).str.split('.', expand=True)

# Convert the entire DataFrame to uppercase and trim spaces
df = df.applymap(lambda x: x.strip().upper() if isinstance(x, str) else x)

# Save the modified dataframe
df.to_csv("output.csv", index=False)

# Display the first few rows
print(df.head())
