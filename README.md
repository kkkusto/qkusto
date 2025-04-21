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



from collections import defaultdict

job_dict = {
    'job1': ['dep1'],
    'job2': []
}

# Count dependencies
dependency_counts = {job: len(deps) for job, deps in job_dict.items()}

# Sort jobs by number of dependencies (desc)
sorted_jobs = sorted(dependency_counts.items(), key=lambda x: x[1], reverse=True)

# Take least dependent jobs first
jobs_in_order = [job for job, _ in reversed(sorted_jobs)]

# Unique nodes
all_nodes = set(jobs_in_order)
for deps in job_dict.values():
    all_nodes.update(deps)
all_nodes = list(all_nodes)

# Assign positions for rendering (just vertically stacked)
node_positions = {node: (100, i * 100 + 50) for i, node in enumerate(all_nodes)}

# Start building HTML + SVG
svg_elements = []

# Draw nodes
for node, (x, y) in node_positions.items():
    svg_elements.append(f'<circle cx="{x}" cy="{y}" r="20" fill="lightblue" />')
    svg_elements.append(f'<text x="{x}" y="{y+5}" text-anchor="middle" font-size="12">{node}</text>')

# Draw edges
for job, deps in job_dict.items():
    x1, y1 = node_positions[job]
    for dep in deps:
        x0, y0 = node_positions[dep]
        svg_elements.append(
            f'<line x1="{x0}" y1="{y0}" x2="{x1}" y2="{y1}" stroke="black" marker-end="url(#arrow)" />'
        )

# Wrap in HTML
html_code = f"""
<!DOCTYPE html>
<html>
<head>
    <title>Job Flow</title>
</head>
<body>
    <svg width="800" height="{len(all_nodes)*120}">
        <defs>
            <marker id="arrow" markerWidth="10" markerHeight="7" refX="10" refY="3.5"
                    orient="auto">
                <polygon points="0 0, 10 3.5, 0 7" fill="black" />
            </marker>
        </defs>
        {''.join(svg_elements)}
    </svg>
</body>
</html>
"""

# Save to file
with open("job_flow.html", "w") as f:
    f.write(html_code)

print("✅ HTML flow chart saved to job_flow.html")
