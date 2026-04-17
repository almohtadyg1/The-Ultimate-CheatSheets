# Searching Algorithms: A Complete Progressive Tutorial

---

## 1. What & Why

Searching is the process of locating a target value — or confirming it is absent — within a collection of data. It is the most frequently executed operation in computing. Every database query, file lookup, URL routing decision, and autocomplete suggestion is a search problem.

Most developers never implement searching algorithms from scratch — their language's standard library handles it. But you need to understand the trade-offs because the right choice depends on three factors that vary enormously across use cases: whether your data is sorted, what data structure it lives in, and how frequently you search versus update.

A naive linear scan is fine for a list of 20 items. It is catastrophic for a database with 20 million rows. Understanding when to use binary search, hash lookup, trie traversal, or graph traversal is the difference between code that scales and code that doesn't.

---

## 2. Mental Model

Think of searching as navigating an information space. Each algorithm makes different assumptions about the shape of that space:

```
Unsorted array — you're searching a random pile of papers.
Linear search: flip each paper one by one. O(n).

Sorted array — you're in a dictionary.
Binary search: open the middle, eliminate half, repeat. O(log n).

Hash table — you have an index card catalog.
Hash lookup: compute the card number directly. O(1).

Graph — you're navigating a road network.
BFS: explore all roads at distance 1, then 2, then 3.
DFS: go as far as possible down one road before backtracking.

String — you're looking for a phrase in a novel.
KMP/Boyer-Moore: use the pattern itself to skip ahead. O(n+m).
```

The algorithm you choose reflects your assumptions about the data's structure. Wrong assumption → wrong algorithm → poor performance.

---

## 3. Progressive Examples

### Level 1: Linear Search — The Baseline

```python
def linear_search(arr, target):
    """
    Scan every element from left to right.
    No assumptions about data — works on any collection.
    O(n) time, O(1) space.
    """
    for i, val in enumerate(arr):
        if val == target:
            return i      # found at index i
    return -1             # not found

data = [64, 34, 25, 12, 22, 11, 90]
print(linear_search(data, 22))   # 4
print(linear_search(data, 99))   # -1

# When linear search is the right answer:
# - Small collections (n < 100): overhead of sorting isn't worth it
# - Unsorted data that you only search once (sorting would be O(n log n))
# - Searching linked lists (no random access for binary search)
# - Searching for all occurrences, not just the first

def find_all(arr, target):
    """Find ALL indices where target appears."""
    return [i for i, val in enumerate(arr) if val == target]

# Self-organizing search: move frequently accessed items to the front
def linear_search_move_to_front(arr, target):
    """
    After a hit, swap the found element to position 0.
    Over repeated searches, frequent items migrate to the front.
    Reduces average search time for skewed access patterns.
    """
    for i in range(len(arr)):
        if arr[i] == target:
            arr[0], arr[i] = arr[i], arr[0]   # move to front
            return 0
    return -1
```

### Level 2: Binary Search — The Sorted Array Power Tool

```python
def binary_search(arr, target):
    """
    Halve the search space at each step by comparing target to the middle element.
    Requires a sorted array.
    O(log n) time, O(1) space (iterative).
    """
    lo, hi = 0, len(arr) - 1

    while lo <= hi:
        mid = lo + (hi - lo) // 2   # avoids integer overflow vs (lo + hi) // 2

        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            lo = mid + 1    # target is in the right half
        else:
            hi = mid - 1    # target is in the left half

    return -1

sorted_data = [3, 7, 9, 10, 27, 38, 43, 82]
print(binary_search(sorted_data, 27))   # 4
print(binary_search(sorted_data, 5))    # -1

# Trace for target=27 in [3, 7, 9, 10, 27, 38, 43, 82]:
# lo=0, hi=7, mid=3, arr[3]=10 < 27  → lo=4
# lo=4, hi=7, mid=5, arr[5]=38 > 27  → hi=4
# lo=4, hi=4, mid=4, arr[4]=27 == 27 → return 4

# Binary search VARIANTS — these come up constantly in interviews and production

def lower_bound(arr, target):
    """
    Find the LEFTMOST position where target could be inserted to keep arr sorted.
    If target exists: returns its first occurrence index.
    If not exists: returns the index of the first element greater than target.
    """
    lo, hi = 0, len(arr)
    while lo < hi:
        mid = (lo + hi) // 2
        if arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid        # don't exclude mid — it might BE the answer
    return lo               # lo == hi at termination

def upper_bound(arr, target):
    """
    Find the RIGHTMOST position where target could be inserted.
    Returns index of the first element STRICTLY GREATER THAN target.
    """
    lo, hi = 0, len(arr)
    while lo < hi:
        mid = (lo + hi) // 2
        if arr[mid] <= target:   # <= instead of <
            lo = mid + 1
        else:
            hi = mid
    return lo

arr = [1, 3, 3, 3, 5, 7]
print(lower_bound(arr, 3))   # 1 — first index where 3 appears
print(upper_bound(arr, 3))   # 4 — index after last 3
# Count occurrences of 3:
print(upper_bound(arr, 3) - lower_bound(arr, 3))   # 3

# Binary search on the answer (the "search space is a range" pattern)
def find_sqrt_floor(n):
    """Find floor(sqrt(n)) using binary search on the answer."""
    if n < 2:
        return n
    lo, hi = 1, n // 2
    while lo <= hi:
        mid = (lo + hi) // 2
        if mid * mid == n:
            return mid
        elif mid * mid < n:
            lo = mid + 1
        else:
            hi = mid - 1
    return hi   # hi is the floor

print(find_sqrt_floor(25))   # 5
print(find_sqrt_floor(26))   # 5
print(find_sqrt_floor(27))   # 5
```

