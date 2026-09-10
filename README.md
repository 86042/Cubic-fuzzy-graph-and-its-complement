import networkx as nx
import matplotlib.pyplot as plt
import itertools


G = nx.Graph()


V = {
    'v1': ((0.70, 0.90), 0.80),
    'v2': ((0.62, 0.84), 0.74),
    'v3': ((0.55, 0.79), 0.68),
    'v4': ((0.66, 0.88), 0.77),
    'v5': ((0.73, 0.92), 0.84),
    'v6': ((0.58, 0.81), 0.70)
}

for v, val in V.items():
    G.add_node(v, cubic=val)

E = {
    ('v1','v2'):((0.72,0.90),0.81),
    ('v1','v3'):((0.65,0.82),0.72),
    ('v1','v5'):((0.61,0.80),0.69),
    ('v1','v6'):((0.64,0.81),0.71),
    ('v2','v3'):((0.69,0.87),0.76),
    ('v2','v4'):((0.63,0.82),0.71),
    ('v2','v6'):((0.59,0.77),0.65),
    ('v3','v4'):((0.75,0.91),0.84),
    ('v3','v5'):((0.60,0.78),0.67),
    ('v4','v5'):((0.71,0.89),0.80),
    ('v4','v6'):((0.66,0.84),0.74),
    ('v5','v6'):((0.73,0.90),0.82)
}

for (u,v), cf in E.items():

    (l,upr), lam = cf

    score = ((1-l)+upr+lam)/3

    G.add_edge(
        u,
        v,
        cubic=cf,
        score=score,
        weight=score
    )

def dominating(graph,D):

    dominated=set(D)

    for u in D:
        dominated.update(graph.neighbors(u))

    return dominated==set(graph.nodes())


def minimum_DS(graph):

    nodes=list(graph.nodes())

    for r in range(1,len(nodes)+1):

        for subset in itertools.combinations(nodes,r):

            if dominating(graph,subset):

                return set(subset)

    return set()


def domination_score(graph,DS):

    visited=set()

    score=0

    for u in DS:

        for v in graph.neighbors(u):

            e=tuple(sorted((u,v)))

            if e not in visited:

                visited.add(e)

                score+=graph[u][v]["score"]

    return score

DS = minimum_DS(G)

DN = domination_score(G,DS)

mst = nx.minimum_spanning_tree(G,weight="weight")

mst_weight = sum(
    G[u][v]["weight"]
    for u,v in mst.edges()
)

print("\n==============================")
print("ORIGINAL CUBIC FUZZY GRAPH")
print("==============================")
print("Dominating Set :",DS)
print("Domination Number :",round(DN,3))
print("MST Weight :",round(mst_weight,3))


Gc = nx.complement(G)


for node in G.nodes():
    Gc.nodes[node]["cubic"] = G.nodes[node]["cubic"]

for u, v in Gc.edges():

    # Vertex cubic fuzzy values
    (Lu, Uu), Lamu = G.nodes[u]["cubic"]
    (Lv, Uv), Lamv = G.nodes[v]["cubic"]

    # Complement edge values
    comp_lower = min(Lu, Lv)
    comp_upper = min(Uu, Uv)
    comp_lambda = min(Lamu, Lamv)

    # Edge score
    score = ((1 - comp_lower) + comp_upper + comp_lambda) / 3

    Gc[u][v]["cubic"] = (
        (comp_lower, comp_upper),
        comp_lambda
    )

    Gc[u][v]["score"] = score
    Gc[u][v]["weight"] = score

DSc = minimum_DS(Gc)

DNc = domination_score(Gc, DSc)


forest = nx.minimum_spanning_tree(
    Gc,
    weight="weight"
)

forest_weight = sum(
    Gc[u][v]["weight"]
    for u, v in forest.edges()
)

print("\n==============================")
print("COMPLEMENT CUBIC FUZZY GRAPH")
print("==============================")
print("Dominating Set :", DSc)
print("Domination Number :", round(DNc,3))
print("Minimum Spanning Forest Weight :", round(forest_weight,3))


plt.figure(figsize=(9,9))

pos = nx.circular_layout(G)

labels={
n:f"{n}\n[{d['cubic'][0][0]:.2f},{d['cubic'][0][1]:.2f}]\nλ={d['cubic'][1]:.2f}"
for n,d in G.nodes(data=True)
}

edge_labels={
(u,v):f"{d['score']:.2f}"
for u,v,d in G.edges(data=True)
}

colors=[
"red" if n in DS else "lightblue"
for n in G.nodes()
]

nx.draw_networkx_nodes(
G,pos,
node_color=colors,
node_size=1700
)

nx.draw_networkx_labels(
G,pos,
labels,
font_size=8
)

nx.draw_networkx_edges(
G,
pos,
edgelist=list(set(G.edges())-set(mst.edges())),
width=2
)

nx.draw_networkx_edges(
G,
pos,
edgelist=mst.edges(),
edge_color="blue",
width=4
)

nx.draw_networkx_edge_labels(
G,
pos,
edge_labels=edge_labels,
font_size=7
)

plt.title("Original Cubic Fuzzy Graph",fontsize=15,fontweight="bold")

info = (
f"DS : {DS}\n"
f"DN : {DN:.2f}\n"
f"MST : {mst_weight:.2f}"
)

plt.gca().text(
0.98,
0.98,
info,
transform=plt.gca().transAxes,
ha="right",
va="top",
fontsize=10,
bbox=dict(facecolor="white",edgecolor="black")
)

plt.axis("off")

plt.figure(figsize=(8,8))

pos2 = nx.circular_layout(Gc)

colors=[
"red" if n in DSc else "lightgreen"
for n in Gc.nodes()
]

edge_labels = {
(u,v):
f"[{d['cubic'][0][0]:.2f},{d['cubic'][0][1]:.2f}]"
"\n"
f"λ={d['cubic'][1]:.2f}"
"\n"
f"S={d['score']:.2f}"
for u,v,d in Gc.edges(data=True)
}

nx.draw_networkx_nodes(
Gc,
pos2,
node_color=colors,
node_size=1700
)

labels = {
n:
f"{n}\n"
f"[{Gc.nodes[n]['cubic'][0][0]:.2f},{Gc.nodes[n]['cubic'][0][1]:.2f}]"
"\n"
f"λ={Gc.nodes[n]['cubic'][1]:.2f}"
for n in Gc.nodes()
}

nx.draw_networkx_labels(
Gc,
pos2,
labels=labels,
font_size=8
)

nx.draw_networkx_edges(
    Gc,
    pos2,
    edgelist=list(set(Gc.edges())-set(forest.edges())),
    width=2,
    connectionstyle="arc3,rad=0.35"
)

nx.draw_networkx_edges(
    Gc,
    pos2,
    edgelist=forest.edges(),
    edge_color="blue",
    width=4,
    connectionstyle="arc3,rad=0.35"
)

nx.draw_networkx_edge_labels(
    Gc,
    pos2,
    edge_labels=edge_labels,
    font_size=7,
    label_pos=0.18,
    rotate=False,
    bbox=dict(
        facecolor="white",
        edgecolor="none",
        alpha=0.90
    )
)

plt.title("Complement Cubic Fuzzy Graph",fontsize=15,fontweight="bold")

info = (
f"DS : {DSc}\n"
f"DN : {DNc:.2f}\n"
f"MSF : {forest_weight:.2f}"
)

plt.gca().text(
0.98,
0.98,
info,
transform=plt.gca().transAxes,
ha="right",
va="top",
fontsize=10,
bbox=dict(facecolor="white",edgecolor="black")
)

plt.axis("off")

plt.show()
