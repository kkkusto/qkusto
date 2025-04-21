job_dict = {
    'job1': ['dep1'],
    'job2': []
}

# Step 1: Count dependencies
dependency_counts = {job: len(deps) for job, deps in job_dict.items()}

# Step 2: Sort by number of dependencies (descending)
sorted_jobs = sorted(dependency_counts.items(), key=lambda x: x[1], reverse=True)

# Step 3: Generate Mermaid flow (least dependent first)
mermaid_lines = ["graph TD"]
for job, _ in reversed(sorted_jobs):  # least dependent first
    for dep in job_dict[job]:
        mermaid_lines.append(f"    {dep} --> {job}")
    if not job_dict[job]:  # Show job if no deps
        mermaid_lines.append(f"    {job}")

# Output Mermaid code
mermaid_code = "\n".join(mermaid_lines)
print(mermaid_code)
