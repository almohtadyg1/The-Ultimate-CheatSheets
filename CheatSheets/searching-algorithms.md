# Searching Algorithms Cheat Sheet
### From Zero to Expert

---

## What is Searching?

**Searching** is the process of locating a target element — or determining it doesn't exist — within a data collection. It is the most frequently executed operation in computing: every database query, file lookup, autocomplete suggestion, and routing decision is a search.

```
Target: 42
Array:  [3, 9, 10, 27, 38, 42, 82]
                         ↑
                      Found at index 5
```

The right algorithm depends on three things: **whether the data is sorted**, **the structure it lives in**, and **how often you search vs. update**.

---

## Complexity Master Table

| Algorithm | Best | Average | Worst | Space | Data Requirement |
|---|---|---|---|---|---|
| Linear Search | O(1) | O(n) | O(n) | O(1) | None |
| Binary Search | O(1) | O(log n) | O(log n) | O(1) iter / O(log n) rec | Sorted array |
| Ternary Search | O(1) | O(log₃ n) | O(log₃ n) | O(1) | Sorted / unimodal |
| Jump Search | O(1) | O(√n) | O(√n) | O(1) | Sorted array |
| Interpolation Search | O(1) | O(log log n) | O(n) | O(1) | Sorted, uniform dist. |
| Exponential Search | O(1) | O(log n) | O(log n) | O(1) | Sorted array |
| Fibonacci Search | O(1) | O(log n) | O(log n) | O(1) | Sorted array |
| BFS | O(1) | O(V+E) | O(V+E) | O(V) | Unweighted graph/tree |
| DFS | O(1) | O(V+E) | O(V+E) | O(V) | Graph/tree |
| Dijkstra | O(1) | O((V+E) log V) | O((V+E) log V) | O(V) | Weighted graph, no neg |
| A* | O(1) | O(E) | O(b^d) | O(b^d) | Weighted graph + heuristic |
| Bellman-Ford | O(E) | O(VE) | O(VE) | O(V) | Weighted, negative edges |
| Hash Search | O(1) | O(1) | O(n) | O(n) | Hash table |
| BST Search | O(1) | O(log n) | O(n) | O(n) | Binary search tree |
| Trie Search | O(m) | O(m) | O(m) | O(n·m) | Trie |
| KMP | O(n) | O(n+m) | O(n+m) | O(m) | String pattern match |
| Rabin-Karp | O(n) | O(n+m) | O(nm) | O(1) | String pattern match |
| Boyer-Moore | O(n/m) | O(n+m) | O(nm) | O(m+Σ) | String pattern match |
| Z Algorithm | O(n+m) | O(n+m) | O(n+m) | O(n+m) | String pattern match |
| Aho-Corasick | O(n+m+z) | O(n+m+z) | O(n+m+z) | O(m·Σ) | Multi-pattern string |
| Suffix Array | O(m log n) | O(m log n) | O(m log n) | O(n) | Text preprocessing |
| Bloom Filter | O(k) | O(k) | O(k) | O(m) | Probabilistic membership |
| Ternary Search Tree | O(log n) | O(log n) | O(n) | O(n) | String keys |

*V = vertices, E = edges, m = pattern length, n = text/array length, k = hash functions, b = branching factor, d = depth*

---

## Linear Search

The most fundamental search. Scan every element from left to right until the target is found or the array is exhausted. Makes **no assumptions** about the data.

```
Target: 27
Array: [38, 7, 43, 27, 9, 82, 10]
        ↑   ↑   ↑   ↑
       ✗   ✗   ✗   ✓  Found at index 3
```

```python
def linear_search(arr, target):
    for i, val in enumerate(arr):
        if val == target:
            return i
    return -1
```

### Optimizations

**Sentinel Search** — place the target at the end of the array. Eliminates the bounds check `i < n` inside the loop, reducing branch overhead.

```python
def sentinel_search(arr, target):
    n = len(arr)
    arr.append(target)          # sentinel — guarantees loop terminates
    i = 0
    while arr[i] != target:
        i += 1
    arr.pop()                   # restore array
    return i if i < n else -1
```

**Self-Organizing Search** — after a successful hit, move the found element toward the front. Over repeated queries, frequently accessed elements migrate to the head, reducing average search time.

