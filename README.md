import os
import pandas as pd

def process_folder(folder_path):
    for filename in os.listdir(folder_path):
        if filename.endswith('.txt'):
            file_path = os.path.join(folder_path, filename)
            temp_lines = []
            unique_entries = set()
            temp_file_path = os.path.join(folder_path, filename.replace('.txt', '_temp.txt'))
            
            with open(file_path, 'r', encoding='utf-8') as file, open(temp_file_path, 'w', encoding='utf-8') as temp_file:
                for line in file:
                    line = line.strip()
                    if not line:
                        continue
                    
                    if '/' in line and ':' in line:
                        temp_file.write(line + '\n')
                        try:
                            parts = line.split(':', 1)
                            if len(parts) < 2:
                                continue
                            
                            first_word = parts[0].split()[0]  # Extract the first word
                            split_first_word = first_word.split('/')  # Split by '/'
                            
                            unique_entries.add((first_word, ','.join(split_first_word)))
                        except Exception:
                            continue
            
            # Convert set to DataFrame
            df = pd.DataFrame(unique_entries, columns=['First_Word', 'Split_Word'])
            
            # Remove duplicates and save CSV
            df.drop_duplicates(inplace=True)
            csv_file_path = os.path.join(folder_path, filename.replace('.txt', '_temp.csv'))
            df.to_csv(csv_file_path, index=False)

# Example usage
folder_path = 'your_folder_path_here'  # Replace with the actual folder path
process_folder(folder_path)
