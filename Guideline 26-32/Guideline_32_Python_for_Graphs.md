# Guideline 32: Python for Graphs

A **graph**, in the mathematical sense this guideline means, has nothing to do with a bar chart or a line plot — it's a structure of **nodes** (also called vertices) connected by **edges**. A social network, a road map, a family tree, and the layers of a neural network are all graphs wearing different clothes. Two libraries do the heavy lifting in Python: **Graphviz**, which specializes in turning a graph's structure into a clean diagram, and **NetworkX**, which specializes in analyzing a graph's structure — computing paths, distances, and properties. This guideline uses both.

```python
pip install graphviz networkx matplotlib
```

## 32.1 Graphviz

Graphviz is actually two things bundled together: a diagramming *language* called DOT, and separate rendering software that turns DOT descriptions into images. The `graphviz` Python package is a thin, convenient wrapper that lets you build a DOT description with Python code instead of writing DOT syntax directly.

```python
from graphviz import Digraph

dot = Digraph()
dot.node("A")
dot.node("B")
dot.node("C")

dot.edge("A", "B")
dot.edge("A", "C")

dot.render("my_graph", format="png", view=True)
```

| Method | What it does |
|---|---|
| `Digraph()` / `Graph()` | Creates a directed / undirected graph object |
| `node(name, **attrs)` | Adds a single node, optionally with style attributes |
| `edge(a, b, **attrs)` | Adds a connection between two nodes |
| `edges([(a, b), ...])` | Adds several edges at once from a list of pairs |
| `attr(**kwargs)` | Sets default styling applied to every node or edge that follows |
| `render(filename, format=, view=)` | Writes an image file, optionally opening it immediately |

## 32.2 Undirected and Directed Graphs

The difference is exactly what it sounds like: an **undirected** edge is a mutual connection with no inherent direction — if A is connected to B, B is equally connected to A. A **directed** edge points one way, and the reverse connection doesn't automatically exist.

A friendship on most social platforms is undirected — mutual by definition. A "follows" relationship (Twitter/X-style) is directed — you can follow someone who doesn't follow you back.

```python
from graphviz import Graph, Digraph

# Undirected: friendships
friends = Graph()
friends.edge("Alice", "Bob")
friends.edge("Bob", "Carol")

# Directed: who follows whom
follows = Digraph()
follows.edge("Alice", "Bob")     # Alice follows Bob
follows.edge("Bob", "Alice")     # Bob also follows Alice back
follows.edge("Carol", "Bob")     # Carol follows Bob (not necessarily mutual)
```

Graphviz draws these differently without any extra work on your part: `Graph` edges render as plain lines, while `Digraph` edges render as arrows pointing from the first node to the second — the visual distinction matches the structural one.

## 32.3 Coloring Graphs

**Graph coloring** assigns a color to every node so that no two *connected* nodes share the same color — classically framed as the map-coloring problem (no two neighboring countries the same color), but it applies anywhere two connected things must be kept visually or logically distinct, like scheduling exams so no student has two exams with the same time slot.

A simple greedy approach: go through the nodes one at a time, and give each one the first color not already used by one of its neighbors.

```python
def greedy_coloring(graph):
    # graph: a dict mapping each node to a list of its neighbors
    colors = {}
    for node in graph:
        used = {colors[neighbor] for neighbor in graph[node] if neighbor in colors}
        color = 0
        while color in used:
            color += 1
        colors[node] = color
    return colors

graph = {
    "A": ["B", "C"],
    "B": ["A", "C"],
    "C": ["A", "B", "D"],
    "D": ["C"],
}

result = greedy_coloring(graph)
print(result)   # e.g. {'A': 0, 'B': 1, 'C': 2, 'D': 0}
```

The numbers are color *indices*, not colors themselves — mapping them to actual colors and feeding that into Graphviz makes the result visible:

```python
palette = ["lightblue", "lightgreen", "salmon", "khaki"]

dot = Graph()
for node, color_index in result.items():
    dot.node(node, style="filled", fillcolor=palette[color_index])
for node, neighbors in graph.items():
    for neighbor in neighbors:
        dot.edge(node, neighbor)

dot.render("colored_graph", format="png", view=True)
```

This greedy method doesn't always use the fewest possible colors — finding the true minimum (the graph's **chromatic number**) is a much harder problem — but it's simple, fast, and good enough for most practical scheduling and layout problems.

## 32.4 Trees

