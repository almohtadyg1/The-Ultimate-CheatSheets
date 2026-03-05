# Graph Algorithms Cheat Sheet
### From Zero to Expert

---

## What is a Graph?

A **graph** G = (V, E) is a set of **vertices** (nodes) connected by **edges**. It is the most expressive and general data structure in computer science — networks, maps, social connections, dependencies, circuits, and the internet itself are all graphs.

```
Undirected Graph:          Directed Graph (Digraph):
   A --- B                    A ──► B
   |   / |                    ▲     |
   |  /  |                    |     ▼
   C --- D                    D ◄── C
```

---

## Graph Vocabulary

| Term | Definition |
|---|---|
| **Vertex / Node** | A fundamental unit of the graph |
| **Edge / Arc** | A connection between two vertices |
| **Directed** | Edges have direction (one-way) |
| **Undirected** | Edges have no direction (two-way) |
| **Weighted** | Edges carry a numeric cost/distance |
| **Degree** | Number of edges incident to a vertex |
| **In-degree / Out-degree** | Directed: edges coming in / going out |
| **Path** | A sequence of vertices connected by edges |
| **Cycle** | A path that starts and ends at the same vertex |
| **DAG** | Directed Acyclic Graph — directed, no cycles |
| **Connected** | Every vertex reachable from every other (undirected) |
| **Strongly Connected** | Every vertex reachable from every other (directed) |
| **Sparse Graph** | E ≈ O(V) — few edges |
| **Dense Graph** | E ≈ O(V²) — many edges |
| **Tree** | Connected acyclic undirected graph with V-1 edges |
| **Forest** | A set of disjoint trees |
| **Bipartite** | Vertices split into two sets; edges only cross sets |
| **Planar** | Can be drawn without edge crossings |

---

## Graph Representations

### Adjacency List
Each vertex stores a list of its neighbors. The standard representation.

```
Graph:  A—B, A—C, B—D, C—D

A → [B, C]
B → [A, D]
C → [A, D]
D → [B, C]
```

Space: **O(V + E)**. Best for sparse graphs (most real-world graphs).  
Neighbor iteration: O(degree). Edge lookup: O(degree).

### Adjacency Matrix
A V×V matrix where `M[i][j] = 1` (or weight) if edge (i,j) exists.

```
    A  B  C  D
A [ 0  1  1  0 ]
B [ 1  0  0  1 ]
C [ 1  0  0  1 ]
D [ 0  1  1  0 ]
```

Space: **O(V²)**. Edge lookup: O(1). Best for dense graphs or when fast edge queries are needed.

### Edge List
A flat list of all edges `(u, v, weight)`. Used in algorithms that iterate over all edges (Kruskal, Bellman-Ford).

```
[(A,B,4), (A,C,2), (B,D,5), (C,D,1)]
```

Space: **O(E)**. Neighbor iteration: O(E). Edge lookup: O(E).

---

## Complexity Master Table

| Algorithm | Time | Space | Graph Type |
|---|---|---|---|
| BFS | O(V + E) | O(V) | Any |
| DFS | O(V + E) | O(V) | Any |
| Topological Sort (DFS) | O(V + E) | O(V) | DAG |
| Kahn's Algorithm | O(V + E) | O(V) | DAG |
| Dijkstra (binary heap) | O((V+E) log V) | O(V) | Weighted, non-neg |
| Dijkstra (Fibonacci heap) | O(E + V log V) | O(V) | Weighted, non-neg |
| Bellman-Ford | O(VE) | O(V) | Weighted, neg ok |
| SPFA | O(kE) avg, O(VE) worst | O(V) | Weighted, neg ok |
| Floyd-Warshall | O(V³) | O(V²) | All-pairs |
| Johnson's Algorithm | O(V² log V + VE) | O(V²) | All-pairs, neg ok |
| Prim (binary heap) | O((V+E) log V) | O(V) | Undirected, weighted |
| Prim (Fibonacci heap) | O(E + V log V) | O(V) | Undirected, weighted |
| Kruskal | O(E log E) | O(V) | Undirected, weighted |
| Borůvka | O(E log V) | O(V) | Undirected, weighted |
| Kosaraju (SCC) | O(V + E) | O(V) | Directed |
| Tarjan (SCC) | O(V + E) | O(V) | Directed |
| Tarjan (Bridges/APs) | O(V + E) | O(V) | Undirected |
| Ford-Fulkerson | O(E · max_flow) | O(V) | Flow network |
| Edmonds-Karp | O(VE²) | O(V) | Flow network |
| Dinic's Algorithm | O(V²E) | O(V) | Flow network |
| Push-Relabel | O(V²E) | O(V) | Flow network |
| Hungarian Algorithm | O(V³) | O(V²) | Bipartite matching |
| Hopcroft-Karp | O(E√V) | O(V) | Bipartite matching |
| Hierholzer (Euler) | O(V + E) | O(V) | Eulerian graph |
| Hamiltonian Path (exact) | O(2^V · V²) | O(2^V · V) | Any |
| Chromatic Number | O(2^V · V) | O(2^V) | Any |
| A* | O(E log V) | O(V) | Weighted + heuristic |
| Bidirectional Dijkstra | O((V+E) log V) | O(V) | Weighted, non-neg |