- *Move-to-front*: swap found element to index 0
- *Transpose*: swap found element with its predecessor

**Use when:** Data is unsorted, small, or accessed rarely. O(n) is acceptable for n ≤ ~1,000 in most contexts.

---

## Binary Search

The canonical efficient search. Repeatedly halves the search space by comparing the target to the **middle element** of a sorted array.

```
Target: 27
Sorted: [3, 7, 9, 10, 27, 38, 43, 82]
         lo               hi
              mid=10  → 27 > 10, search right half
                        [27, 38, 43, 82]
                         lo      hi
                            mid=43 → 27 < 43, search left half
                        [27, 38]
                         lo hi
                            mid=27 → Found ✓
```

```python
def binary_search(arr, target):           # iterative — O(1) space
    lo, hi = 0, len(arr) - 1
    while lo <= hi:
        mid = lo + (hi - lo) // 2        # avoids integer overflow
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1
```

> **Critical detail:** always compute `mid = lo + (hi - lo) // 2`, not `(lo + hi) // 2`. The latter overflows in languages with fixed-width integers when lo + hi > MAX_INT.

### Binary Search Variants

**Find first occurrence** of target (leftmost):
```python
def lower_bound(arr, target):
    lo, hi, result = 0, len(arr) - 1, -1
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        if arr[mid] == target:
            result = mid; hi = mid - 1  # keep searching left
        elif arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return result
```

**Find last occurrence** (rightmost): same pattern but `lo = mid + 1` on match.

**Find first element ≥ target** (lower_bound in C++):
```python
def lower_bound_geq(arr, target):
    lo, hi = 0, len(arr)
    while lo < hi:
        mid = (lo + hi) // 2
        if arr[mid] < target: lo = mid + 1
        else: hi = mid
    return lo
```

**Binary search on answer** — when the answer is a number and you can verify a candidate in O(f(n)), binary search the answer space directly. A powerful technique for optimization problems.

```
"Find minimum speed such that task completes in T hours"
  → binary search speed in [1..MAX], check feasibility at each midpoint
```

**Use when:** Array is sorted and static (infrequent updates). The go-to for O(log n) search.

---

## Ternary Search

Divides the search space into **three parts** instead of two. Primarily used for finding the **maximum or minimum of a unimodal function** (one that strictly increases then strictly decreases, or vice versa).

```
Unimodal function f(x):

         peak
        /    \
       /      \
──────/        \──────
   lo  m1  m2  hi

If f(m1) < f(m2): peak is in [m1, hi] → lo = m1
If f(m1) > f(m2): peak is in [lo, m2] → hi = m2
```

```python
def ternary_search(f, lo, hi, eps=1e-9):
    while hi - lo > eps:
        m1 = lo + (hi - lo) / 3
        m2 = hi - (hi - lo) / 3
        if f(m1) < f(m2):
            lo = m1
        else:
            hi = m2
    return (lo + hi) / 2
```

On a sorted array, ternary search has a *worse* constant than binary search (uses ~2 comparisons per iteration vs. 1). **Only prefer ternary search for unimodal optimization problems.**

---

## Jump Search

Jumps ahead by fixed blocks of size **√n**, then does a linear scan within the identified block. A middle ground between linear and binary search — useful when backward traversal is expensive (e.g., magnetic tape, one-directional streams).

```
Target: 55
Array: [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89]
Block size: √12 ≈ 3

Jump: index 0  → arr[0]=0  < 55, jump
Jump: index 3  → arr[3]=2  < 55, jump
Jump: index 6  → arr[6]=8  < 55, jump
Jump: index 9  → arr[9]=34 < 55, jump
Jump: index 12 → out of bounds, stop
Linear scan backward from index 9:
  arr[9]=34, arr[10]=55 ✓
```

```python
import math

def jump_search(arr, target):
    n = len(arr)
    step = int(math.sqrt(n))
    prev = 0
    while arr[min(step, n) - 1] < target:
        prev = step
        step += int(math.sqrt(n))
        if prev >= n:
            return -1
    for i in range(prev, min(step, n)):
        if arr[i] == target:
            return i
    return -1
```

**Optimal block size is √n**, balancing the number of jumps (√n) against the linear scan length (√n).

---

## Interpolation Search

