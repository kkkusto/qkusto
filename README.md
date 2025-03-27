import pandas as pd
import plotly.express as px

# Load CSV
df = pd.read_csv('your_file.csv')
df['table_name'] = df['table_name'].astype(str)

# Apply all 6 conditions
df['starts_with_number'] = df['table_name'].str.match(r'^\d')
df['has_tmp_or_temp'] = df['table_name'].str.contains(r'\b(tmp|temp)\b', case=False, regex=True)
df['has_backup_variants'] = df['table_name'].str.contains(r'\b(backup|bkp|bkup)\b', case=False, regex=True)
df['len_leq_3'] = df['table_name'].str.len() <= 3
df['ends_with_number'] = df['table_name'].str.match(r'.*\d$')
df['starts_with_3_digits'] = df['table_name'].str.match(r'^\d{3}\D') | df['table_name'].str.fullmatch(r'\d{3}')

# Melt the DataFrame to long format for plotting
melted = df.melt(id_vars='table_name', 
                 value_vars=[
                     'starts_with_number',
                     'has_tmp_or_temp',
                     'has_backup_variants',
                     'len_leq_3',
                     'ends_with_number',
                     'starts_with_3_digits'
                 ],
                 var_name='Rule',
                 value_name='Matched')

# Keep only matched rows
matched = melted[melted['Matched'] == True]

# Count how many table names matched each rule
rule_counts = matched['Rule'].value_counts().reset_index()
rule_counts.columns = ['Rule', 'Count']

# Optional: make labels prettier
label_map = {
    'starts_with_number': 'Starts with Number',
    'has_tmp_or_temp': 'Has tmp/temp',
    'has_backup_variants': 'Has backup/bkp/bkup',
    'len_leq_3': 'Length ≤ 3',
    'ends_with_number': 'Ends with Number',
    'starts_with_3_digits': 'Starts with 3 Digits'
}
rule_counts['Rule'] = rule_counts['Rule'].map(label_map)

# Plot using Plotly
fig = px.bar(rule_counts, x='Rule', y='Count',
             text='Count', title='Table Name Rule Matches',
             labels={'Rule': 'Rule Condition', 'Count': 'Number of Matches'})

fig.update_traces(textposition='outside')
fig.update_layout(xaxis_tickangle=-30, yaxis=dict(title='Count'), showlegend=False)

fig.show()
