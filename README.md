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
