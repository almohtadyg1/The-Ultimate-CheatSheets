# Graph Algorithms: A Complete Progressive Tutorial

---

## 1. What & Why

A graph G = (V, E) is a collection of vertices (nodes) connected by edges. It is the most general and expressive data structure in computer science — because real-world relationships are graphs. Road networks, social connections, dependency chains, circuit layouts, the internet, molecular structures, and scheduling problems are all graphs.

The algorithms that operate on graphs solve some of the most practically valuable problems in computing: find the shortest route between two cities (Dijkstra), determine if two users are connected through mutual friends (BFS), detect circular dependencies in a build system (DFS cycle detection), schedule tasks that depend on each other (topological sort), and distribute load across a network (max-flow).

You will encounter graphs constantly in software engineering. Understanding the core algorithms — their correctness, complexity, and the conditions under which each applies — is a fundamental competency.

---

## 2. Mental Model

Think of a graph as a map with cities and roads:

```
                    [Cairo]
                   /       \
               225km        134km
              /               \
       [Alexandria]          [Suez]
             |                  |
          290km              195km
             |                  |
     [Marsa Matruh]       [Hurghada]
```

Vertices = cities. Edges = roads. Weights = distances. Directed edges = one-way roads.

Different algorithms answer different questions about this map:
- **BFS**: What is the minimum number of road segments to get from Cairo to Hurghada?
- **Dijkstra**: What is the shortest total distance from Cairo to any city?
- **DFS/Topological Sort**: If I need to visit certain cities in dependency order, what is a valid sequence?
- **MST (Prim/Kruskal)**: What is the minimum total road length needed to connect all cities?
- **Union-Find**: Are two cities in the same connected region at all?

The representation you choose — adjacency list, adjacency matrix, or edge list — determines which operations are cheap. Most real-world graphs are sparse (few edges relative to vertices squared), so adjacency lists are the default.

---

## 3. Progressive Examples

### Level 1: Graph Representation and Basic Traversal

```python
from collections import defaultdict, deque

class Graph:
    """
    Adjacency list graph — O(V + E) space.
    Default for most problems: efficient for sparse graphs, supports both
    directed and undirected, weighted and unweighted.
    """

    def __init__(self, directed=False):
        self.adj = defaultdict(list)
        self.directed = directed

    def add_edge(self, u, v, weight=1):
        self.adj[u].append((v, weight))
        if not self.directed:
            self.adj[v].append((u, weight))

    def vertices(self):
        return set(self.adj.keys())

    def neighbors(self, u):
        return self.adj[u]

# Build the city network
g = Graph(directed=False)
g.add_edge("Cairo", "Alexandria", 225)
g.add_edge("Cairo", "Suez", 134)
g.add_edge("Alexandria", "Marsa Matruh", 290)
g.add_edge("Suez", "Hurghada", 195)

# Adjacency matrix representation — O(V²) space
# Better when: dense graph, need O(1) edge existence check, graph is small
def build_adjacency_matrix(vertices, edges):
    """edges: list of (u, v, weight) tuples"""
    v_to_i = {v: i for i, v in enumerate(vertices)}
    n = len(vertices)
    matrix = [[0] * n for _ in range(n)]
    for u, v, w in edges:
        matrix[v_to_i[u]][v_to_i[v]] = w
        matrix[v_to_i[v]][v_to_i[u]] = w  # undirected
    return matrix, v_to_i
```

### Level 2: BFS and DFS — The Foundation of Everything