An improvement on binary search for **uniformly distributed sorted data**. Instead of always probing the middle, it estimates the likely position of the target using linear interpolation — like how a human would search in a phone book.

```
pos = lo + ((target - arr[lo]) * (hi - lo)) / (arr[hi] - arr[lo])
```

```python
def interpolation_search(arr, target):
    lo, hi = 0, len(arr) - 1
    while lo <= hi and arr[lo] <= target <= arr[hi]:
        if lo == hi:
            return lo if arr[lo] == target else -1
        pos = lo + ((target - arr[lo]) * (hi - lo)) // (arr[hi] - arr[lo])
        if arr[pos] == target:
            return pos
        elif arr[pos] < target:
            lo = pos + 1
        else:
            hi = pos - 1
    return -1
```

**O(log log n)** for uniformly distributed data — significantly faster than binary search.  
**O(n) worst case** for heavily skewed distributions (e.g., exponentially spaced values).

---

## Exponential Search

Finds a suitable range for binary search on **unbounded or infinite arrays**. First finds the range `[2^k, 2^(k+1)]` that contains the target, then applies binary search within it.

```
Target: 10
Array: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, ...]

Check index 1:  arr[1]=2  < 10
Check index 2:  arr[2]=3  < 10
Check index 4:  arr[4]=5  < 10
Check index 8:  arr[8]=9  < 10
Check index 16: arr[16]=? ≥ 10 (or out of bounds)
Binary search in [8, 16] → finds 10 at index 9
```

```python
def exponential_search(arr, target):
    if arr[0] == target:
        return 0
    i = 1
    while i < len(arr) and arr[i] <= target:
        i *= 2
    return binary_search(arr[i // 2 : min(i, len(arr))], target)
```

**O(log n)** — identical to binary search asymptotically, but ideal when the array size is unknown or the element is near the beginning.

---

## Fibonacci Search

Uses **Fibonacci numbers** to divide the array into unequal parts. Avoids division (uses only addition and subtraction), making it useful on hardware where division is expensive.

```
Fibonacci numbers: 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, ...

For array of size n, find smallest Fibonacci number ≥ n.
Use two consecutive Fibonacci numbers as offsets to probe.
Each step eliminates either the larger (2/3) or smaller (1/3) portion.
```

Like binary search, runs in O(log n). The divide ratio is the **golden ratio φ ≈ 1.618** instead of 2. Historically useful on systems without fast division hardware.

---

## Breadth-First Search (BFS)

Explores a graph or tree **level by level**, using a queue. Guarantees finding the **shortest path** (fewest edges) in an unweighted graph.

```
Graph:                BFS from A:
  A — B — E          Level 0: [A]
  |   |              Level 1: [B, C]
  C — D              Level 2: [E, D]
                     Level 3: (none)

Queue trace: [A] → [B,C] → [C,E] → [E,D] → [D] → []
```

```python
from collections import deque

def bfs(graph, start, target):
    visited = set([start])
    queue = deque([(start, [start])])     # (node, path)
    while queue:
        node, path = queue.popleft()
        if node == target:
            return path
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, path + [neighbor]))
    return None
```

**Use when:** Finding shortest path in unweighted graphs, level-order tree traversal, connected components, social network distance, web crawlers.

---

## Depth-First Search (DFS)

Explores a graph by going **as deep as possible** before backtracking, using a stack (or recursion).

```
Graph:                DFS from A (recursive):
  A — B — E          Visit A → B → E (backtrack) → (backtrack)
  |   |                → D → C (backtrack to A) done
  C — D

DFS order: A, B, E, D, C
```

```python
def dfs(graph, node, target, visited=None):
    if visited is None:
        visited = set()
    visited.add(node)
    if node == target:
        return True
    for neighbor in graph[node]:
        if neighbor not in visited:
            if dfs(graph, neighbor, target, visited):
                return True
    return False
```

**Iterative DFS** (explicit stack — avoids recursion limit):
```python
def dfs_iterative(graph, start, target):
    stack, visited = [start], set()
    while stack:
        node = stack.pop()
        if node in visited: continue
        visited.add(node)
        if node == target: return True
        stack.extend(graph[node])
    return False
```

**Use when:** Cycle detection, topological sorting, maze solving, connected components, strongly connected components, backtracking problems.