---

## Traversal Algorithms

### Breadth-First Search (BFS)

Explores the graph **level by level** using a queue. Visits all neighbors of a node before going deeper. Guarantees the **shortest path** (by number of edges) in an unweighted graph.

```
Graph:          BFS from A:
A — B — E       Queue: [A]
|   |           Visit A → enqueue B, C
C — D           Queue: [B, C]
                Visit B → enqueue E, D
                Queue: [C, E, D]
                Visit C → D already queued
                Queue: [E, D]
                Visit E, D...

BFS order: A, B, C, E, D
Shortest path A→D: A→B→D (2 hops) or A→C→D (2 hops)
```

```python
from collections import deque

def bfs(graph, start):
    visited = {start}
    queue = deque([start])
    order = []
    while queue:
        node = queue.popleft()
        order.append(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
    return order

def bfs_shortest_path(graph, start, end):
    visited = {start: None}           # node → parent
    queue = deque([start])
    while queue:
        node = queue.popleft()
        if node == end:
            path = []
            while node is not None:
                path.append(node)
                node = visited[node]
            return path[::-1]
        for nb in graph[node]:
            if nb not in visited:
                visited[nb] = node
                queue.append(nb)
    return None                       # no path found
```

**Applications:** Shortest path (unweighted), level-order traversal, connected components, bipartite check, web crawlers, social network distance.

---

### Depth-First Search (DFS)

Explores as **deep as possible** before backtracking. Uses a stack (explicitly or via recursion).

```
Graph:          DFS from A (recursive):
A — B — E       Visit A
|   |             Visit B
C — D               Visit E (dead end, backtrack)
                    Visit D
                      Visit C (dead end, backtrack)
                    Done D
                  Done B
                Done A

DFS order: A, B, E, D, C
```

```python
def dfs_recursive(graph, node, visited=None):
    if visited is None:
        visited = set()
    visited.add(node)
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs_recursive(graph, neighbor, visited)
    return visited

def dfs_iterative(graph, start):
    visited, stack, order = set(), [start], []
    while stack:
        node = stack.pop()
        if node in visited:
            continue
        visited.add(node)
        order.append(node)
        stack.extend(reversed(graph[node]))   # reversed to match recursive order
    return order
```

**DFS timestamps** — recording discovery time `d[v]` and finish time `f[v]` is fundamental to many advanced algorithms.

**Edge classification in DFS:**
| Edge Type | Definition | Significance |
|---|---|---|
| **Tree edge** | Edge in DFS tree | Normal traversal |
| **Back edge** | Points to ancestor | Indicates a cycle |
| **Forward edge** | Points to descendant | Only in directed graphs |
| **Cross edge** | Points to unrelated node | Only in directed graphs |

**Applications:** Cycle detection, topological sort, SCC, articulation points, bridges, maze solving, backtracking.

---

## Topological Sort

Orders vertices of a **DAG** such that for every directed edge (u→v), u comes before v. Used to resolve dependencies.

```
DAG (build dependencies):
  compile_a → link → run
  compile_b → link

Valid topological orders:
  compile_a, compile_b, link, run
  compile_b, compile_a, link, run
```

### DFS-Based Topological Sort

```python
def topo_sort_dfs(graph):
    visited, stack = set(), []

    def dfs(node):
        visited.add(node)
        for nb in graph.get(node, []):
            if nb not in visited:
                dfs(nb)
        stack.append(node)          # append AFTER visiting all descendants

    for node in graph:
        if node not in visited:
            dfs(node)
    return stack[::-1]              # reverse post-order = topological order
```

### Kahn's Algorithm (BFS-based)

Repeatedly removes nodes with **in-degree 0** (no dependencies). Naturally detects cycles — if not all nodes are processed, a cycle exists.

```python
from collections import deque

def kahns(graph, nodes):
    in_degree = {n: 0 for n in nodes}
    for node in nodes:
        for nb in graph.get(node, []):
            in_degree[nb] += 1

    queue = deque([n for n in nodes if in_degree[n] == 0])
    order = []
    while queue:
        node = queue.popleft()
        order.append(node)
        for nb in graph.get(node, []):
            in_degree[nb] -= 1
            if in_degree[nb] == 0:
                queue.append(nb)

    if len(order) != len(nodes):
        raise ValueError("Graph has a cycle")
    return order
```

**Applications:** Build systems (Make, Bazel), package managers (npm, apt), course prerequisite scheduling, spreadsheet cell evaluation, compiler instruction ordering.