### Level 3: Graph Search — BFS and DFS

```python
from collections import deque

# BFS: Breadth-First Search
# Explores all nodes at distance 1 from start, then distance 2, etc.
# Guarantees shortest path (minimum edges) in unweighted graphs.
def bfs(graph, start, goal):
    """
    Find shortest path from start to goal in an unweighted graph.
    Returns the path as a list, or None if no path exists.
    """
    if start == goal:
        return [start]

    visited = {start}
    queue = deque([(start, [start])])   # (current node, path to here)

    while queue:
        node, path = queue.popleft()

        for neighbor in graph.get(node, []):
            if neighbor == goal:
                return path + [neighbor]

            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, path + [neighbor]))

    return None   # no path found

# DFS: Depth-First Search
# Explores as far as possible along each branch before backtracking.
# Used for: topological sort, cycle detection, connected components, maze solving.
def dfs(graph, start, visited=None):
    """
    Visit all nodes reachable from start using depth-first traversal.
    Returns list of visited nodes in DFS order.
    """
    if visited is None:
        visited = set()

    visited.add(start)
    result = [start]

    for neighbor in graph.get(start, []):
        if neighbor not in visited:
            result.extend(dfs(graph, neighbor, visited))

    return result

# Example: social network — find path between two users
social_graph = {
    "Alice": ["Bob", "Carol"],
    "Bob": ["Alice", "Dave", "Eve"],
    "Carol": ["Alice", "Frank"],
    "Dave": ["Bob"],
    "Eve": ["Bob", "Frank"],
    "Frank": ["Carol", "Eve"],
}

path = bfs(social_graph, "Alice", "Frank")
print("Shortest path:", " → ".join(path))
# Alice → Carol → Frank

all_reachable = dfs(social_graph, "Alice")
print("All reachable from Alice:", all_reachable)

# DFS application: detect cycle in directed graph
def has_cycle(graph, nodes):
    """Detect cycle using DFS with three-color marking."""
    WHITE, GRAY, BLACK = 0, 1, 2   # unvisited, in-progress, complete
    color = {node: WHITE for node in nodes}

    def dfs_cycle(node):
        color[node] = GRAY         # mark as in-progress
        for neighbor in graph.get(node, []):
            if color[neighbor] == GRAY:
                return True        # back edge — cycle!
            if color[neighbor] == WHITE and dfs_cycle(neighbor):
                return True
        color[node] = BLACK        # mark complete
        return False

    return any(dfs_cycle(n) for n in nodes if color[n] == WHITE)
```

### Level 4: Hash-Based Search and Binary Search Trees

