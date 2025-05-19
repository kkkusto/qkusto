import pandas as pd
import networkx as nx
import matplotlib.pyplot as plt

# === CONFIG: path to your single CSV ===
JOBS_CSV = 'jobs.csv'  # columns: job,before,after

# === STEP 1: load CSV ===
# 'before' and 'after' can be empty cells if not applicable
df = pd.read_csv(JOBS_CSV, dtype=str).fillna('')

# === STEP 2: build directed graph ===
G = nx.DiGraph()
for _, row in df.iterrows():
    job = row['job'].strip()
    before = row['before'].strip()
    after = row['after'].strip()
    # add the job node even if it has no deps
    G.add_node(job)
    # before → job
    if before:
        G.add_edge(before, job)
    # job → after
    if after:
        G.add_edge(job, after)

# === STEP 3: layout (hierarchical if possible) ===
try:
    pos = nx.nx_agraph.graphviz_layout(G, prog='dot')
except (ImportError, nx.NetworkXException):
    pos = nx.spring_layout(G)

# === STEP 4: draw ===
plt.figure(figsize=(12, 8))
nx.draw(
    G,
    pos,
    with_labels=True,
    arrows=True,
    node_size=2000,
    font_size=10,
    arrowstyle='-|>',
    arrowsize=12
)
plt.title('Job Flow Diagram')
plt.axis('off')
plt.tight_layout()
plt.show()
