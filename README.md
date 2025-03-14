import pandas as pd

def process_text_file(file_path, error_file):
    unique_entries = set()
    
    with open(file_path, 'r', encoding='utf-8') as file, open(error_file, 'w', encoding='utf-8') as err_file:
        for line in file:
            line = line.strip()
            if not line:
                continue
            
            try:
                parts = line.split(':', 1)
                if len(parts) < 2:
                    raise ValueError("Invalid format: missing ':'")
                
                first_word = parts[0].split()[0]  # Extract the first word
                split_first_word = first_word.split('/')  # Split by '/'
                
                # Convert list to tuple to make it hashable for a set
                unique_entries.add((first_word, ','.join(split_first_word)))
            except Exception as e:
                err_file.write(line + '\n')
    
    # Convert set to DataFrame
    df = pd.DataFrame(unique_entries, columns=['First_Word', 'Split_Word'])
    
    # Display DataFrame
    import ace_tools as tools
    tools.display_dataframe_to_user(name="Processed Data", dataframe=df)

# Example usage
file_path = 'input.txt'  # Change this to your actual file path
error_file = 'error_lines.txt'  # File to store erroneous lines
process_text_file(file_path, error_file)