A **tree** is a specific, restricted kind of graph: connected (every node is reachable from every other), and acyclic (no loops — there's exactly one path between any two nodes). Trees have their own vocabulary: the **root** is the single node everything grows from, a node's **children** are the nodes directly below it, its **parent** is the node directly above, and a **leaf** is a node with no children.

Because a tree has no loops to worry about, Graphviz renders one cleanly with no extra effort — just build it like any other `Digraph`:

```python
family_tree = Digraph()
family_tree.edge("Grandparent", "Parent A")
family_tree.edge("Grandparent", "Parent B")
family_tree.edge("Parent A", "Child 1")
family_tree.edge("Parent A", "Child 2")
family_tree.edge("Parent B", "Child 3")

family_tree.render("family_tree", format="png", view=True)
```

A **binary tree** — the specific case where every node has at most two children — is the shape behind binary search, heaps (from Chapter 3 of *AI in Everything*, if you've read it), and a great deal of efficient searching and sorting:

```python
binary = Digraph()
binary.edge("8", "3")
binary.edge("8", "10")
binary.edge("3", "1")
binary.edge("3", "6")
binary.edge("10", "14")
```

## 32.5 NetworkX

Where Graphviz's strength is drawing a graph, **NetworkX**'s strength is *analyzing* one — measuring distances, checking connectivity, and running well-known graph algorithms without implementing them yourself.

```python
import networkx as nx
import matplotlib.pyplot as plt

G = nx.Graph()
G.add_node("A")
G.add_edges_from([("A", "B"), ("B", "C"), ("A", "C"), ("C", "D")])

nx.draw(G, with_labels=True, node_color="lightblue", node_size=800)
plt.show()
```

A `DiGraph()` gives you the directed equivalent, exactly like Graphviz's `Digraph` vs `Graph` split. Some methods worth knowing immediately:

| Method | What it does |
|---|---|
| `add_node(n)` / `add_nodes_from([...])` | Adds one node / several at once |
| `add_edge(a, b)` / `add_edges_from([...])` | Adds one edge / several at once |
| `G.nodes()` / `G.edges()` | Lists all nodes / edges |
| `G.degree(n)` | How many edges connect to node `n` |
| `nx.shortest_path(G, source, target)` | The shortest path between two nodes |
| `nx.is_connected(G)` | Whether every node can reach every other |

```python
print(nx.shortest_path(G, "A", "D"))   # ['A', 'C', 'D']
print(G.degree("C"))                    # 3
```

## 32.6 Advanced Graphs & Trees

Real-world graphs are often **weighted** — each edge carries a cost, distance, or strength, not just a bare connection. A road map is the natural example: the edges aren't just "these two towns are connected," they're "these two towns are 40 kilometers apart."

```python
roads = nx.Graph()
roads.add_edge("Town A", "Town B", weight=40)
roads.add_edge("Town B", "Town C", weight=15)
roads.add_edge("Town A", "Town C", weight=60)

shortest = nx.shortest_path(roads, "Town A", "Town C", weight="weight")
distance = nx.shortest_path_length(roads, "Town A", "Town C", weight="weight")
print(shortest, distance)   # ['Town A', 'Town B', 'Town C'] 55
```

**Traversal** — visiting every reachable node, systematically — comes in two classic flavors. Breadth-first search (BFS) explores level by level, outward from the start; depth-first search (DFS) follows one path as deep as it goes before backing up. NetworkX builds a traversal tree for either directly:

```python
bfs_tree = nx.bfs_tree(roads, source="Town A")
dfs_tree = nx.dfs_tree(roads, source="Town A")
```

A **minimum spanning tree** connects every node in a weighted graph using the smallest possible total edge weight, with no cycles — the cheapest way to make sure everything is still reachable:

```python
mst = nx.minimum_spanning_tree(roads)
print(list(mst.edges(data=True)))
```

## 32.7 Neural Networks

A neural network — the subject of several later guidelines — is, structurally, nothing more than a directed, weighted, *layered* graph: each node is a neuron, each edge is a connection carrying a weight, and the layers restrict which nodes are allowed to connect to which. Building small ones with NetworkX is a genuinely useful way to see that structure before ever training a real one.

```python
import networkx as nx
import matplotlib.pyplot as plt

nn = nx.DiGraph()

input_layer = ["x1", "x2", "x3"]
hidden_layer = ["h1", "h2"]
output_layer = ["y"]

for i in input_layer:
    for h in hidden_layer:
        nn.add_edge(i, h)

for h in hidden_layer:
    for o in output_layer:
        nn.add_edge(h, o)

# position each layer in its own vertical column
pos = {}
for i, node in enumerate(input_layer):
    pos[node] = (0, i)
for i, node in enumerate(hidden_layer):
    pos[node] = (1, i + 0.5)
for i, node in enumerate(output_layer):
    pos[node] = (2, i + 1)

nx.draw(nn, pos, with_labels=True, node_color="lightyellow", node_size=1000, arrows=True)
plt.show()
```

Every connection here could just as easily carry a `weight` attribute (32.6), and every node could hold whatever value it computes — at that point, this diagram *is* a neural network's architecture, laid out exactly the way it would appear in a paper or a textbook figure. The graph doesn't compute anything on its own; it's the map that the actual math — covered when this guidebook reaches machine learning — runs on top of.

---

Everything in this guideline shares one lens: nodes and the connections between them. A colored map, a family tree, a road network, and a neural network's layers are the same underlying structure, described with the same handful of tools, adapted to whatever the connections happen to represent.
