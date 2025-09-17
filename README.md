import pandas as pd
import cx_Oracle  # or oracledb
import json

# --- Oracle connection ---
dsn = cx_Oracle.makedsn("host", 1521, service_name="service")
conn = cx_Oracle.connect(user="username", password="password", dsn=dsn)

# Define which PARM_NAMEs you want as top-level columns
KEY_COLUMNS = {"threshold", "retries", "timeout"}   # <-- customize this list

def fetch_job_params(proj_id, dept_id, job_id):
    query = """
        SELECT PARM_NAME, PARM_VAL, SUB_PARM_TYPE
        FROM job_parameter_detail
        WHERE proj_id = :proj_id
          AND dept_id = :dept_id
          AND job_id  = :job_id
    """
    df = pd.read_sql(query, conn, params={"proj_id": proj_id, "dept_id": dept_id, "job_id": job_id})
    return df

def process_job(proj_id, dept_id, job_id, base_row):
    df = fetch_job_params(proj_id, dept_id, job_id)

    # If child jobs exist → recurse
    if "childName" in df["PARM_NAME"].values:
        results = []
        child_rows = df[df["PARM_NAME"] == "childName"]

        for _, row in child_rows.iterrows():
            if row["SUB_PARM_TYPE"].startswith("Child"):
                new_job_id = row["PARM_VAL"]
                results.extend(process_job(proj_id, dept_id, new_job_id, base_row))
        return results
    else:
        # Leaf job → split into key columns and json dict
        record = {}
        extra_json = {}

        for _, row in df.iterrows():
            parm = row["PARM_NAME"]
            val = row["PARM_VAL"]

            if parm in KEY_COLUMNS:
                record[parm] = val
            else:
                extra_json[parm] = val

        # Attach metadata from first sheet
        record.update(base_row.to_dict())
        record["leaf_job_id"] = job_id
        record["other_params_json"] = json.dumps(extra_json, ensure_ascii=False)

        return [record]

# --- Main ---
excel_df = pd.read_excel("input.xlsx")

all_results = []

for idx, row in excel_df.iterrows():
    try:
        proj_id, dept_id, job_id = row["proj_id_dept_job"].split()
        results = process_job(proj_id, dept_id, job_id, row)
        all_results.extend(results)
    except Exception as e:
        print(f"Error processing row {idx}: {e}")

# Convert to DataFrame
final_df = pd.DataFrame(all_results)

# Save to Excel
final_df.to_excel("output.xlsx", index=False)
