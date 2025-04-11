import networkx as nx
import matplotlib.pyplot as plt

# Define jobs
jobs = [
    {"job_id": "A", "before": []},
    {"job_id": "B", "before": ["A"]},
    {"job_id": "C", "before": ["A"]},
    {"job_id": "D", "before": ["B", "C"]},
    {"job_id": "E", "before": ["D"]},
]

# Create directed graph
G = nx.DiGraph()

# Add edges
for job in jobs:
    for dep in job["before"]:
        G.add_edge(dep, job["job_id"])

# Draw the graph
plt.figure(figsize=(8, 6))
pos = nx.spring_layout(G)
nx.draw(G, pos, with_labels=True, node_size=2000, node_color='skyblue', arrows=True, font_size=12)
plt.title("Job Dependency Graph")
plt.show()


from pyvis.network import Network

# Create network
net = Network(notebook=True, directed=True)
net.barnes_hut()

# Add nodes and edges
for job in jobs:
    net.add_node(job["job_id"])
    for dep in job["before"]:
        net.add_edge(dep, job["job_id"])

# Show interactive visualization
net.show("job_dependencies.html")



import plotly.express as px
import pandas as pd

# Define jobs with hierarchical relationships
jobs = [
    {"job_id": "A", "before": []},
    {"job_id": "B", "before": ["A"]},
    {"job_id": "C", "before": ["A"]},
    {"job_id": "D", "before": ["B", "C"]},
    {"job_id": "E", "before": ["D"]},
]

# Convert dependencies to parent-child relationships
hierarchy = []
for job in jobs:
    if not job["before"]:  # Root node (A)
        hierarchy.append({"id": job["job_id"], "parent": ""})
    else:
        for parent in job["before"]:
            hierarchy.append({"id": job["job_id"], "parent": parent})

# Create DataFrame
df = pd.DataFrame(hierarchy)

# Create treemap
fig = px.treemap(
    df,
    path=['parent', 'id'],
    title="Job Dependencies (Treemap)",
    width=800,
    height=400
)
fig.update_traces(root_color="lightgrey")
fig.show()



import plotly.graph_objects as go

# Create network layout
G = nx.DiGraph()
for job in jobs:
    for dep in job["before"]:
        G.add_edge(dep, job["job_id"])

pos = nx.spring_layout(G)

# Plot with Plotly
edge_x, edge_y = [], []
for edge in G.edges():
    x0, y0 = pos[edge[0]]
    x1, y1 = pos[edge[1]]
    edge_x.extend([x0, x1, None])
    edge_y.extend([y0, y1, None])

fig = go.Figure(
    data=[
        go.Scatter(x=edge_x, y=edge_y, mode="lines", line=dict(color="gray")),
        go.Scatter(
            x=[pos[n][0] for n in G.nodes()],
            y=[pos[n][1] for n in G.nodes()],
            mode="markers+text",
            text=list(G.nodes()),
            marker=dict(size=30, color="skyblue"),
            textfont=dict(size=14)
        )
    ],
    layout=go.Layout(showlegend=False, title="Job Dependencies (Network Graph)")
)
fig.show()