```python
# Hash table search: O(1) average
# The fastest search for exact key lookup — but requires knowing the exact key.

class HashSearch:
    """Demonstrate hash-based search patterns."""

    def __init__(self):
        self.table = {}

    def insert(self, key, value):
        self.table[key] = value    # O(1) amortized

    def search(self, key):
        return self.table.get(key)  # O(1) average, O(n) worst case (hash collisions)

    def search_by_value(self, value):
        """Search by value, not key — O(n), must scan everything."""
        return [k for k, v in self.table.items() if v == value]

# BST search: O(log n) average, O(n) worst case (degenerate tree)
class BSTNode:
    def __init__(self, key, value):
        self.key = key
        self.value = value
        self.left = self.right = None

class BSTSearch:
    def __init__(self):
        self.root = None

    def insert(self, key, value):
        self.root = self._insert(self.root, key, value)

    def _insert(self, node, key, value):
        if node is None:
            return BSTNode(key, value)
        if key < node.key:
            node.left = self._insert(node.left, key, value)
        elif key > node.key:
            node.right = self._insert(node.right, key, value)
        else:
            node.value = value   # update existing key
        return node

    def search(self, key):
        """O(log n) for balanced tree, O(n) for degenerate."""
        node = self.root
        while node:
            if key == node.key:
                return node.value
            elif key < node.key:
                node = node.left
            else:
                node = node.right
        return None

    def range_search(self, low, high):
        """Find all entries with keys in [low, high]. O(log n + k) where k = results."""
        result = []
        self._range(self.root, low, high, result)
        return result

    def _range(self, node, low, high, result):
        if not node:
            return
        if low < node.key:
            self._range(node.left, low, high, result)
        if low <= node.key <= high:
            result.append((node.key, node.value))
        if node.key < high:
            self._range(node.right, low, high, result)

# When to use BST vs hash table:
# Hash table: fastest for exact key lookup, no ordering
# BST: slightly slower, but supports range queries, sorted iteration, predecessor/successor
```

### Level 5: String Pattern Searching — KMP Algorithm

```python
def build_failure_function(pattern):
    """
    Build the 'failure function' (also called the prefix function) for KMP.
    failure[i] = length of longest proper prefix of pattern[:i+1] that is also a suffix.

    This encodes the information we can use to skip comparisons.
    """
    m = len(pattern)
    failure = [0] * m
    j = 0   # length of previous longest prefix-suffix

    for i in range(1, m):
        while j > 0 and pattern[i] != pattern[j]:
            j = failure[j - 1]    # fall back using failure function

        if pattern[i] == pattern[j]:
            j += 1
        failure[i] = j

    return failure

def kmp_search(text, pattern):
    """
    Knuth-Morris-Pratt string matching.
    Finds all occurrences of pattern in text.
    O(n + m) time — never re-examines a character.

    The key insight: when a mismatch occurs, the failure function tells us
    how far back we can jump in the pattern without re-examining text.
    """
    n, m = len(text), len(pattern)
    if m == 0:
        return [0]

    failure = build_failure_function(pattern)
    matches = []
    j = 0   # current position in pattern

    for i in range(n):      # i advances through text, never goes back
        while j > 0 and text[i] != pattern[j]:
            j = failure[j - 1]   # partial match — skip using failure function

        if text[i] == pattern[j]:
            j += 1

        if j == m:           # found a complete match
            matches.append(i - m + 1)   # start index of match
            j = failure[j - 1]          # prepare for next search

    return matches

text = "ABABDABACDABABCABAB"
pattern = "ABABCABAB"
positions = kmp_search(text, pattern)
print(f"Pattern found at positions: {positions}")   # [10]

# Compare: naive string search is O(nm) — much worse for long patterns
def naive_search(text, pattern):
    """O(nm) — shift pattern by 1 each mismatch, re-examine all chars."""
    n, m = len(text), len(pattern)
    for i in range(n - m + 1):
        if text[i:i+m] == pattern:   # each comparison is O(m)
            yield i

# For n=1M, m=100: KMP does ~1M operations. Naive does ~100M.
```

### Level 6: Dijkstra's Algorithm — Weighted Graph Search

