import pandas as pd

# Sample data
df1 = pd.DataFrame({'colX': [10, 20, 30]})
df2 = pd.DataFrame({'A': [10, 20, 40], 'B': ['apple', 'banana', 'cherry']})

# List to store matched values
matched_values = []

# Iterate over df1
for value in df1['colX']:
    # Check if value exists in df2['A']
    match = df2.loc[df2['A'] == value, 'B']
    
    # Append the matched value if found, else None
    if not match.empty:
        matched_values.append(match.values[0])
    else:
        matched_values.append(None)

# Add list as new column
df1['matched_B'] = matched_values

print(df1)