---

## Cycle Detection

### Undirected Graph — DFS with Parent Tracking

A back edge (edge to an already-visited non-parent node) indicates a cycle.

```python
def has_cycle_undirected(graph, node, visited, parent):
    visited.add(node)
    for nb in graph[node]:
        if nb not in visited:
            if has_cycle_undirected(graph, nb, visited, node):
                return True
        elif nb != parent:            # back edge found
            return True
    return False
```

### Directed Graph — DFS with Recursion Stack

Track nodes in the current DFS path. A back edge to any node on the current path means a cycle.

```python
def has_cycle_directed(graph, node, visited, rec_stack):
    visited.add(node)
    rec_stack.add(node)
    for nb in graph.get(node, []):
        if nb not in visited:
            if has_cycle_directed(graph, nb, visited, rec_stack):
                return True
        elif nb in rec_stack:         # cycle detected
            return True
    rec_stack.remove(node)
    return False
```

### Union-Find Cycle Detection

Add edges one by one. If both endpoints already share the same component, adding the edge creates a cycle.

```python
def has_cycle_union_find(edges, n):
    parent = list(range(n))

    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]   # path compression
            x = parent[x]
        return x

    for u, v, _ in edges:
        pu, pv = find(u), find(v)
        if pu == pv:
            return True                      # same component → cycle
        parent[pu] = pv
    return False
```

---

## Shortest Path Algorithms

### Dijkstra's Algorithm

Finds the shortest path from a **single source** to all vertices. Works on graphs with **non-negative edge weights**. Uses a min-heap to always expand the cheapest known vertex.

```
Graph:
  A --(4)--> B --(1)--> C
  |                     |
 (2)                   (3)
  |                     |
  D --(5)-----------> E

From A:
  Init:    {A:0, B:∞, C:∞, D:∞, E:∞}
  Pop A:   update B=4, D=2
  Pop D:   update E=7
  Pop B:   update C=5
  Pop C:   update E=8 (no improvement, 7 < 8)
  Pop E:   done

Shortest: A=0, B=4, C=5, D=2, E=7
Path A→E: A→D→E
```

```python
import heapq

def dijkstra(graph, start):
    dist = {node: float('inf') for node in graph}
    prev = {node: None for node in graph}
    dist[start] = 0
    heap = [(0, start)]

    while heap:
        d, u = heapq.heappop(heap)
        if d > dist[u]:
            continue                      # outdated entry
        for v, w in graph[u]:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                prev[v] = u
                heapq.heappush(heap, (dist[v], v))
    return dist, prev

def reconstruct_path(prev, end):
    path = []
    while end is not None:
        path.append(end)
        end = prev[end]
    return path[::-1]
```

**O((V + E) log V)** with a binary heap. Fails on negative edges — use Bellman-Ford.

---

### Bellman-Ford Algorithm

Handles **negative edge weights**. Relaxes every edge V-1 times. A Vth relaxation indicates a **negative cycle**.

```
Relaxation: if dist[u] + w(u,v) < dist[v], update dist[v]
Repeat V-1 times to propagate shortest paths across all vertices.
```

```python
def bellman_ford(vertices, edges, start):
    dist = {v: float('inf') for v in vertices}
    prev = {v: None for v in vertices}
    dist[start] = 0

    for _ in range(len(vertices) - 1):
        updated = False
        for u, v, w in edges:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                prev[v] = u
                updated = True
        if not updated:
            break                         # early exit if converged

    # Detect negative cycles
    for u, v, w in edges:
        if dist[u] + w < dist[v]:
            raise ValueError("Negative cycle detected")

    return dist, prev
```

**O(VE)**. Used in: network routing (RIP protocol), currency arbitrage detection, constraint systems.

---

### SPFA (Shortest Path Faster Algorithm)

A queue-based optimization of Bellman-Ford. Only re-relaxes a vertex's neighbors when that vertex's distance improves. Average case **O(kE)** where k is small in practice, but worst case remains O(VE).

```python
from collections import deque

def spfa(graph, start):
    dist = {v: float('inf') for v in graph}
    dist[start] = 0
    in_queue = {start}
    queue = deque([start])

    while queue:
        u = queue.popleft()
        in_queue.remove(u)
        for v, w in graph[u]:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                if v not in in_queue:
                    queue.append(v)
                    in_queue.add(v)
    return dist
```

---

### Floyd-Warshall Algorithm

Computes **all-pairs shortest paths** in a single O(V³) pass. Works with negative weights (but not negative cycles). The classic dynamic programming approach over intermediate vertices.

```
dist[i][j] = min(dist[i][j],  dist[i][k] + dist[k][j])
              ↑ current best   ↑ route through vertex k
```

