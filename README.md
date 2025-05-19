import pandas as pd
import networkx as nx
import matplotlib.pyplot as plt

# 'dataset' is the DataFrame Power BI passes you
df = dataset.fillna('')

G = nx.DiGraph()
for _, row in df.iterrows():
    if row['before']:
        G.add_edge(row['before'], row['job'])
    if row['after']:
        G.add_edge(row['job'], row['after'])

pos = nx.nx_agraph.graphviz_layout(G, prog='dot')  # requires pygraphviz
plt.figure(figsize=(8, 6))
nx.draw(G, pos, with_labels=True, arrows=True, node_size=1500, font_size=8)
plt.axis('off')
plt.show()