```python
def bfs(graph, start):
    """
    Breadth-First Search: visit all nodes level by level.
    Explores all nodes at distance 1 before distance 2, etc.

    Guarantees SHORTEST PATH in terms of edge count (unweighted graphs).
    Time: O(V + E), Space: O(V)
    """
    visited = {start}
    queue = deque([(start, 0)])   # (node, distance from start)
    distances = {start: 0}
    order = []

    while queue:
        node, dist = queue.popleft()
        order.append(node)

        for neighbor, _ in graph.adj[node]:
            if neighbor not in visited:
                visited.add(neighbor)              # mark when ENQUEUING
                distances[neighbor] = dist + 1
                queue.append((neighbor, dist + 1))

    return order, distances

def bfs_path(graph, start, goal):
    """BFS that reconstructs the actual path."""
    if start == goal:
        return [start]

    visited = {start}
    queue = deque([(start, [start])])

    while queue:
        node, path = queue.popleft()
        for neighbor, _ in graph.adj[node]:
            if neighbor == goal:
                return path + [neighbor]
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, path + [neighbor]))
    return []   # no path

order, distances = bfs(g, "Cairo")
print("BFS order:", order)
print("Distances:", distances)
path = bfs_path(g, "Cairo", "Hurghada")
print("Path:", " → ".join(path))

def dfs(graph, start):
    """
    Depth-First Search: dive deep before backtracking.
    Uses recursion (or an explicit stack for large graphs).

    Used for: cycle detection, topological sort, connected components,
              strongly connected components, maze solving.
    Time: O(V + E), Space: O(V) recursion depth
    """
    visited = set()
    order = []

    def _dfs(node):
        visited.add(node)
        order.append(node)
        for neighbor, _ in graph.adj[node]:
            if neighbor not in visited:
                _dfs(neighbor)

    _dfs(start)
    return order

def has_cycle_undirected(graph, vertices):
    """
    Detect a cycle in an undirected graph using DFS.
    If we reach an already-visited node that is NOT our parent → cycle.
    """
    visited = set()

    def _dfs(node, parent):
        visited.add(node)
        for neighbor, _ in graph.adj[node]:
            if neighbor not in visited:
                if _dfs(neighbor, node):
                    return True
            elif neighbor != parent:    # visited AND not parent → back edge → cycle
                return True
        return False

    return any(_dfs(v, None) for v in vertices if v not in visited)
```

### Level 3: Shortest Path Algorithms

```python
import heapq

def dijkstra(graph, start):
    """
    Dijkstra's algorithm: shortest path in a weighted graph with non-negative weights.

    The greedy insight: always extend the currently closest unfinalized vertex.
    Once a vertex is popped from the heap, its distance is final — no shorter path
    can reach it (because all weights are non-negative).

    Time: O((V + E) log V) with binary heap
    Space: O(V)
    """
    dist = defaultdict(lambda: float('inf'))
    dist[start] = 0
    heap = [(0, start)]   # (distance, vertex)
    prev = {start: None}

    while heap:
        d, u = heapq.heappop(heap)

        if d > dist[u]:
            continue    # stale entry — we already found a shorter path

        for v, weight in graph.adj[u]:
            new_dist = d + weight
            if new_dist < dist[v]:
                dist[v] = new_dist
                prev[v] = u
                heapq.heappush(heap, (new_dist, v))

    return dict(dist), prev

def reconstruct_path(prev, start, goal):
    path = []
    node = goal
    while node is not None:
        path.append(node)
        node = prev.get(node)
    path.reverse()
    return path if path[0] == start else []

distances, prev = dijkstra(g, "Cairo")
print("Shortest distances from Cairo:", distances)
path = reconstruct_path(prev, "Cairo", "Hurghada")
print("Shortest path:", " → ".join(path), f"({distances['Hurghada']} km)")

def bellman_ford(vertices, edges, start):
    """
    Bellman-Ford: shortest path with negative weight edges.
    Also detects negative weight cycles (which make shortest paths undefined).

    Time: O(VE) — much slower than Dijkstra.
    Use only when negative weights are possible.
    """
    dist = {v: float('inf') for v in vertices}
    dist[start] = 0

    # Relax all edges V-1 times
    # A shortest path with no cycles has at most V-1 edges
    for _ in range(len(vertices) - 1):
        for u, v, w in edges:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w

    # Check for negative weight cycles
    # If we can still relax an edge, there's a negative cycle
    for u, v, w in edges:
        if dist[u] + w < dist[v]:
            raise ValueError("Graph contains a negative weight cycle")

    return dist

def floyd_warshall(n, adj_matrix):
    """
    All-pairs shortest path: find shortest paths between ALL pairs of vertices.
    Time: O(V³). Only practical for small graphs (V ≤ ~500).

    Uses dynamic programming: dist[i][j] through intermediate vertex k.
    """
    dist = [row[:] for row in adj_matrix]  # copy
    INF = float('inf')

    for k in range(n):          # try each vertex as intermediate
        for i in range(n):
            for j in range(n):
                if dist[i][k] + dist[k][j] < dist[i][j]:
                    dist[i][j] = dist[i][k] + dist[k][j]

    return dist
```