```python
def floyd_warshall(n, edges):
    INF = float('inf')
    dist = [[INF] * n for _ in range(n)]
    for i in range(n):
        dist[i][i] = 0
    for u, v, w in edges:
        dist[u][v] = w

    for k in range(n):
        for i in range(n):
            for j in range(n):
                if dist[i][k] + dist[k][j] < dist[i][j]:
                    dist[i][j] = dist[i][k] + dist[k][j]

    # Detect negative cycle: dist[i][i] < 0 for some i
    return dist
```

**Use when:** n ≤ ~500, you need all-pairs distances, or the graph is dense.

---

### Johnson's Algorithm

Computes **all-pairs shortest paths** efficiently for **sparse graphs**, including those with negative weights (but no negative cycles). Reweights edges to eliminate negatives, then runs Dijkstra from every vertex.

```
Steps:
1. Add a new vertex q connected to all other vertices with weight 0
2. Run Bellman-Ford from q → get h[v] for each vertex
3. Reweight edges: w'(u,v) = w(u,v) + h[u] - h[v]   (all non-negative now)
4. Remove q; run Dijkstra from every vertex using w'
5. Adjust results: dist(u,v) = dijkstra_dist(u,v) - h[u] + h[v]
```

**O(V² log V + VE)** — faster than Floyd-Warshall on sparse graphs.

---

### A* (A-Star) Search

A **heuristic-guided** shortest path algorithm. Prioritizes nodes that appear most promising by combining the actual cost so far with an estimated cost to the goal.

```
f(n) = g(n) + h(n)
         ↑       ↑
   cost from   estimated cost
     start      to goal
```

```python
import heapq

def astar(graph, start, goal, heuristic):
    open_set = [(heuristic(start, goal), 0, start)]
    g_score = {start: 0}
    came_from = {}

    while open_set:
        _, g, current = heapq.heappop(open_set)
        if current == goal:
            path = []
            while current in came_from:
                path.append(current)
                current = came_from[current]
            return [current] + path[::-1]
        if g > g_score.get(current, float('inf')):
            continue
        for neighbor, weight in graph[current]:
            new_g = g + weight
            if new_g < g_score.get(neighbor, float('inf')):
                g_score[neighbor] = new_g
                came_from[neighbor] = current
                f = new_g + heuristic(neighbor, goal)
                heapq.heappush(open_set, (f, new_g, neighbor))
    return None
```

**Heuristic must be admissible** (never overestimates) to guarantee optimality.  

| Domain | Heuristic |
|---|---|
| Grid (4-dir) | Manhattan: `|dx| + |dy|` |
| Grid (8-dir) | Chebyshev: `max(|dx|, |dy|)` |
| Any Euclidean | `sqrt(dx² + dy²)` |
| Unweighted | Constant 0 → degenerates to Dijkstra |

**Applications:** Game pathfinding, GPS navigation, robot motion planning, puzzle solving.

---

### Bidirectional Dijkstra

Runs Dijkstra simultaneously **from both source and target**, meeting in the middle. Explores roughly half the nodes of standard Dijkstra.

```
Search space reduction:
  Standard Dijkstra:      O(b^d) nodes
  Bidirectional Dijkstra: O(2 × b^(d/2)) nodes

For b=10, d=6: 1,000,000 vs 2,000 — 500× improvement
```

Used in: Google Maps, Waze, transit routing systems.

---

## Minimum Spanning Tree (MST)

A **spanning tree** is a subgraph that connects all vertices with no cycles. The **MST** is the spanning tree with the minimum total edge weight. Used in network design, clustering, and approximation algorithms.

```
Graph:         MST:
  A--(4)--B      A--(4)--B
  |       |      |
 (2)     (3)    (2)
  |       |      |
  C--(1)--D      C--(1)--D

Total MST weight: 2 + 1 + 3 = 6 (not 2+1+4 or others)
```

### Prim's Algorithm

Grows the MST from a starting vertex by always adding the **cheapest edge** connecting the current tree to a new vertex.

```python
import heapq

def prim(graph, start):
    visited = {start}
    heap = [(w, start, nb) for nb, w in graph[start]]
    heapq.heapify(heap)
    mst_edges = []
    total = 0

    while heap:
        w, u, v = heapq.heappop(heap)
        if v in visited:
            continue
        visited.add(v)
        mst_edges.append((u, v, w))
        total += w
        for nb, weight in graph[v]:
            if nb not in visited:
                heapq.heappush(heap, (weight, v, nb))

    return mst_edges, total
```

**O((V + E) log V)** with binary heap. Grows a single connected component — naturally suited for **dense graphs**.

---

### Kruskal's Algorithm

Sorts all edges by weight and greedily adds each edge that does **not create a cycle** (using Union-Find).

