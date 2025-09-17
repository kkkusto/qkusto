import pandas as pd
import cx_Oracle  # or use oracledb

# --- Oracle connection ---
dsn = cx_Oracle.makedsn("host", 1521, service_name="service")
conn = cx_Oracle.connect(user="username", password="password", dsn=dsn)

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
        # Leaf job → return dict of params + all columns from first sheet
        record = {row["PARM_NAME"]: row["PARM_VAL"] for _, row in df.iterrows()}
        # Keep original row metadata
        record.update(base_row.to_dict())
        # Add current identifiers (in case needed)
        record.update({
            "leaf_job_id": job_id
        })
        return [record]

# --- Main ---
excel_df = pd.read_excel("input.xlsx")

all_results = []

for idx, row in excel_df.iterrows():
    try:
        # Assuming your column is like "proj_id dept_id job_id"
        proj_id, dept_id, job_id = row["proj_id_dept_job"].split()
        results = process_job(proj_id, dept_id, job_id, row)
        all_results.extend(results)
    except Exception as e:
        print(f"Error processing row {idx}: {e}")

# Convert to DataFrame
final_df = pd.DataFrame(all_results)

# Save to Excel
final_df.to_excel("output.xlsx", index=False)