### Level 4: Minimum Spanning Tree

```python
def kruskal(vertices, edges):
    """
    Kruskal's MST: greedily add the cheapest edge that doesn't create a cycle.
    Uses Union-Find to detect cycles in O(α(n)) ≈ O(1).

    Time: O(E log E) — dominated by sorting edges
    Best when: edges are already sorted, or you want simplicity
    """
    # Union-Find data structure
    parent = {v: v for v in vertices}
    rank = {v: 0 for v in vertices}

    def find(x):
        if parent[x] != x:
            parent[x] = find(parent[x])   # path compression
        return parent[x]

    def union(x, y):
        rx, ry = find(x), find(y)
        if rx == ry:
            return False    # already in same component — would create cycle
        if rank[rx] < rank[ry]:
            rx, ry = ry, rx
        parent[ry] = rx
        if rank[rx] == rank[ry]:
            rank[rx] += 1
        return True

    mst_edges = []
    total_weight = 0

    for weight, u, v in sorted(edges):   # sort by weight ascending
        if union(u, v):
            mst_edges.append((u, v, weight))
            total_weight += weight
            if len(mst_edges) == len(vertices) - 1:
                break   # MST complete — V-1 edges

    return mst_edges, total_weight

def prim(graph, start):
    """
    Prim's MST: grow the tree one vertex at a time, always adding the cheapest
    edge that connects a new vertex to the existing tree.

    Time: O((V + E) log V) with binary heap
    Best when: graph is dense (many edges), or vertices are the primary structure
    """
    visited = {start}
    mst_edges = []
    total_weight = 0

    # Min-heap: (weight, from_vertex, to_vertex)
    heap = [(w, start, v) for v, w in graph.adj[start]]
    heapq.heapify(heap)

    while heap and len(visited) < len(graph.vertices()):
        weight, u, v = heapq.heappop(heap)

        if v in visited:
            continue    # vertex already in MST

        visited.add(v)
        mst_edges.append((u, v, weight))
        total_weight += weight

        for neighbor, w in graph.adj[v]:
            if neighbor not in visited:
                heapq.heappush(heap, (w, v, neighbor))

    return mst_edges, total_weight
```

### Level 5: Topological Sort and Strongly Connected Components