### DFS Applications
| Application | How DFS helps |
|---|---|
| Topological Sort | Post-order DFS on a DAG |
| Cycle Detection | Back edge found during DFS |
| SCC (Kosaraju/Tarjan) | Two DFS passes |
| Articulation Points | Track discovery time + low values |
| Bipartite Check | 2-color nodes during DFS |

---

## Dijkstra's Algorithm

Finds the **shortest path from a source to all vertices** in a weighted graph with **non-negative edge weights**. Uses a min-heap priority queue to always expand the lowest-cost known vertex.

```
Graph (weighted):
  A —(4)— B —(1)— C
  |               |
 (2)             (3)
  |               |
  D —(5)——————— E

From A:
  dist = {A:0, B:∞, C:∞, D:∞, E:∞}
  Process A: update B=4, D=2
  Process D: update E=7
  Process B: update C=5
  Process C: update E=8 (no improvement)
  Process E: done
  Result: A→D→E = 7  (not A→B→C→E = 8)
```

```python
import heapq

def dijkstra(graph, start):
    dist = {node: float('inf') for node in graph}
    dist[start] = 0
    heap = [(0, start)]                   # (cost, node)

    while heap:
        cost, u = heapq.heappop(heap)
        if cost > dist[u]:
            continue                      # stale entry
        for v, weight in graph[u]:
            if dist[u] + weight < dist[v]:
                dist[v] = dist[u] + weight
                heapq.heappush(heap, (dist[v], v))
    return dist
```

**O((V + E) log V)** with a binary heap. **O(V + E log V)** with Fibonacci heap (theoretical).  
**Fails with negative edge weights** — use Bellman-Ford instead.

---

## A* Search

An **informed** best-first search that combines Dijkstra's cost-so-far with a **heuristic** estimate of cost-to-goal. Finds the optimal path while exploring far fewer nodes than Dijkstra's.

```
f(n) = g(n) + h(n)
         ↑       ↑
   actual cost  estimated cost
   from start   to goal (heuristic)
```

```python
import heapq

def astar(graph, start, goal, h):
    open_set = [(h(start), 0, start, [start])]  # (f, g, node, path)
    visited = {}

    while open_set:
        f, g, node, path = heapq.heappop(open_set)
        if node == goal:
            return path
        if node in visited and visited[node] <= g:
            continue
        visited[node] = g
        for neighbor, weight in graph[node]:
            new_g = g + weight
            heapq.heappush(open_set,
                (new_g + h(neighbor), new_g, neighbor, path + [neighbor]))
    return None
```

### Heuristic Requirements
A heuristic h(n) must be **admissible** (never overestimates) to guarantee optimality.

| Domain | Heuristic |
|---|---|
| Grid (4-directional) | Manhattan distance: `|dx| + |dy|` |
| Grid (8-directional) | Chebyshev distance: `max(|dx|, |dy|)` |
| Any metric space | Euclidean distance |
| Word ladder | Hamming distance (mismatched chars) |

A heuristic is **consistent** (monotone) if `h(n) ≤ cost(n, n') + h(n')`. Consistent → admissible. Consistent heuristics prevent re-expansion of nodes.