```python
def kruskal(vertices, edges):
    edges.sort(key=lambda e: e[2])    # sort by weight
    parent = {v: v for v in vertices}
    rank = {v: 0 for v in vertices}
    mst = []

    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]
            x = parent[x]
        return x

    def union(x, y):
        px, py = find(x), find(y)
        if px == py: return False
        if rank[px] < rank[py]: px, py = py, px
        parent[py] = px
        if rank[px] == rank[py]: rank[px] += 1
        return True

    for u, v, w in edges:
        if union(u, v):
            mst.append((u, v, w))

    return mst
```

**O(E log E)**. Naturally suited for **sparse graphs**. Produces the MST as a set of edges (not rooted at a particular vertex).

---

### Borůvka's Algorithm

In each phase, every component finds its **cheapest outgoing edge** and merges components along those edges. Completes in O(log V) phases.

```
Phase 1: Each vertex is its own component
         Each finds cheapest edge out → merge
Phase 2: Fewer, larger components
         Repeat until one component remains
```

**O(E log V)**. Naturally parallelizable — each component acts independently per phase. Used in parallel/distributed MST computation.

---

## Strongly Connected Components (SCC)

A **Strongly Connected Component** is a maximal subgraph where every vertex is reachable from every other vertex. Fundamental to understanding the structure of directed graphs.

```
Directed graph:
  A → B → C → A    (SCC 1: {A,B,C})
  C → D → E        (SCCs: {D}, {E})
```

### Kosaraju's Algorithm

Two DFS passes — simple to implement.

```python
def kosaraju(graph, vertices):
    # Pass 1: DFS on original graph, record finish order
    visited, finish_order = set(), []

    def dfs1(v):
        visited.add(v)
        for nb in graph.get(v, []):
            if nb not in visited:
                dfs1(nb)
        finish_order.append(v)

    for v in vertices:
        if v not in visited:
            dfs1(v)

    # Build reversed graph
    rev = {v: [] for v in vertices}
    for v in graph:
        for nb in graph[v]:
            rev[nb].append(v)

    # Pass 2: DFS on reversed graph in reverse finish order
    visited2, sccs = set(), []

    def dfs2(v, comp):
        visited2.add(v)
        comp.append(v)
        for nb in rev.get(v, []):
            if nb not in visited2:
                dfs2(nb, comp)

    for v in reversed(finish_order):
        if v not in visited2:
            comp = []
            dfs2(v, comp)
            sccs.append(comp)

    return sccs
```

**O(V + E)**. Intuition: finishing order of DFS encodes reachability; reversing the graph reveals SCC membership.

---

### Tarjan's Algorithm

Single DFS pass. Maintains a **stack** and tracks discovery time + low-link values to identify SCC roots.

```python
def tarjan(graph, vertices):
    index_counter = [0]
    stack, on_stack = [], set()
    index = {}
    lowlink = {}
    sccs = []

    def strongconnect(v):
        index[v] = lowlink[v] = index_counter[0]
        index_counter[0] += 1
        stack.append(v)
        on_stack.add(v)

        for w in graph.get(v, []):
            if w not in index:
                strongconnect(w)
                lowlink[v] = min(lowlink[v], lowlink[w])
            elif w in on_stack:
                lowlink[v] = min(lowlink[v], index[w])

        if lowlink[v] == index[v]:         # v is root of an SCC
            scc = []
            while True:
                w = stack.pop()
                on_stack.remove(w)
                scc.append(w)
                if w == v: break
            sccs.append(scc)

    for v in vertices:
        if v not in index:
            strongconnect(v)

    return sccs
```

**O(V + E)**. In practice faster than Kosaraju (single pass, better cache behavior). Also used for **2-SAT** problems.

---

## Bridges and Articulation Points

### Bridges
An edge whose removal **disconnects** the graph. Critical for identifying network vulnerabilities.

### Articulation Points (Cut Vertices)
A vertex whose removal **disconnects** the graph.

Both found with a single DFS using **discovery time** and **low values**:

```python
def find_bridges_and_aps(graph, vertices):
    disc = {}
    low = {}
    parent = {}
    timer = [0]
    bridges = []
    aps = set()

    def dfs(u):
        disc[u] = low[u] = timer[0]
        timer[0] += 1
        children = 0

        for v in graph[u]:
            if v not in disc:
                children += 1
                parent[v] = u
                dfs(v)
                low[u] = min(low[u], low[v])

                # Articulation point condition
                if parent.get(u) is None and children > 1:
                    aps.add(u)
                if parent.get(u) is not None and low[v] >= disc[u]:
                    aps.add(u)

                # Bridge condition
                if low[v] > disc[u]:
                    bridges.append((u, v))
            elif v != parent.get(u):
                low[u] = min(low[u], disc[v])

    for v in vertices:
        if v not in disc:
            dfs(v)

    return bridges, aps
```

**O(V + E)**. Used in: network reliability analysis, road traffic planning, circuit design.

---

## Network Flow Algorithms

A **flow network** is a directed weighted graph where edges have **capacities**. Flow can be sent from a **source** s to a **sink** t, respecting capacities.

