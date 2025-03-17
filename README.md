import os
import pandas as pd

def process_folder(folder_path, reference_csv):
    # Load reference CSV
    ref_df = pd.read_csv(reference_csv)
    
    for filename in os.listdir(folder_path):
        if filename.endswith('_temp.csv'):
            csv_path = os.path.join(folder_path, filename)
            base_name = filename.replace('_temp.csv', '')
            
            # Load temp CSV
            temp_df = pd.read_csv(csv_path)
            
            # Find matching rows in reference CSV
            matching_rows = ref_df[ref_df['table_name'] == base_name]
            
            if not matching_rows.empty:
                # Add original_ids column
                original_ids = ','.join(temp_df['First_Word'].tolist())
                ref_df.loc[ref_df['table_name'] == base_name, 'original_ids'] = original_ids
    
    # Save updated reference CSV
    updated_csv_path = os.path.join(folder_path, 'updated_reference.csv')
    ref_df.to_csv(updated_csv_path, index=False)

# Example usage
folder_path = 'your_folder_path_here'  # Replace with actual folder path
reference_csv = 'reference.csv'  # Replace with actual reference CSV path
process_folder(folder_path, reference_csv)