```python
def topological_sort_dfs(vertices, adj):
    """
    Topological sort of a DAG using DFS.
    A topological order is a linear ordering where for every directed edge
    u→v, u appears before v in the ordering.

    Applications: task scheduling, build systems (make, bazel), course prerequisites.
    Time: O(V + E)
    """
    visited = set()
    stack = []   # nodes are added to stack AFTER all their dependencies are visited

    def dfs(node):
        visited.add(node)
        for neighbor in adj.get(node, []):
            if neighbor not in visited:
                dfs(neighbor)
        stack.append(node)   # add to stack only after visiting all successors

    for v in vertices:
        if v not in visited:
            dfs(v)

    return stack[::-1]   # reverse of DFS finish order = topological order

def kahn_topological_sort(vertices, edges):
    """
    Kahn's algorithm: BFS-based topological sort.
    Also detects cycles: if not all vertices are processed, graph has a cycle.

    Used in build systems, dependency resolvers, scheduling.
    Time: O(V + E)
    """
    in_degree = {v: 0 for v in vertices}
    adj = defaultdict(list)

    for u, v in edges:
        adj[u].append(v)
        in_degree[v] += 1

    # Start with vertices that have no dependencies
    queue = deque([v for v in vertices if in_degree[v] == 0])
    result = []

    while queue:
        node = queue.popleft()
        result.append(node)

        for neighbor in adj[node]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)

    if len(result) != len(vertices):
        raise ValueError("Graph has a cycle — topological sort impossible")

    return result

# Example: build system task ordering
tasks = ["compile", "test", "lint", "package", "deploy", "install_deps"]
task_edges = [
    ("install_deps", "compile"),
    ("compile", "test"),
    ("compile", "lint"),
    ("test", "package"),
    ("lint", "package"),
    ("package", "deploy"),
]
order = kahn_topological_sort(tasks, task_edges)
print("Build order:", " → ".join(order))

def kosaraju_scc(vertices, adj):
    """
    Kosaraju's algorithm: find all Strongly Connected Components (SCCs).
    An SCC is a maximal subset of vertices where every vertex is reachable
    from every other (in a directed graph).

    Applications: find circular dependencies, analyze program control flow,
                  identify tight communities in social networks.
    Time: O(V + E) — two DFS passes
    """
    # Pass 1: DFS on original graph, collect finish-order stack
    visited = set()
    finish_stack = []

    def dfs1(node):
        visited.add(node)
        for neighbor in adj.get(node, []):
            if neighbor not in visited:
                dfs1(neighbor)
        finish_stack.append(node)

    for v in vertices:
        if v not in visited:
            dfs1(v)

    # Build reversed graph
    rev_adj = defaultdict(list)
    for u in adj:
        for v in adj[u]:
            rev_adj[v].append(u)

    # Pass 2: DFS on reversed graph in reverse finish order
    visited.clear()
    sccs = []

    def dfs2(node, component):
        visited.add(node)
        component.append(node)
        for neighbor in rev_adj.get(node, []):
            if neighbor not in visited:
                dfs2(neighbor, component)

    while finish_stack:
        node = finish_stack.pop()
        if node not in visited:
            component = []
            dfs2(node, component)
            sccs.append(component)

    return sccs
```

### Level 6: A* Search and Union-Find