```
Flow network:
  s --(10)--> A --(10)--> t
  s --(10)--> B --(10)--> t
  A --(5)---> B

Max flow from s to t = 20
(10 through A, 10 through B)
```

**Max-flow Min-cut Theorem:** The maximum amount of flow from s to t equals the minimum capacity of any cut that separates s from t.

### Ford-Fulkerson Method

Repeatedly finds an **augmenting path** from s to t in the residual graph and pushes flow along it.

```python
def ford_fulkerson(graph, source, sink):
    def bfs_path(source, sink, parent):       # find augmenting path
        visited = {source}
        queue = deque([source])
        while queue:
            u = queue.popleft()
            for v in graph[u]:
                if v not in visited and graph[u][v] > 0:
                    visited.add(v)
                    parent[v] = u
                    if v == sink: return True
                    queue.append(v)
        return False

    max_flow = 0
    while True:
        parent = {}
        if not bfs_path(source, sink, parent):
            break
        # Find min capacity along path
        path_flow = float('inf')
        v = sink
        while v != source:
            u = parent[v]
            path_flow = min(path_flow, graph[u][v])
            v = u
        # Update residual capacities
        v = sink
        while v != source:
            u = parent[v]
            graph[u][v] -= path_flow
            graph[v][u] += path_flow
            v = u
        max_flow += path_flow
    return max_flow
```

Using **BFS** to find augmenting paths = **Edmonds-Karp**, which guarantees **O(VE²)**.

---

### Dinic's Algorithm

Builds a **level graph** (BFS layers) then saturates it with **blocking flows** using DFS. Much faster than Edmonds-Karp in practice.

```
Phase 1: BFS → build level graph (assign level = BFS depth to each node)
Phase 2: DFS → find blocking flow (all paths in level graph are saturated)
Repeat until no augmenting path exists.
```

**O(V²E)** general. **O(E√V)** for unit-capacity graphs. **O(E√E)** for unit networks (e.g., bipartite matching). The go-to max-flow algorithm in competitive programming.

---

### Push-Relabel Algorithm

Instead of augmenting paths, **pushes excess flow** from vertices and **relabels** heights. More sophisticated but theoretically faster.

```
height[s] = V  (source elevated)
Push flow downhill (to lower-height neighbors)
When stuck, relabel (increase height of the current vertex)
```

**O(V²E)** or **O(V³)** with FIFO/highest-label variant. Preferred for dense graphs.

---

## Bipartite Graphs & Matching

A graph is **bipartite** if vertices can be split into two sets L and R such that all edges go between L and R (none within a set).

```
L: {A, B, C}   R: {1, 2, 3}
A—1, A—2, B—2, B—3, C—3

Bipartite check: 2-color with BFS/DFS
If any edge connects same-color vertices → not bipartite
```

### Maximum Bipartite Matching (Hopcroft-Karp)

Finds the maximum set of edges with no shared endpoints. Each left vertex matched to at most one right vertex.

```python
from collections import deque

def hopcroft_karp(graph, left, right):
    match_l = {u: None for u in left}
    match_r = {v: None for v in right}

    def bfs():
        dist = {}
        queue = deque()
        for u in left:
            if match_l[u] is None:
                dist[u] = 0
                queue.append(u)
            else:
                dist[u] = float('inf')
        found = False
        while queue:
            u = queue.popleft()
            for v in graph[u]:
                w = match_r[v]
                if w is None:
                    found = True
                elif dist.get(w, float('inf')) == float('inf'):
                    dist[w] = dist[u] + 1
                    queue.append(w)
        return found, dist

    def dfs(u, dist):
        for v in graph[u]:
            w = match_r[v]
            if w is None or (dist.get(w) == dist[u] + 1 and dfs(w, dist)):
                match_l[u] = v
                match_r[v] = u
                return True
        dist[u] = float('inf')
        return False

    matching = 0
    while True:
        found, dist = bfs()
        if not found: break
        for u in left:
            if match_l[u] is None:
                if dfs(u, dist):
                    matching += 1
    return matching, match_l
```

**O(E√V)**. Faster than repeated augmenting paths for bipartite matching.

**Applications:** Job assignment, stable marriage, scheduling, image segmentation (graph cut).

---

### Hungarian Algorithm (Min-Cost Perfect Matching)

Finds the **minimum-cost perfect matching** in a weighted bipartite graph (assigns every left node to exactly one right node at minimum total cost).

**O(V³)**. Used in: assignment problems, optimal task scheduling, transportation problems.

---

## Eulerian and Hamiltonian Paths

### Eulerian Path / Circuit

An **Eulerian path** visits every **edge** exactly once.  
An **Eulerian circuit** starts and ends at the same vertex.