**Use when:** Pathfinding in games, maps (Google Maps uses A* variants), robotics navigation, puzzle solving (15-puzzle, Rubik's cube).

---

## Bellman-Ford Algorithm

Finds shortest paths from a source, handling **negative edge weights**. Also detects **negative cycles** (a cycle whose total weight is negative — makes shortest path undefined).

```python
def bellman_ford(vertices, edges, start):
    dist = {v: float('inf') for v in vertices}
    dist[start] = 0

    for _ in range(len(vertices) - 1):    # relax all edges V-1 times
        for u, v, w in edges:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w

    # Detect negative cycles: if any edge still relaxes, cycle exists
    for u, v, w in edges:
        if dist[u] + w < dist[v]:
            raise ValueError("Negative cycle detected")

    return dist
```

**O(VE)** — slower than Dijkstra but handles negative weights. Used in: network routing protocols (RIP), currency arbitrage detection, constraint solving.

---

## Floyd-Warshall Algorithm

Finds **all-pairs shortest paths** — shortest path between every pair of vertices — in a single pass.

```python
def floyd_warshall(graph, n):
    dist = [[float('inf')] * n for _ in range(n)]
    for i in range(n):
        dist[i][i] = 0
    for u, v, w in graph:
        dist[u][v] = w

    for k in range(n):                  # intermediate vertex
        for i in range(n):
            for j in range(n):
                dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])
    return dist
```

**O(V³)** time, O(V²) space. Best for **dense graphs** where you need all pairwise distances. Used in: network analysis, transitive closure, detecting negative cycles.

---

## Hash-Based Search

Insert elements into a **hash table**; lookup by key in O(1) average time. The fastest practical search for exact-match queries.

```
Insert: hash("alice") % 8 = 3  →  table[3] = ("alice", 99)
Lookup: hash("alice") % 8 = 3  →  table[3].key == "alice" ✓  → return 99
```

**O(1) average, O(n) worst case** (all keys collide). Worst case avoided with good hash functions and load factor management (resize when α > 0.7).

**Use when:** Exact-key lookup, deduplication, frequency counting. The most common search structure in production systems.

---

## Binary Search Tree (BST) Search

Traverse the BST by comparing the target to each node: go left if smaller, go right if larger.

```
Search 7 in BST:
        [8]
       /   \
     [3]   [10]
    /   \
  [1]   [6]
           \
           [7]  ← found

Path: 8 → 3 → 6 → 7
```

```python
def bst_search(node, target):
    if node is None or node.val == target:
        return node
    if target < node.val:
        return bst_search(node.left, target)
    return bst_search(node.right, target)
```

O(log n) average on a balanced BST; O(n) worst case on a degenerate (linear) tree. Use **AVL or Red-Black trees** for guaranteed O(log n).

---

## Trie Search

A Trie stores strings as paths through a tree, one character per node. Search runs in **O(m)** time where m is the length of the search key — independent of how many strings are stored.

```
Trie storing: "car", "card", "care", "cat"

        root
         |
        [c]
         |
        [a]
       /   \
     [r]   [t*]      * = end of word
    /   \
  [d*] [e*]
```

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

def search(root, word):
    node = root
    for ch in word:
        if ch not in node.children:
            return False
        node = node.children[ch]
    return node.is_end
```

**Use when:** Prefix-based search, autocomplete, spell checkers, IP routing (CIDR lookup), dictionary word validation.

---

## String Searching Algorithms

Searching for a **pattern** (length m) inside a **text** (length n).

---

### Naïve String Search

Try every possible starting position. O(nm) — the baseline to beat.

```
Text:    AABAACAADAABAABA
Pattern: AABA

Try pos 0: AABA == AABA ✓
Try pos 1: ABAA ≠ AABA
...
```

---

### Knuth-Morris-Pratt (KMP)

Preprocesses the pattern into a **failure function** (partial match table) that tells you how much to slide the pattern forward without re-examining characters you've already matched.

```
Pattern: ABABCABAB
Failure: [0,0,1,2,0,1,2,3,4]

Text:    ABABDABABCABABCABAB
         ABABCABAB
             ↑ mismatch at D
             failure[3]=2 → slide pattern by 3-2=1, resume from pos 2
```

```python
def kmp_search(text, pattern):
    n, m = len(text), len(pattern)
    fail = compute_failure(pattern)
    j = 0
    results = []
    for i in range(n):
        while j > 0 and text[i] != pattern[j]:
            j = fail[j - 1]
        if text[i] == pattern[j]:
            j += 1
        if j == m:
            results.append(i - m + 1)
            j = fail[j - 1]
    return results

def compute_failure(pattern):
    m = len(pattern)
    fail = [0] * m
    j = 0
    for i in range(1, m):
        while j > 0 and pattern[i] != pattern[j]:
            j = fail[j - 1]
        if pattern[i] == pattern[j]:
            j += 1
        fail[i] = j
    return fail
```

**O(n + m)** — linear. Never re-examines a character in the text.

---

### Boyer-Moore

Searches **right to left** within the pattern window. Two heuristics skip large portions of the text:

**Bad Character Rule** — when a mismatch occurs at text[i], shift the pattern so the rightmost occurrence of text[i] in the pattern aligns with i. If text[i] doesn't appear in the pattern, shift past it entirely.

**Good Suffix Rule** — if a suffix of the pattern matched before a mismatch, shift the pattern to align the next occurrence of that suffix.

```
Text:    AABCBABCDABCDABDE
Pattern: ABCD

          ABCD          ← mismatch: text has 'B', pattern has 'C'
          Bad char 'B' is at index 1 in pattern → shift right by 2
              ABCD      ← try new position
```

**O(n/m) best case** — can skip most of the text. In practice, the fastest algorithm for large alphabets and long patterns. Used in: `grep`, text editors.

---

### Rabin-Karp

Uses **rolling hashes** to find pattern matches. Computes a hash of the pattern and slides a hash window over the text — only do a full string comparison when hashes match.

```
Pattern hash: hash("ABCD") = 42
Rolling window over text:
  hash("AABC") = 17  ≠ 42, skip
  hash("ABCB") = 31  ≠ 42, skip
  hash("BCBA") = 55  ≠ 42, skip
  hash("ABCD") = 42  → hashes match → verify character by character ✓
```

```python
def rabin_karp(text, pattern, base=256, mod=101):
    n, m = len(text), len(pattern)
    ph = th = 0
    h = pow(base, m - 1, mod)          # base^(m-1) mod p
    results = []

    for i in range(m):
        ph = (base * ph + ord(pattern[i])) % mod
        th = (base * th + ord(text[i])) % mod

    for i in range(n - m + 1):
        if ph == th:
            if text[i:i+m] == pattern:  # verify to rule out hash collision
                results.append(i)
        if i < n - m:
            th = (base * (th - ord(text[i]) * h) + ord(text[i + m])) % mod
            th = (th + mod) % mod
    return results
```

**O(n + m) average, O(nm) worst** (all hash collisions). Excellent for **multiple pattern matching** — hash all patterns, single text pass.

---

### Z Algorithm

Builds a **Z-array** where `Z[i]` = length of the longest substring starting at position i that is also a prefix of the string. Finding a pattern in text becomes a single Z-computation on `pattern + "$" + text`.

```
String: AABXAAB
Z-array:[7, 1, 0, 0, 3, 1, 0]
         ↑                ↑
      Z[0]=n          Z[4]=3 because "AAB" matches the prefix
```

**O(n + m)** — linear. Conceptually simpler than KMP; builds the same information differently.

---

### Aho-Corasick

Extends KMP to search for **multiple patterns simultaneously** in a single pass through the text. Builds a finite automaton from all patterns combined.

```
Patterns: {"he", "she", "his", "hers"}
Text:     "ushers"

Single O(n + m + z) pass finds:
  "she" at position 1
  "he"  at position 2
  "hers" at position 2
  (z = total number of matches)
```

**Construction:** O(total pattern length × alphabet size)  
**Search:** O(text length + total matches)

Used in: antivirus scanning (scan for thousands of virus signatures at once), network intrusion detection, plagiarism detection.

---

### Suffix Array + LCP Array

A **suffix array** is the sorted array of all suffixes of a string. Combined with the **Longest Common Prefix (LCP) array**, it enables O(m log n) pattern search and O(n) solutions to problems like longest repeated substring.

```
Text: "banana"
Suffixes sorted:
  0: "a"
  1: "ana"
  2: "anana"
  3: "banana"
  4: "na"
  5: "nana"

Search "ana": binary search in suffix array → found at positions 1,3
```

**Construction:** O(n log n) with prefix doubling, O(n) with SA-IS.  
**Search:** O(m log n) via binary search on the suffix array.  
Used in: genome sequencing, full-text search engines, data compression (BWT).

---

## Bidirectional Search

Run **two simultaneous BFS** — one from the start, one from the goal — meeting in the middle. Dramatically reduces the search space.

```
Standard BFS explores: b^d nodes
Bidirectional BFS:     2 × b^(d/2) nodes

For b=10, d=6:
  Standard:       10^6 = 1,000,000
  Bidirectional:  2 × 10^3 = 2,000  ← 500× faster
```

Used in: Google Maps routing, social network "degrees of separation", word ladder puzzles.

---

## Iterative Deepening DFS (IDDFS)

Combines DFS's **O(1) space** with BFS's **shortest-path guarantee**. Runs DFS with increasing depth limits: 1, 2, 3, … until the target is found.

```
Depth limit 1: explore all nodes at depth ≤ 1
Depth limit 2: explore all nodes at depth ≤ 2
...
Depth limit d: target found
```

Yes, it re-explores nodes — but each level has more nodes than all previous levels combined, so the repeated work is only a small constant overhead. **Optimal for tree search when branching factor is large and depth is unknown.**

Used in: IDA* (IDDFS + heuristic — memory-efficient A*), chess engines, puzzle solvers.

---

## Bloom Filter Search

A **probabilistic** membership structure that answers "is X in the set?" using multiple hash functions and a bit array.

```
Insert "alice":  h1("alice")=3, h2("alice")=7, h3("alice")=11 → set bits 3,7,11
Query  "alice":  check bits 3,7,11 → all 1 → PROBABLY in set
Query  "bob":    h1("bob")=3, h2("bob")=5, h3("bob")=9 → bit 5 = 0 → DEFINITELY NOT in set
```

| Response | Meaning |
|---|---|
| **No** | Element is **guaranteed** absent |
| **Yes** | Element is **probably** present (false positive rate ε) |

False positive rate: `ε ≈ (1 - e^(-kn/m))^k`  
Optimal k (hash functions): `k = (m/n) ln 2`

**O(k)** per operation — effectively O(1) since k is a small constant.  
No false negatives. Cannot delete (use **Counting Bloom Filter** for deletions).

**Used in:** Chrome Safe Browsing (skip database lookup for known-safe URLs), Cassandra/HBase/LevelDB (avoid disk reads for missing keys), CDN cache filtering, spell checkers.

---

## How to Choose a Searching Algorithm

```
Data structure is a hash table?
  └── Hash Search — O(1) average. Best for exact key lookup.

Data is a sorted array?
  ├── General case         → Binary Search
  ├── Uniformly distributed → Interpolation Search (O(log log n))
  ├── Array is unbounded   → Exponential Search then Binary Search
  └── Need unimodal min/max → Ternary Search

Data is unsorted / no structure?
  └── Linear Search (or sort first if you'll search many times)

Data is a graph or tree?
  ├── Unweighted, need shortest path  → BFS
  ├── Any path / cycle detection      → DFS
  ├── Weighted, no negative edges     → Dijkstra
  ├── Weighted + heuristic available  → A*
  ├── Negative edge weights           → Bellman-Ford
  └── All-pairs shortest paths        → Floyd-Warshall

Searching for a string pattern?
  ├── Single pattern, large text      → Boyer-Moore (fastest in practice)
  ├── Single pattern, streaming text  → KMP
  ├── Multiple patterns at once       → Aho-Corasick
  ├── Many searches on same text      → Suffix Array
  └── Rolling hash / plagiarism       → Rabin-Karp

Need probabilistic membership (huge sets, RAM-constrained)?
  └── Bloom Filter

Searching strings with prefix queries?
  └── Trie

Search space is huge, goal state known, good heuristic available?
  └── A* or IDA* (memory-constrained)

Graph is huge, need to meet in the middle?
  └── Bidirectional BFS
```

---

## Quick Reference Card

```
ARRAY SEARCH                    GRAPH SEARCH
  Linear      O(n)    unsorted    BFS        O(V+E)  shortest path (unweighted)
  Binary      O(log n) sorted     DFS        O(V+E)  any path, cycle detect
  Interpolate O(log log n) uniform Dijkstra  O((V+E)logV) weighted, no neg
  Jump        O(√n)   sorted      A*         O(E)    weighted + heuristic
  Exponential O(log n) unbounded  Bellman-F  O(VE)   negative weights
  Fibonacci   O(log n) sorted     Floyd-W    O(V³)   all-pairs shortest path
  Ternary     O(log n) unimodal

STRING SEARCH                   SPECIALIZED
  Naïve       O(nm)              Hash Table  O(1) avg   exact key
  KMP         O(n+m)  single     BST         O(log n)   ordered keys
  Boyer-Moore O(n/m)  single     Trie        O(m)       prefix / string keys
  Rabin-Karp  O(n+m)  rolling    Bloom Filter O(k)      probabilistic membership
  Z-Algorithm O(n+m)  single     Suffix Array O(m log n) text preprocessing
  Aho-Corasick O(n+m+z) multi    Bidirectional O(b^d/2) meet in the middle
                                 IDDFS       O(b^d)     DFS + BFS guarantee
```
