import pandas as pd

# Step 1: Load the main dataframe
main_df = pd.read_csv('main_file.csv')  # Replace with actual filename

# Step 1: Filter out ENVIRON_NAME containing 'prod' or 'prd'
main_df = main_df[~main_df['ENVIRON_NAME'].str.contains('prod|prd', case=False, na=False)]

# Step 2: Filter out PARM_DESC with substrings
exclude_keywords = ['table', 'tbl', 'tlb', 'target', 'source']
pattern = '|'.join(exclude_keywords)
main_df = main_df[~main_df['PARM_DESC'].str.contains(pattern, case=False, na=False)]

# Step 3: Load table mapping
table_map_df = pd.read_csv('db_table_map.csv')  # must contain 'table_name' column

# Placeholder for the final dataframe
final_df = pd.DataFrame()

# Step 4: For each table in table_map, process
for _, row in table_map_df.iterrows():
    table_name = row['table_name']
    db_name = row['db_name'] if 'db_name' in row else None  # Optional

    # Filter main_df for matching table names in PARM_VAL
    temp_df = main_df[main_df['PARM_VAL'].str.contains(table_name, na=False)]

    if not temp_df.empty:
        temp_df = temp_df.copy()
        temp_df['table_name'] = table_name
        temp_df['db_name'] = db_name

        # Step 5: Read runsheet
        runsheet_df = pd.read_csv('runsheet.csv')  # replace with actual filename

        # Step 6: Match JOB_ID in runsheet column 'Executables_Symbolics'
        temp2_df = pd.DataFrame()
        for job_id in temp_df['JOB_ID']:
            matched_rows = runsheet_df[runsheet_df['Executables_Symbolics'].str.contains(str(job_id), na=False)]
            if not matched_rows.empty:
                matched_rows = matched_rows.copy()
                matched_rows['JOB_ID'] = job_id  # add for traceability
                temp2_df = pd.concat([temp2_df, matched_rows], ignore_index=True)

        # Step 7: Add 'table_name' and 'db_name'
        temp2_df['table_name'] = table_name
        temp2_df['db_name'] = db_name

        # Step 8: Append to final dataframe
        final_df = pd.concat([final_df, temp2_df], ignore_index=True)

# Save final result
final_df.to_csv('table_oracle_runsheet.csv', index=False)