**Existence conditions:**
| Graph Type | Eulerian Circuit | Eulerian Path |
|---|---|---|
| Undirected | All vertices even degree | Exactly 2 odd-degree vertices |
| Directed | in-degree = out-degree ∀v | Exactly one v with out−in=1 (start), one with in−out=1 (end) |

**Hierholzer's Algorithm** — O(V + E):
```python
def hierholzer(graph, start):
    stack, circuit = [start], []
    adj = {v: list(neighbors) for v, neighbors in graph.items()}

    while stack:
        v = stack[-1]
        if adj[v]:
            u = adj[v].pop()
            adj[u].remove(v)              # undirected: remove both directions
            stack.append(u)
        else:
            circuit.append(stack.pop())

    return circuit[::-1]
```

**Applications:** Chinese Postman Problem (mail delivery), DNA fragment assembly, drawing figures without lifting pen.

---

### Hamiltonian Path / Cycle

A **Hamiltonian path** visits every **vertex** exactly once.  
Unlike Eulerian paths, no efficient general algorithm exists — this is **NP-complete**.

**Exact algorithm — Held-Karp DP:**
```
dp[S][v] = minimum cost to visit all vertices in set S, ending at v
dp[{s}][s] = 0
dp[S][v] = min(dp[S \ {v}][u] + cost(u,v))  for u ∈ S, u ≠ v

Time: O(2^V · V²)   Space: O(2^V · V)
```

Practical up to V ≈ 20–25. For larger V, use **approximation algorithms** (Christofides: 1.5× optimal for metric TSP).

**Hamiltonian Cycle = Travelling Salesman Problem (TSP)** when edge weights are included.

---

## Graph Coloring

Assign **colors** to vertices such that no two adjacent vertices share a color.

**Chromatic number χ(G):** minimum colors needed.  
Finding χ(G) is **NP-hard** in general.

### Greedy Coloring
Assign the smallest valid color to each vertex in some order. Not optimal, but fast.

```python
def greedy_coloring(graph, vertices):
    color = {}
    for v in vertices:
        neighbor_colors = {color[nb] for nb in graph[v] if nb in color}
        c = 0
        while c in neighbor_colors:
            c += 1
        color[v] = c
    return color
```

Uses at most Δ+1 colors (Δ = max degree). **Brooks' theorem**: χ(G) ≤ Δ unless G is a complete graph or odd cycle.

**Applications:** Register allocation, scheduling, map coloring, frequency assignment, Sudoku.

---

## Disjoint Set Union (Union-Find)

Efficiently maintains a partition of vertices into **disjoint sets** and answers connectivity queries.

```python
class DSU:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n

    def find(self, x):
        while self.parent[x] != x:
            self.parent[x] = self.parent[self.parent[x]]  # path compression
            x = self.parent[x]
        return x

    def union(self, x, y):
        px, py = self.find(x), self.find(y)
        if px == py: return False
        if self.rank[px] < self.rank[py]: px, py = py, px
        self.parent[py] = px
        if self.rank[px] == self.rank[py]:
            self.rank[px] += 1
        return True

    def connected(self, x, y):
        return self.find(x) == self.find(y)
```

**O(α(n)) ≈ O(1)** per operation with path compression + union by rank. α is the inverse Ackermann function — practically constant for all real inputs.

**Used in:** Kruskal's MST, cycle detection, dynamic connectivity, image segmentation, Kruskal's algorithm.

---

## Directed Acyclic Graphs (DAG) Algorithms

### Longest Path in a DAG

Unlike general graphs (NP-hard), longest path in a **DAG** is solvable in O(V + E) using topological sort + DP.

```python
def dag_longest_path(graph, vertices, source):
    topo = topo_sort_dfs(graph)   # topological order
    dist = {v: float('-inf') for v in vertices}
    dist[source] = 0
    for u in topo:
        for v, w in graph.get(u, []):
            if dist[u] + w > dist[v]:
                dist[v] = dist[u] + w
    return dist
```

### Critical Path Method (CPM)

In project scheduling, the **critical path** is the longest path through the task dependency DAG. Its length = minimum project duration. Tasks on the critical path have zero float (no slack).

### DAG Shortest Path

Relax edges in **topological order** — single pass, **O(V + E)**, works with negative edges.

---

## Advanced Graph Algorithms

### Centroid Decomposition

Decomposes a tree recursively by finding and removing **centroids** (nodes whose removal splits the tree into components of size ≤ n/2). Enables O(n log n) solutions to path queries on trees.

### Heavy-Light Decomposition (HLD)

Decomposes a tree into chains of "heavy" edges. Combined with a segment tree, enables O(log²n) path queries and updates on trees.

### Lowest Common Ancestor (LCA)

Find the deepest node that is an ancestor of both u and v.

```
Methods:
  Naïve:          O(n) per query — walk up from both nodes
  Binary lifting:  O(log n) per query, O(n log n) preprocessing
  Sparse table:    O(1) per query, O(n log n) preprocessing (via Euler tour + RMQ)
  Tarjan offline:  O(α(n)) per query for all queries at once
```