```python
import heapq
from typing import Dict, List, Tuple

def dijkstra(graph: Dict[str, List[Tuple[str, int]]], start: str) -> Dict[str, int]:
    """
    Find shortest path distances from start to all reachable vertices.
    Works on weighted graphs with non-negative edge weights.

    graph: adjacency list {node: [(neighbor, weight), ...]}
    Returns: {node: shortest_distance_from_start}

    Time: O((V + E) log V) with binary heap
    Space: O(V)
    """
    distances = {node: float('inf') for node in graph}
    distances[start] = 0

    # Priority queue: (distance, node)
    # Always processes the closest unvisited node first
    heap = [(0, start)]

    while heap:
        dist, node = heapq.heappop(heap)

        # Skip stale entries — we may have found a shorter path since this was added
        if dist > distances[node]:
            continue

        for neighbor, weight in graph.get(node, []):
            new_dist = dist + weight
            if new_dist < distances[neighbor]:
                distances[neighbor] = new_dist
                heapq.heappush(heap, (new_dist, neighbor))

    return distances

def dijkstra_with_path(graph, start, goal):
    """Dijkstra that also reconstructs the actual path."""
    distances = {node: float('inf') for node in graph}
    distances[start] = 0
    previous = {node: None for node in graph}
    heap = [(0, start)]

    while heap:
        dist, node = heapq.heappop(heap)
        if dist > distances[node]:
            continue
        if node == goal:
            break   # early exit — we found the shortest path to goal

        for neighbor, weight in graph.get(node, []):
            new_dist = dist + weight
            if new_dist < distances[neighbor]:
                distances[neighbor] = new_dist
                previous[neighbor] = node
                heapq.heappush(heap, (new_dist, neighbor))

    # Reconstruct path
    path = []
    node = goal
    while node is not None:
        path.append(node)
        node = previous[node]
    path.reverse()

    return distances[goal], path if path[0] == start else []

# Example: city routing
city_graph = {
    "Cairo":     [("Alexandria", 225), ("Suez", 134)],
    "Alexandria":[("Cairo", 225), ("Marsa Matruh", 290)],
    "Suez":      [("Cairo", 134), ("Hurghada", 195)],
    "Marsa Matruh": [("Alexandria", 290)],
    "Hurghada":  [("Suez", 195)],
}

distances = dijkstra(city_graph, "Cairo")
print("Distances from Cairo:")
for city, dist in sorted(distances.items(), key=lambda x: x[1]):
    print(f"  {city}: {dist} km")

cost, path = dijkstra_with_path(city_graph, "Cairo", "Hurghada")
print(f"\nShortest route to Hurghada: {' → '.join(path)} ({cost} km)")
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Using binary search on unsorted data**

```python
# WRONG: binary search requires sorted data — results are undefined otherwise
data = [64, 34, 25, 12, 22, 11, 90]
print(binary_search(data, 22))    # may return wrong answer or -1

# CORRECT: sort first, then binary search (only worth it if you search multiple times)
data.sort()
print(binary_search(data, 22))    # correct

# If you're only searching once, sort + binary search is O(n log n) total —
# worse than linear search's O(n). Binary search pays off only when you search
# the same sorted structure multiple times.
```

**Mistake 2: Integer overflow in the mid-point calculation**

```python
# WRONG in languages with fixed-width integers (C, Java):
# mid = (lo + hi) // 2    -- lo + hi can overflow if both are large

# CORRECT: equivalent but overflow-safe
mid = lo + (hi - lo) // 2

# In Python, integers are arbitrary precision so this doesn't matter.
# But write it correctly to build the habit — it matters in C++, Java, Go.
```

**Mistake 3: Off-by-one errors in binary search boundaries**

```python
# The condition and boundary updates must be consistent:

# Pattern A: lo <= hi, update to mid±1
lo, hi = 0, len(arr) - 1
while lo <= hi:            # = because lo==hi is a valid single-element range
    mid = lo + (hi - lo) // 2
    if arr[mid] < target:
        lo = mid + 1       # +1: we've already checked mid
    else:
        hi = mid - 1       # -1: we've already checked mid

# Pattern B: lo < hi, update hi to mid (not mid-1)
# Used for lower_bound / upper_bound — subtly different
lo, hi = 0, len(arr)       # hi = len(arr), NOT len(arr)-1
while lo < hi:             # < because lo==hi means empty range
    mid = (lo + hi) // 2
    if arr[mid] < target:
        lo = mid + 1
    else:
        hi = mid           # don't subtract 1 — mid might be the answer

# Mixing these patterns causes infinite loops or missed elements.
```

**Mistake 4: BFS with a visited set in the wrong place**

```python
# WRONG: marking visited when DEQUEUING instead of ENQUEUING
# Can cause the same node to be added to the queue multiple times
queue = deque([start])
while queue:
    node = queue.popleft()
    if node in visited:     # too late — already enqueued multiple times
        continue
    visited.add(node)
    for neighbor in graph[node]:
        queue.append(neighbor)

# CORRECT: mark visited when ENQUEUING
visited = {start}
queue = deque([start])
while queue:
    node = queue.popleft()
    for neighbor in graph[node]:
        if neighbor not in visited:
            visited.add(neighbor)   # mark before adding to queue
            queue.append(neighbor)
# This prevents duplicate entries in the queue.
```

**Mistake 5: Using DFS when you need shortest path**

```python
# DFS does NOT find shortest paths in unweighted graphs.
# It finds A path, not necessarily the SHORTEST path.

graph = {
    "A": ["B", "C"],
    "B": ["D"],
    "C": ["D"],
    "D": []
}