```python
import math

def a_star(graph, start, goal, heuristic):
    """
    A* search: Dijkstra's algorithm guided by a heuristic function.
    The heuristic h(v) estimates the distance from v to goal.

    Guarantees optimal path when h is admissible (never overestimates).
    Much faster than Dijkstra when a good heuristic is available.

    Time: O(E log V) in practice with a good heuristic
    Applications: game pathfinding, GPS routing, robotics
    """
    g_score = defaultdict(lambda: float('inf'))
    g_score[start] = 0

    f_score = {start: heuristic(start, goal)}   # f = g + h
    heap = [(f_score[start], start)]
    came_from = {}

    while heap:
        f, current = heapq.heappop(heap)

        if current == goal:
            # Reconstruct path
            path = []
            while current in came_from:
                path.append(current)
                current = came_from[current]
            path.append(start)
            return path[::-1], g_score[goal]

        for neighbor, weight in graph.adj[current]:
            tentative_g = g_score[current] + weight
            if tentative_g < g_score[neighbor]:
                came_from[neighbor] = current
                g_score[neighbor] = tentative_g
                f_val = tentative_g + heuristic(neighbor, goal)
                heapq.heappush(heap, (f_val, neighbor))

    return [], float('inf')   # no path found

# Union-Find / Disjoint Set Union — for dynamic connectivity
class UnionFind:
    """
    Tracks which vertices are connected.
    Operations are effectively O(1) with path compression + union by rank.

    Applications: Kruskal's MST, detecting cycles, network connectivity,
                  image segmentation, Kruskal's, clustering.
    """

    def __init__(self, vertices):
        self.parent = {v: v for v in vertices}
        self.rank = {v: 0 for v in vertices}
        self.components = len(vertices)

    def find(self, x):
        """Find root with path compression — flattens the tree."""
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])   # path compression
        return self.parent[x]

    def union(self, x, y):
        """Merge components. Returns False if already connected."""
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return False   # already in same component

        # Union by rank: attach smaller tree under larger
        if self.rank[rx] < self.rank[ry]:
            rx, ry = ry, rx
        self.parent[ry] = rx
        if self.rank[rx] == self.rank[ry]:
            self.rank[rx] += 1
        self.components -= 1
        return True

    def connected(self, x, y):
        return self.find(x) == self.find(y)

# Example: social network connectivity
people = ["Alice", "Bob", "Carol", "Dave", "Eve"]
uf = UnionFind(people)

uf.union("Alice", "Bob")    # Alice and Bob are friends
uf.union("Carol", "Dave")   # Carol and Dave are friends
uf.union("Bob", "Carol")    # Bob and Carol are friends → merges groups

print(uf.connected("Alice", "Dave"))   # True — they're in the same component
print(uf.connected("Alice", "Eve"))    # False — Eve is isolated
print(f"Connected components: {uf.components}")   # 2 (Alice-Bob-Carol-Dave, Eve)
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Marking nodes visited when dequeuing, not enqueuing, in BFS**

```python
# WRONG: same node can be added to queue multiple times
queue = deque([start])
while queue:
    node = queue.popleft()
    if node in visited:     # check happens too late
        continue
    visited.add(node)
    for neighbor in graph[node]:
        queue.append(neighbor)   # may add already-queued nodes

# CORRECT: mark visited when adding to queue
visited = {start}
queue = deque([start])
while queue:
    node = queue.popleft()
    for neighbor in graph[node]:
        if neighbor not in visited:
            visited.add(neighbor)    # mark immediately
            queue.append(neighbor)
```

**Mistake 2: Using Dijkstra with negative edge weights**

```python
# WRONG: Dijkstra's greedy property fails with negative weights
# A shorter path via a negative edge may exist after we've "finalized" a vertex

# Example: A→B (weight 5), A→C (weight 2), C→B (weight -4)
# Dijkstra finalizes B with distance 5.
# But the path A→C→B has distance 2-4 = -2, which is shorter!

# CORRECT: use Bellman-Ford for graphs with negative weights
# Only use Dijkstra when all weights are strictly non-negative.
```

**Mistake 3: Forgetting to handle disconnected graphs in DFS**

```python
# WRONG: only starts from one vertex — misses other components
def dfs_wrong(graph, start):
    visited = set()
    stack = [start]
    while stack:
        node = stack.pop()
        if node not in visited:
            visited.add(node)
            for neighbor in graph[node]:
                stack.append(neighbor)
    return visited  # misses nodes not reachable from 'start'

# CORRECT: iterate over ALL vertices to handle disconnected graphs
def dfs_all(graph, vertices):
    visited = set()
    components = []
    for v in vertices:
        if v not in visited:
            component = []
            def dfs(node):
                visited.add(node)
                component.append(node)
                for neighbor in graph.get(node, []):
                    if neighbor not in visited:
                        dfs(neighbor)
            dfs(v)
            components.append(component)
    return components
```

**Mistake 4: Topological sort on graphs with cycles**

```python
# Topological sort is only defined for DAGs (Directed Acyclic Graphs).
# If the graph has a cycle, no topological ordering exists.

# Kahn's algorithm catches this:
# If the result doesn't include all vertices, the remaining vertices form a cycle.

result = kahn_topological_sort(vertices, edges)
if len(result) != len(vertices):
    print("Cycle detected — cannot topologically sort")