### Euler Tour + Range Minimum Query (RMQ)

Convert LCA to RMQ: do an Euler tour of the tree, recording depths. LCA(u,v) = minimum depth in the Euler tour between first occurrences of u and v. Sparse table gives O(1) RMQ.

### 2-SAT (2-Satisfiability)

Given a Boolean formula in 2-CNF (each clause has at most 2 literals), determine if it's satisfiable in **O(V + E)** using SCC (Tarjan or Kosaraju).

```
(A ∨ B) ∧ (¬A ∨ C) ∧ (¬B ∨ ¬C)
→ implication graph: ¬A→B, ¬B→A, A→C, ¬C→¬A, B→¬C, C→¬B
→ find SCCs → check if any variable and its negation are in the same SCC
```

### Steiner Tree

Minimum-weight tree connecting a specific subset of vertices (not necessarily a spanning tree). NP-hard in general; solvable in **O(3^k · V + 2^k · V²)** via DP where k = number of required vertices.

---

## How to Choose a Graph Algorithm

```
Traversal / Connectivity?
  └── BFS or DFS — O(V+E)

Shortest path (unweighted)?
  └── BFS — O(V+E)

Shortest path (non-negative weights, single source)?
  └── Dijkstra — O((V+E) log V)

Shortest path (negative weights, single source)?
  └── Bellman-Ford — O(VE)

Shortest path (all pairs, dense)?
  └── Floyd-Warshall — O(V³)

Shortest path (all pairs, sparse, negative weights)?
  └── Johnson's Algorithm — O(V² log V + VE)

Best-first / heuristic shortest path?
  └── A* — O(E log V)

Minimum Spanning Tree?
  ├── Dense graph → Prim — O((V+E) log V)
  └── Sparse graph → Kruskal — O(E log E)

Dependency ordering / scheduling?
  └── Topological Sort (Kahn's or DFS) — O(V+E)

Cycle detection?
  ├── Undirected → DFS with parent tracking
  ├── Directed   → DFS with recursion stack
  └── Edge-by-edge → Union-Find

Strongly connected components?
  └── Tarjan (single pass) or Kosaraju (two passes) — O(V+E)

Network vulnerabilities (bridges / cut vertices)?
  └── Tarjan bridge/AP algorithm — O(V+E)

Maximum flow?
  ├── General → Dinic's — O(V²E)
  └── Unit capacity → Dinic's — O(E√V)

Bipartite matching (unweighted)?
  └── Hopcroft-Karp — O(E√V)

Bipartite matching (min-cost)?
  └── Hungarian Algorithm — O(V³)

Eulerian circuit / path?
  └── Hierholzer — O(V+E)

Hamiltonian path / TSP?
  └── Held-Karp DP — O(2^V · V²)  (exact, small V)
  └── Christofides — 1.5× approx for metric TSP

Dynamic connectivity / grouping?
  └── Union-Find — O(α(n)) per operation

Path queries on a tree?
  └── Heavy-Light Decomposition + Segment Tree — O(log²n)

Lowest Common Ancestor?
  └── Binary lifting — O(log n) query, O(n log n) preprocess
```

---

## Quick Reference Card

```
TRAVERSAL                   SHORTEST PATH
  BFS       O(V+E)  queue     Dijkstra    O((V+E)logV)  non-neg weights
  DFS       O(V+E)  stack     Bellman-F   O(VE)         neg weights
  Topo Sort O(V+E)  DAG only  SPFA        O(kE) avg     neg weights
                              Floyd-W     O(V³)         all-pairs
SPANNING TREE               Johnson's   O(V²logV+VE)  all-pairs sparse
  Prim      O((V+E)logV)      A*          O(E logV)     heuristic
  Kruskal   O(E log E)
  Borůvka   O(E log V)      FLOW / MATCHING
                              Ford-Fulk   O(E·maxflow)
SCC / STRUCTURE             Edmonds-K   O(VE²)
  Kosaraju  O(V+E)  2 DFS    Dinic's     O(V²E)        fast in practice
  Tarjan    O(V+E)  1 DFS    Push-Relabel O(V²E)
  Bridges   O(V+E)           Hopcroft-K  O(E√V)        bipartite
  Art.Pts   O(V+E)           Hungarian   O(V³)         min-cost match

CONNECTIVITY                EULERIAN / HAMILTONIAN
  Union-Find O(α(n))          Hierholzer  O(V+E)        Euler circuit
  Bipartite  O(V+E)           Held-Karp   O(2^V · V²)   Hamiltonian / TSP

TREE ALGORITHMS
  LCA (binary lift)  O(log n) query
  HLD + Seg Tree     O(log²n) path query
  Centroid Decomp    O(n log n) path problems
  2-SAT              O(V+E) via SCC
```
