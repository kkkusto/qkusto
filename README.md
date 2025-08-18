
import os
import re
import json

def parse_sql_hql_files(folder_path):
    parsed_data = []

    # Regex patterns for DDL statements
    DDL_PATTERNS = {
        'CREATE_TABLE': re.compile(r'CREATE\s+(?:EXTERNAL\s+)?TABLE\s+IF\s+NOT\s+EXISTS\s+([^\s(]+)', re.IGNORECASE),
        'ALTER_TABLE': re.compile(r'ALTER\s+TABLE\s+([^\s]+)', re.IGNORECASE),
        'DROP_TABLE': re.compile(r'DROP\s+(?:EXTERNAL\s+)?TABLE\s+IF\s+EXISTS\s+([^\s]+)', re.IGNORECASE),
        'CREATE_VIEW': re.compile(r'CREATE\s+VIEW\s+IF\s+NOT\s+EXISTS\s+([^\s(]+)', re.IGNORECASE),
        'DROP_VIEW': re.compile(r'DROP\s+VIEW\s+IF\s+EXISTS\s+([^\s]+)', re.IGNORECASE),
    }

    # Regex patterns for DML statements
    DML_PATTERNS = {
        'INSERT': re.compile(r'INSERT\s+(?:INTO\s+)?([^\s(]+)', re.IGNORECASE),
        'UPDATE': re.compile(r'UPDATE\s+([^\s]+)', re.IGNORECASE),
        'DELETE': re.compile(r'DELETE\s+FROM\s+([^\s]+)', re.IGNORECASE),
        'SELECT': re.compile(r'FROM\s+([^\s;\n]+)', re.IGNORECASE), # This is a simplified regex for SELECT to get tables after FROM
    }

    for root, _, files in os.walk(folder_path):
        for file_name in files:
            if file_name.endswith(('.sql', '.hql')):
                file_path = os.path.join(root, file_name)
                with open(file_path, 'r') as f:
                    content = f.read()

                # Split content into individual statements (basic splitting by semicolon)
                statements = [s.strip() for s in content.split(';') if s.strip()]

                for statement in statements:
                    statement_type = 'UNKNOWN'
                    category = 'UNKNOWN'
                    table_name = 'UNKNOWN'

                    # Check for DDL
                    for ddl_type, pattern in DDL_PATTERNS.items():
                        match = pattern.search(statement)
                        if match:
                            statement_type = ddl_type
                            category = 'DDL'
                            table_name = match.group(1).strip().split('.')[0] # Get the first part of the table name
                            break

                    # Check for DML if not DDL
                    if category == 'UNKNOWN':
                        for dml_type, pattern in DML_PATTERNS.items():
                            match = pattern.search(statement)
                            if match:
                                statement_type = dml_type
                                category = 'DML'
                                # For SELECT, we might get multiple tables, for simplicity, taking the first one
                                if dml_type == 'SELECT':
                                    # This is a very basic attempt to get table names from SELECT. 
                                    # A full-fledged SQL parser would be needed for accurate SELECT parsing.
                                    tables_found = re.findall(r'FROM\s+([^\s;\n,]+)|JOIN\s+([^\s;\n,]+)', statement, re.IGNORECASE)
                                    if tables_found:
                                        table_name = ', '.join([t[0] or t[1] for t in tables_found if t[0] or t[1]])
                                else:
                                    table_name = match.group(1).strip().split('.')[0]
                                break

                    parsed_data.append({
                        'table': table_name,
                        'statement_type': statement_type,
                        'category': category,
                        'statement': statement,
                        'source_file': file_name,
                        'source_folder': os.path.basename(root)
                    })
    return parsed_data

if __name__ == '__main__':
    # Example usage: Replace 'sql_files' with your folder path
    # For testing, create a folder named 'sql_files' and put some .sql or .hql files inside.
    # Example:
    # sql_files/
    #   -- create_table.sql
    #   CREATE TABLE my_table (id INT, name STRING);
    #   -- insert_data.sql
    #   INSERT INTO my_table VALUES (1, 'test');
    #   -- select_data.hql
    #   SELECT * FROM another_table WHERE id = 1;

    # Create a dummy folder and files for demonstration
    if not os.path.exists('sql_files'):
        os.makedirs('sql_files')
    
    with open('sql_files/create_table.sql', 'w') as f:
        f.write('CREATE TABLE IF NOT EXISTS my_database.my_table (id INT, name STRING);\n')
        f.write('ALTER TABLE my_database.my_table ADD COLUMNS (age INT);\n')
    
    with open('sql_files/insert_data.sql', 'w') as f:
        f.write('INSERT INTO my_database.my_table VALUES (1, \'test\');\n')
        f.write('UPDATE my_database.my_table SET name = \'new_test\' WHERE id = 1;\n')
    
    with open('sql_files/select_data.hql', 'w') as f:
        f.write('SELECT a.col1, b.col2 FROM my_database.table_a a JOIN my_database.table_b b ON a.id = b.id;\n')
        f.write('DROP VIEW IF EXISTS my_view;\n')

    results = parse_sql_hql_files('sql_files')
    print(json.dumps(results, indent=4))

    # Clean up dummy files and folder
    os.remove('sql_files/create_table.sql')
    os.remove('sql_files/insert_data.sql')
    os.remove('sql_files/select_data.hql')
    os.rmdir('sql_files')


===============



import os
import re
import json
import csv