```

**Mistake 5: Choosing adjacency matrix for sparse graphs**

```python
# Adjacency matrix uses O(V²) space regardless of edge count.
# For a social network with 1 million users, each with average 100 friends:
# Adjacency matrix: 10^12 entries = ~1 TB of memory
# Adjacency list:   100 million entries = ~800 MB

# Rule: use adjacency list unless:
# - Graph is dense (E ≈ V²)
# - You need O(1) edge existence queries
# - V is small (V < ~1000)
```

---

## 5. The "Why Does This Work" Layer

### Why Dijkstra's Greedy Choice Is Correct

The key insight: if all edge weights are non-negative, once we've found the shortest path to a vertex u (popped from the heap), no shorter path can ever be found later. Any alternative path would have to go through some unfinalized vertex v, and since dist[v] ≥ dist[u] (heap property), and all edge weights ≥ 0, the path through v can only be at least as long.

This greedy property breaks with negative weights: a path through a later-discovered vertex with a negative edge could make the total distance shorter than what we computed. Hence Bellman-Ford, which relaxes all edges V-1 times to handle this.

### Why Kruskal's Always Produces a Valid MST

Kruskal's adds the cheapest edge that connects two previously disconnected components. The correctness proof uses the **cut property**: for any cut of the graph (partition of vertices into two sets), the minimum-weight edge crossing the cut is in SOME minimum spanning tree.

When Kruskal's considers edge (u, v), it has already added all cheaper edges. The vertices in u's component form one side of a cut, and v's component forms another. The cheapest edge crossing this cut is (u, v) — so it must be in the MST. Union-Find efficiently tracks components to detect when an edge would create a cycle (both endpoints in the same component).

### How Path Compression Makes Union-Find Nearly O(1)

Without optimization, Union-Find's `find` traverses up the tree to the root: O(tree height). With union by rank, the height is O(log n). With path compression, after each `find`, we make every node on the path point directly to the root. Future finds on those nodes take O(1).

The combined effect of union by rank + path compression gives amortized O(α(n)) per operation, where α(n) is the inverse Ackermann function — it grows so slowly that for any practical input size (less than 10^80000 elements), α(n) ≤ 4. It is effectively constant.

---

## 6. Quick Reference

### Algorithm Selection

| Problem | Algorithm | Complexity |
|---------|-----------|------------|
| Shortest path (unweighted) | BFS | O(V + E) |
| Shortest path (weighted, ≥0) | Dijkstra | O((V+E) log V) |
| Shortest path (negative weights) | Bellman-Ford | O(VE) |
| All-pairs shortest path | Floyd-Warshall | O(V³) |
| Minimum spanning tree | Kruskal / Prim | O(E log E) / O((V+E) log V) |
| Topological sort | DFS / Kahn's | O(V + E) |
| Cycle detection (undirected) | DFS | O(V + E) |
| Cycle detection (directed) | DFS (three-color) | O(V + E) |
| Strongly connected components | Tarjan / Kosaraju | O(V + E) |
| Connectivity queries | Union-Find | O(α(n)) |
| Pathfinding with heuristic | A* | O(E log V) |

### Representation Trade-offs

| | Adjacency List | Adjacency Matrix | Edge List |
|--|---------------|-----------------|-----------|
| Space | O(V + E) | O(V²) | O(E) |
| Edge check | O(degree) | O(1) | O(E) |
| Neighbor iter | O(degree) | O(V) | O(E) |
| Best for | Sparse graphs | Dense / small | Kruskal, Bellman-Ford |

### Graph Vocabulary

| Term | Definition |
|------|-----------|
| Vertex / Node | Fundamental unit |
| Edge / Arc | Connection between two vertices |
| DAG | Directed Acyclic Graph |
| SCC | Strongly Connected Component |
| MST | Minimum Spanning Tree |
| In-degree / Out-degree | Directed: edges in / out |
| Connected | All vertices mutually reachable (undirected) |
| Bipartite | Two-colorable (no odd cycles) |
