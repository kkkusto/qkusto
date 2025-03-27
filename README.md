import pandas as pd
import plotly.express as px

# Load the CSV
df = pd.read_csv('your_file.csv')
df['table_name'] = df['table_name'].astype(str)

# Core 6 Rules
df['starts_with_number'] = df['table_name'].str.match(r'^\d')
df['has_tmp_or_temp'] = df['table_name'].str.contains(r'\b(tmp|temp)\b', case=False, regex=True)
df['has_backup_variants'] = df['table_name'].str.contains(r'\b(backup|bkp|bkup)\b', case=False, regex=True)
df['len_leq_3'] = df['table_name'].str.len() <= 3
df['ends_with_number'] = df['table_name'].str.match(r'.*\d$')
df['starts_with_3_digits'] = df['table_name'].str.match(r'^\d{3}\D') | df['table_name'].str.fullmatch(r'\d{3}')

# New rules: ends with exactly N digits (1 to 6)
for n in range(1, 7):
    rule_name = f'ends_with_{n}_digits'
    df[rule_name] = df['table_name'].str.contains(fr'(?<!\d)\d{{{n}}}$')

# Prepare all rule names for melting
rule_columns = [
    'starts_with_number',
    'has_tmp_or_temp',
    'has_backup_variants',
    'len_leq_3',
    'ends_with_number',
    'starts_with_3_digits'
] + [f'ends_with_{n}_digits' for n in range(1, 7)]

# Melt DataFrame
melted = df.melt(id_vars='table_name',
                 value_vars=rule_columns,
                 var_name='Rule',
                 value_name='Matched')

# Filter matched only
matched = melted[melted['Matched'] == True]

# Count occurrences
rule_counts = matched['Rule'].value_counts().reset_index()
rule_counts.columns = ['Rule', 'Count']

# Label mapping for better readability
label_map = {
    'starts_with_number': 'Starts with Number',
    'has_tmp_or_temp': 'Has tmp/temp',
    'has_backup_variants': 'Has backup/bkp/bkup',
    'len_leq_3': 'Length ≤ 3',
    'ends_with_number': 'Ends with Number',
    'starts_with_3_digits': 'Starts with 3 Digits',
    **{f'ends_with_{n}_digits': f'Ends with {n} Digit{"s" if n > 1 else ""}' for n in range(1, 7)}
}
rule_counts['Rule'] = rule_counts['Rule'].map(label_map)

# Plot
fig = px.bar(rule_counts, x='Rule', y='Count', text='Count',
             title='Table Name Rule Matches',
             labels={'Rule': 'Rule Condition', 'Count': 'Number of Matches'})

fig.update_traces(textposition='outside')
fig.update_layout(xaxis_tickangle=-30, yaxis=dict(title='Count'), showlegend=False)

fig.show()


# Save table names matching each rule to separate CSVs
for rule in rule_columns:
    matched_names = df[df[rule]][['table_name']].drop_duplicates()

    # Map to user-friendly filename
    label = label_map.get(rule, rule).lower().replace(" ", "_").replace("≤", "leq").replace("≥", "geq").replace(">", "gt").replace("<", "lt")
    
    filename = f"tables_{label}.csv"
    matched_names.to_csv(filename, index=False)