def parse_sql_hql_files(folder_path):
    parsed_data = []

    # Regex patterns for DDL statements
    DDL_PATTERNS = {
        'CREATE_TABLE': re.compile(r'CREATE\s+(?:EXTERNAL\s+)?TABLE\s+IF\s+NOT\s+EXISTS\s+([^\s(]+)', re.IGNORECASE),
        'ALTER_TABLE': re.compile(r'ALTER\s+TABLE\s+([^\s]+)', re.IGNORECASE),
        'DROP_TABLE': re.compile(r'DROP\s+(?:EXTERNAL\s+)?TABLE\s+IF\s+EXISTS\s+([^\s]+)', re.IGNORECASE),
        'CREATE_VIEW': re.compile(r'CREATE\s+VIEW\s+IF\s+NOT\s+EXISTS\s+([^\s(]+)', re.IGNORECASE),
        'DROP_VIEW': re.compile(r'DROP\s+VIEW\s+IF\s+EXISTS\s+([^\s]+)', re.IGNORECASE),
    }

    # Regex patterns for DML statements
    DML_PATTERNS = {
        'INSERT': re.compile(r'INSERT\s+(?:INTO\s+)?([^\s(]+)', re.IGNORECASE),
        'UPDATE': re.compile(r'UPDATE\s+([^\s]+)', re.IGNORECASE),
        'DELETE': re.compile(r'DELETE\s+FROM\s+([^\s]+)', re.IGNORECASE),
        'SELECT': re.compile(r'FROM\s+([^\s;\n]+)', re.IGNORECASE), # This is a simplified regex for SELECT to get tables after FROM
    }

    for root, _, files in os.walk(folder_path):
        for file_name in files:
            if file_name.endswith(('.sql', '.hql')):
                file_path = os.path.join(root, file_name)
                with open(file_path, 'r') as f:
                    content = f.read()

                # Split content into individual statements (basic splitting by semicolon)
                statements = [s.strip() for s in content.split(';') if s.strip()]

                for statement in statements:
                    statement_type = 'UNKNOWN'
                    category = 'UNKNOWN'
                    table_name = 'UNKNOWN'

                    # Check for DDL
                    for ddl_type, pattern in DDL_PATTERNS.items():
                        match = pattern.search(statement)
                        if match:
                            statement_type = ddl_type
                            category = 'DDL'
                            table_name = match.group(1).strip().split('.')[0] # Get the first part of the table name
                            break

                    # Check for DML if not DDL
                    if category == 'UNKNOWN':
                        for dml_type, pattern in DML_PATTERNS.items():
                            match = pattern.search(statement)
                            if match:
                                statement_type = dml_type
                                category = 'DML'
                                # For SELECT, we might get multiple tables, for simplicity, taking the first one
                                if dml_type == 'SELECT':
                                    # This is a very basic attempt to get table names from SELECT. 
                                    # A full-fledged SQL parser would be needed for accurate SELECT parsing.
                                    tables_found = re.findall(r'FROM\s+([^\s;\n,]+)|JOIN\s+([^\s;\n,]+)', statement, re.IGNORECASE)
                                    if tables_found:
                                        table_name = ', '.join([t[0] or t[1] for t in tables_found if t[0] or t[1]])
                                else:
                                    table_name = match.group(1).strip().split('.')[0]
                                break

                    parsed_data.append({
                        'table': table_name,
                        'statement_type': statement_type,
                        'category': category,
                        'statement': statement,
                        'source_file': file_name,
                        'source_folder': os.path.basename(root)
                    })
    return parsed_data

def write_to_csv(data, folder_path):
    if not data:
        print("No data to write to CSV.")
        return

    folder_name = os.path.basename(os.path.normpath(folder_path))
    csv_file_name = f"{folder_name}_parsed_stmts.csv"
    csv_file_path = os.path.join(os.getcwd(), csv_file_name)

    keys = data[0].keys()
    with open(csv_file_path, 'w', newline='') as output_file:
        dict_writer = csv.DictWriter(output_file, keys)
        dict_writer.writeheader()
        dict_writer.writerows(data)
    print(f"Data successfully written to {csv_file_path}")

if __name__ == '__main__':
    # Example usage: Replace 'sql_files' with your folder path
    # For testing, create a folder named 'sql_files' and put some .sql or .hql files inside.
    # Example:
    # sql_files/
    #   -- create_table.sql
    #   CREATE TABLE my_table (id INT, name STRING);
    #   -- insert_data.sql
    #   INSERT INTO my_table VALUES (1, 'test');
    #   -- select_data.hql
    #   SELECT * FROM another_table WHERE id = 1;

    # Create a dummy folder and files for demonstration
    dummy_folder = 'sql_files'
    if not os.path.exists(dummy_folder):
        os.makedirs(dummy_folder)
    
    with open(os.path.join(dummy_folder, 'create_table.sql'), 'w') as f:
        f.write('CREATE TABLE IF NOT EXISTS my_database.my_table (id INT, name STRING);\n')
        f.write('ALTER TABLE my_database.my_table ADD COLUMNS (age INT);\n')
    
    with open(os.path.join(dummy_folder, 'insert_data.sql'), 'w') as f:
        f.write('INSERT INTO my_database.my_table VALUES (1, \'test\');\n')
        f.write('UPDATE my_database.my_table SET name = \'new_test\' WHERE id = 1;\n')
    
    with open(os.path.join(dummy_folder, 'select_data.hql'), 'w') as f:
        f.write('SELECT a.col1, b.col2 FROM my_database.table_a a JOIN my_database.table_b b ON a.id = b.id;\n')
        f.write('DROP VIEW IF EXISTS my_view;\n')

    results = parse_sql_hql_files(dummy_folder)
    write_to_csv(results, dummy_folder)

    # Clean up dummy files and folder
    os.remove(os.path.join(dummy_folder, 'create_table.sql'))
    os.remove(os.path.join(dummy_folder, 'insert_data.sql'))
    os.remove(os.path.join(dummy_folder, 'select_data.hql'))
    os.rmdir(dummy_folder)