# DFS from A to D might return A → B → D (2 hops)
# or A → C → D (also 2 hops — same length here)
# But in a more complex graph, DFS can find longer paths than necessary.

# BFS guarantees shortest path (fewest edges) in unweighted graphs.
# Dijkstra guarantees shortest path in weighted graphs (non-negative weights).
# Bellman-Ford handles negative weights.
```

---

## 5. The "Why Does This Work" Layer

### Why Binary Search Is O(log n)

Binary search halves the search space at every step. Starting with n elements:

```
After 1 comparison: n/2 elements remain
After 2 comparisons: n/4 elements remain
After 3 comparisons: n/8 elements remain
After k comparisons: n/2^k elements remain
```

The search ends when `n/2^k = 1`, meaning `k = log₂(n)` comparisons. For n = 1,000,000: at most 20 comparisons. For n = 1,000,000,000: at most 30 comparisons. This is why O(log n) algorithms are nearly indifferent to input size beyond a certain point.

### Why KMP Never Re-examines Characters

The naive string search slides the pattern one position and retries from the beginning — potentially re-examining the same text characters many times. KMP eliminates this by precomputing the "failure function" for the pattern.

The failure function encodes: if we matched k characters and then fail, how many of those matched characters can we "reuse" without re-examining the text? The answer depends on whether the matched portion has a prefix that is also a suffix.

```
Pattern: ABABCABAB
Failure: 0 0 1 2 0 1 2 3 4

If we matched 6 characters (ABABCA) and fail at position 6,
failure[5] = 1, meaning we can skip ahead to pattern position 1
without re-examining any text. The text cursor never moves backward.
```

This is why KMP is O(n + m): the text cursor moves forward n times total, and pattern position adjustments using the failure function never exceed m operations total.

### Why Dijkstra Fails with Negative Weights

Dijkstra's correctness relies on a greedy property: once a node is "finalized" (popped from the priority queue), its shortest distance is correct and won't improve.

This holds because: if all weights are non-negative, any path through an unfinalized node must be at least as long as the direct path (since we're adding non-negative weights). A negative edge breaks this: a path through a later-discovered node could be shorter by taking a negative-weight edge.

For graphs with negative weights, use Bellman-Ford (O(VE)) — it relaxes all edges V-1 times, guaranteeing that any shortest path of at most V-1 edges is found correctly.

---

## 6. Quick Reference

### Algorithm Selection Guide

| Situation | Algorithm | Complexity |
|-----------|-----------|------------|
| Unsorted, any data type | Linear search | O(n) |
| Sorted array, exact match | Binary search | O(log n) |
| Sorted array, range query | Binary search (lower/upper bound) | O(log n) |
| Hash table, exact key | Hash lookup | O(1) avg |
| Unweighted graph, shortest path | BFS | O(V + E) |
| Graph, any traversal / cycle detection | DFS | O(V + E) |
| Weighted graph, shortest path (≥0) | Dijkstra | O((V+E) log V) |
| Weighted graph, negative edges | Bellman-Ford | O(VE) |
| String pattern matching | KMP or Boyer-Moore | O(n + m) |
| String prefix search | Trie | O(m) |
| Approximate membership | Bloom filter | O(k) |

### Binary Search Templates

```python
# Exact match
lo, hi = 0, len(arr) - 1
while lo <= hi:
    mid = lo + (hi - lo) // 2
    if arr[mid] == target: return mid
    elif arr[mid] < target: lo = mid + 1
    else: hi = mid - 1
return -1

# Lower bound (first position where arr[i] >= target)
lo, hi = 0, len(arr)
while lo < hi:
    mid = (lo + hi) // 2
    if arr[mid] < target: lo = mid + 1
    else: hi = mid
return lo

# Upper bound (first position where arr[i] > target)
lo, hi = 0, len(arr)
while lo < hi:
    mid = (lo + hi) // 2
    if arr[mid] <= target: lo = mid + 1
    else: hi = mid
return lo
```

### Python Built-ins for Searching

```python
import bisect

# Binary search (requires sorted list)
bisect.bisect_left(arr, target)   # lower bound
bisect.bisect_right(arr, target)  # upper bound
bisect.insort(arr, value)         # insert in sorted order

# Hash-based search (O(1))
value in my_dict                  # key existence
value in my_set                   # membership

# Standard library BFS
from collections import deque
# Use deque as shown in examples above

# For graphs, consider networkx:
import networkx as nx
G = nx.Graph()
nx.shortest_path(G, source, target)        # BFS-based
nx.shortest_path_length(G, source, target) # with/without weights
```
