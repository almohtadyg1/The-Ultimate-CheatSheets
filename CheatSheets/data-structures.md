# Data Structures Cheat Sheet
### From Zero to Expert

---

## What is a Data Structure?

A **data structure** is a way of organizing, storing, and relating data in memory so that operations on it — access, search, insert, delete — can be performed efficiently.

Choosing the right data structure is often the difference between an algorithm that runs in milliseconds and one that times out.

```
The same problem. Two structures. Two outcomes:
  Linear search in an Array:     O(n)   → 1,000,000 ops for n=1M
  Lookup in a Hash Table:        O(1)   → ~1 op for n=1M
```

---

## Complexity Reference

Every structure below is evaluated against these operations. Always know your use case before choosing.

| Notation | Meaning |
|---|---|
| `O(1)` | Constant — doesn't grow with input size |
| `O(log n)` | Logarithmic — halves the problem each step |
| `O(n)` | Linear — grows proportionally |
| `O(n log n)` | Linearithmic — common in good sorting |
| `O(n²)` | Quadratic — nested loops over input |
| `O(2ⁿ)` | Exponential — avoid at scale |

---

## Array

The most fundamental structure. A **contiguous block of memory** holding elements of the same type, accessed by index.

```
Index:   0    1    2    3    4
Value: [ 10 | 25 | 7  | 42 | 3 ]
         ↑
       base address
       each element = base + (index × element_size)
```

| Operation | Time | Notes |
|---|---|---|
| Access by index | O(1) | Direct memory calculation |
| Search (unsorted) | O(n) | Must scan every element |
| Search (sorted) | O(log n) | Binary search |
| Insert at end | O(1) amortized | May trigger resize |
| Insert at middle | O(n) | Must shift elements right |
| Delete at middle | O(n) | Must shift elements left |

**Use when:** You need fast indexed access, predictable memory layout, or cache efficiency.  
**Avoid when:** You need frequent insertions/deletions in the middle.

### Dynamic Array (ArrayList / Python list / Vec)
Starts with a fixed capacity. When full, allocates a new block (~2× size) and copies.  
Amortized O(1) append because doubling means each element is copied ≤ 2 times on average.

---

## Linked List

A sequence of **nodes**, each storing a value and a pointer to the next node. Memory is non-contiguous.

```
Singly Linked:
[10 | •]──►[25 | •]──►[7 | •]──►[42 | null]
 head

Doubly Linked:
null◄─[10 | •]◄──►[25 | •]◄──►[7 | •]──►null
       head                      tail
```

| Operation | Singly | Doubly | Notes |
|---|---|---|---|
| Access by index | O(n) | O(n) | No random access |
| Search | O(n) | O(n) | Linear scan |
| Insert at head | O(1) | O(1) | Just rewire pointers |
| Insert at tail | O(n) / O(1)* | O(1) | O(1) if tail pointer kept |
| Insert at middle | O(n) | O(n) | Finding position is O(n) |
| Delete at head | O(1) | O(1) | |
| Delete known node | O(n) | O(1) | Doubly can unlink itself |

**Use when:** Frequent insertions/deletions at the head or tail; building stacks/queues.  
**Avoid when:** You need random access or cache efficiency.

### Variants
- **Circular linked list** — tail's next points back to head. Useful for round-robin scheduling.
- **Skip list** — layered linked lists enabling O(log n) search. Used in Redis, LevelDB.

---

## Stack

A **Last In, First Out (LIFO)** structure. Only the top element is accessible.

```
push(A) push(B) push(C)       pop() → C
                               pop() → B
    [C]  ← top                pop() → A
    [B]
    [A]
```

| Operation | Time |
|---|---|
| Push | O(1) |
| Pop | O(1) |
| Peek (top) | O(1) |
| Search | O(n) |

**Implemented via:** Array (with a top pointer) or singly linked list.

**Use when:** Function call management, undo/redo, expression evaluation, DFS traversal, balancing brackets.

```
Classic use — balanced parentheses:
  "({[]})"  → valid    ← each closing bracket matches last opened
  "({[})"   → invalid
```

---

## Queue

A **First In, First Out (FIFO)** structure. Elements enter at the back and leave from the front.

```
enqueue →  [A][B][C][D]  → dequeue
           back      front
```

| Operation | Time |
|---|---|
| Enqueue | O(1) |
| Dequeue | O(1) |
| Peek (front) | O(1) |
| Search | O(n) |

**Implemented via:** Circular array (avoids shifting) or doubly linked list.

**Use when:** BFS traversal, task scheduling, print spoolers, message queues, buffering.

### Variants

**Deque (Double-Ended Queue)** — insert and remove from both ends in O(1). Python's `collections.deque`.

**Priority Queue** — dequeue returns the element with the **highest priority**, not the oldest. Implemented with a heap (see below).

---

## Hash Table

Maps **keys to values** using a hash function. The backbone of dictionaries, sets, caches, and indexes.

```
key "alice"  →  hash()  →  index 3  →  [ → ("alice", 99) ]
key "bob"    →  hash()  →  index 7  →  [ → ("bob", 82)   ]
key "carol"  →  hash()  →  index 3  →  COLLISION at index 3
```

| Operation | Average | Worst |
|---|---|---|
| Insert | O(1) | O(n) |
| Delete | O(1) | O(n) |
| Lookup | O(1) | O(n) |

Worst case occurs when many keys collide into the same bucket.

### Hash Functions
A good hash function is **fast**, **deterministic**, and **distributes uniformly**.

```
index = hash(key) % table_size
```

Common algorithms: MurmurHash, FNV, xxHash, SipHash (secure).

### Collision Resolution

**Chaining** — each bucket holds a linked list of all entries that hash to it.  
Simple but pointer-heavy; performance degrades as chains grow long.

**Open Addressing** — on collision, probe for the next free slot.
- *Linear probing*: check index+1, index+2, … (causes clustering)
- *Quadratic probing*: check index+1², index+2², … (reduces clustering)
- *Double hashing*: use a second hash function as the step size (best distribution)

### Load Factor & Resizing
```
load factor α = n / m     (n = entries, m = buckets)
```
When α exceeds a threshold (~0.7), resize to ~2× and **rehash** all entries.  
Python dicts resize at α = 0.67. Java HashMap at α = 0.75.

**Use when:** Fast lookup by key, deduplication, counting frequencies, caching.  
**Avoid when:** You need ordered iteration, range queries, or worst-case guarantees.

---

## Tree

A hierarchical structure of **nodes** connected by edges, with one **root** and no cycles.

```
            [10]          ← root (depth 0)
           /    \
        [5]     [15]      ← depth 1
       /   \       \
     [3]   [7]    [20]    ← depth 2 (leaves)
```

**Terminology:**
- **Root** — the top node with no parent
- **Leaf** — a node with no children
- **Height** — longest path from root to a leaf
- **Depth** — distance from the root to a node
- **Subtree** — a node and all its descendants

---

## Binary Search Tree (BST)

A binary tree where for every node:  
- All values in the **left** subtree are **smaller**  
- All values in the **right** subtree are **larger**

```
         [8]
        /   \
      [3]   [10]
     /   \     \
   [1]   [6]  [14]
```

| Operation | Average | Worst (unbalanced) |
|---|---|---|
| Search | O(log n) | O(n) |
| Insert | O(log n) | O(n) |
| Delete | O(log n) | O(n) |
| Min / Max | O(log n) | O(n) |
| Inorder traversal | O(n) | O(n) |

**Inorder traversal of a BST gives sorted output.**

Worst case (O(n)) occurs when inserting already-sorted data — the tree degenerates into a linked list. Solved by **self-balancing trees**.

---

## Balanced BSTs

### AVL Tree
The original self-balancing BST. After every insert/delete, the tree **rotates** to ensure:
```
|height(left) - height(right)| ≤ 1   for every node
```
Guarantees O(log n) for all operations. More rigidly balanced than Red-Black, so slightly faster lookups but more rotations on insert/delete.

### Red-Black Tree
Each node is colored **red** or **black**. Rules ensure the tree stays approximately balanced:
1. Root is black
2. Red nodes cannot have red children
3. Every path from root to null has the same number of black nodes

Guarantees O(log n). Fewer rotations than AVL → faster inserts/deletes.  
Used in: **C++ `std::map`**, **Java `TreeMap`**, **Linux kernel scheduler (CFS)**.

### B-Tree
A generalization of BST where each node holds **multiple keys** and has **multiple children**.  
Designed for **disk-based storage** — a single node fills one disk page, minimizing I/O.

```
[10 | 20 | 30]
 /    |    |    \
[..]  [..] [..] [..]
```

Used in: **database indexes (MySQL InnoDB, PostgreSQL)**, **file systems (NTFS, ext4, HFS+)**.

**B+ Tree** — all values stored only in leaves; internal nodes hold only keys for routing. Leaves linked for efficient range queries. The most common database index structure.

---

## Heap

A **complete binary tree** satisfying the heap property, typically stored in an array.

**Max-heap:** Every parent ≥ its children. Root is the maximum.  
**Min-heap:** Every parent ≤ its children. Root is the minimum.

```
Max-heap:           Array representation:
      [90]          [90, 70, 80, 40, 60, 30, 50]
     /    \          0   1   2   3   4   5   6
  [70]    [80]
  /  \    /  \      Parent of i:      (i-1) / 2
[40][60][30][50]    Left child of i:   2i + 1
                    Right child of i:  2i + 2
```

| Operation | Time |
|---|---|
| Get max/min | O(1) |
| Insert | O(log n) |
| Delete max/min | O(log n) |
| Build heap from array | O(n) |
| Heapsort | O(n log n) |

**Use when:** Priority queues, scheduling, Dijkstra's / A* algorithms, top-k problems.

---

## Trie (Prefix Tree)

A tree where each **path from root to a node** represents a string prefix. Each node's children represent the next possible characters.

```
Stored words: "car", "card", "care", "cat", "bat"

        root
       /    \
     [c]    [b]
      |      |
     [a]    [a]
    /   \    |
  [r]   [t] [t]
  / \
[d] [e]
```

| Operation | Time |
|---|---|
| Insert | O(m) — m = key length |
| Search | O(m) |
| Prefix search | O(m + results) |
| Delete | O(m) |

**Use when:** Autocomplete, spell checkers, IP routing tables, dictionary lookups.  
**Trade-off:** High memory usage (many pointers per node). Mitigated by **compressed tries (radix trees)**.

---

## Graph

A set of **vertices (nodes)** connected by **edges**. The most general data structure — trees and linked lists are special cases of graphs.

```
Undirected:          Directed (Digraph):
  A — B               A ──► B
  |   |               ▲     |
  C — D               |     ▼
                      D ◄── C
```

**Weighted graph** — edges have associated costs (distances, times, capacities).

### Representations

**Adjacency Matrix** — 2D array where `matrix[i][j] = 1` (or weight) if edge exists.
```
    A  B  C  D
A [ 0  1  1  0 ]
B [ 1  0  0  1 ]
C [ 1  0  0  1 ]
D [ 0  1  1  0 ]
```
Space: O(V²). Edge lookup: O(1). Good for dense graphs.

**Adjacency List** — each vertex stores a list of its neighbors.
```
A → [B, C]
B → [A, D]
C → [A, D]
D → [B, C]
```
Space: O(V + E). Good for sparse graphs (most real-world graphs).

| | Adjacency Matrix | Adjacency List |
|---|---|---|
| Space | O(V²) | O(V + E) |
| Edge lookup | O(1) | O(degree) |
| Iterate neighbors | O(V) | O(degree) |
| Best for | Dense graphs | Sparse graphs |

### Graph Traversals

**BFS (Breadth-First Search)** — explores neighbors level by level using a queue.  
Finds **shortest path** in unweighted graphs.
```
Order from A:  A → B, C → D
```

**DFS (Depth-First Search)** — dives as deep as possible using a stack (or recursion).  
Used for topological sort, cycle detection, connected components.

### Key Graph Algorithms

| Problem | Algorithm | Complexity |
|---|---|---|
| Shortest path (unweighted) | BFS | O(V + E) |
| Shortest path (weighted, no neg) | Dijkstra | O((V + E) log V) |
| Shortest path (negative weights) | Bellman-Ford | O(VE) |
| All-pairs shortest path | Floyd-Warshall | O(V³) |
| Minimum spanning tree | Prim / Kruskal | O(E log V) |
| Topological sort | DFS / Kahn's | O(V + E) |
| Strongly connected components | Tarjan / Kosaraju | O(V + E) |
| Maximum flow | Ford-Fulkerson | O(VE²) |

---

## Disjoint Set (Union-Find)

Tracks a collection of elements partitioned into **disjoint (non-overlapping) sets**. Answers the question: *are these two elements in the same group?*

```
Initially: {0} {1} {2} {3} {4}

union(0,1): {0,1} {2} {3} {4}
union(2,3): {0,1} {2,3} {4}
union(0,2): {0,1,2,3} {4}

find(1) == find(3)?  → YES (same set)
find(1) == find(4)?  → NO  (different sets)
```

| Operation | Time (with optimizations) |
|---|---|
| Find | O(α(n)) ≈ O(1) |
| Union | O(α(n)) ≈ O(1) |

α(n) is the inverse Ackermann function — effectively constant for any real input.

**Optimizations:**  
- **Union by rank/size** — always attach the smaller tree under the larger  
- **Path compression** — flatten the tree during `find` so future lookups are faster

**Use when:** Kruskal's MST, network connectivity, detecting cycles, image segmentation.

---

## Segment Tree

A binary tree built over an array that supports **range queries** and **point updates** efficiently.

```
Array: [2, 4, 3, 1, 6, 7, 2, 5]

Segment tree (range sum):
              [30]           ← sum of [0..7]
            /      \
         [10]      [20]      ← [0..3] and [4..7]
        /    \    /    \
      [6]  [4] [13]  [7]
      / \  / \  / \  / \
     [2][4][3][1][6][7][2][5]
```

| Operation | Time |
|---|---|
| Build | O(n) |
| Range query | O(log n) |
| Point update | O(log n) |
| Range update (lazy) | O(log n) |

**Use when:** Range sum/min/max queries with updates. Competitive programming staple.  
**Lazy propagation** — defers range updates to avoid updating every node immediately.

---

## Fenwick Tree (Binary Indexed Tree / BIT)

A more compact alternative to a segment tree for **prefix sum queries** and **point updates**.  
Uses clever bit manipulation (`i & (-i)`) to navigate the structure.

```
Array:  [3, 2, -1, 6, 5, 4, -3, 3, 7, 2]
Query:  prefix_sum(6) = 3+2-1+6+5+4 = 19   → O(log n)
Update: add 5 to index 3                    → O(log n)
```

| Operation | Time |
|---|---|
| Build | O(n log n) |
| Prefix query | O(log n) |
| Point update | O(log n) |

**Use when:** Counting inversions, order statistics, frequency tables with updates. Simpler and faster constant than segment trees for prefix queries only.

---

## Bloom Filter

A **probabilistic** data structure that answers "is this element in the set?" with:
- **No** → definitely not in the set
- **Yes** → *probably* in the set (false positives possible, false negatives impossible)

```
Insert "alice":  hash1("alice")=3, hash2("alice")=7, hash3("alice")=12
                 set bits[3], bits[7], bits[12] = 1

Query "bob":     hash1("bob")=3, hash2("bob")=5, hash3("bob")=9
                 bits[5] = 0  → "bob" is DEFINITELY NOT in the set

Query "carol":   all three bits happen to be 1  → "carol" is PROBABLY in the set
```

**Space:** O(m) — a bit array of size m, far smaller than storing actual elements.  
**False positive rate:** ~(1 - e^(-kn/m))^k, decreases with larger m.

**Use when:** Cache filtering (avoid expensive lookups for items definitely absent), spam detection, network routers, Chrome's Safe Browsing, Cassandra/HBase/LevelDB.

---

## LRU Cache

A cache that evicts the **Least Recently Used** item when full. Implemented by combining:
- A **doubly linked list** — maintains usage order (most recent at head, LRU at tail)
- A **hash map** — maps keys to list nodes for O(1) access

```
Capacity: 3. Operations:
  get(A)   → miss. Add A.   List: [A]
  get(B)   → miss. Add B.   List: [B, A]
  get(C)   → miss. Add C.   List: [C, B, A]
  get(A)   → hit.  Move front. List: [A, C, B]
  get(D)   → miss. Evict B. List: [D, A, C]
```

| Operation | Time |
|---|---|
| Get | O(1) |
| Put | O(1) |

**Use when:** Any bounded cache needing recency-based eviction. Used in CPU caches, CDNs, database buffer pools, OS page replacement.

---

## Choosing the Right Structure

```
Need fast lookup by key?              → Hash Table
Need sorted data + range queries?     → BST / B+ Tree
Need min/max repeatedly?              → Heap (Priority Queue)
Need LIFO (undo, call stack)?         → Stack
Need FIFO (tasks, BFS)?               → Queue
Need prefix/autocomplete?             → Trie
Need range queries + updates?         → Segment Tree / Fenwick Tree
Need connectivity queries?            → Union-Find
Need huge set membership (low memory)?→ Bloom Filter
Need graph shortest path?             → BFS (unweighted) / Dijkstra (weighted)
Need bounded recency cache?           → LRU Cache
Need simple indexed access?           → Array
Need fast head/tail insert-delete?    → Linked List / Deque
```

---

## Complexity Master Table

| Structure | Access | Search | Insert | Delete | Space |
|---|---|---|---|---|---|
| Array | O(1) | O(n) | O(n) | O(n) | O(n) |
| Dynamic Array | O(1) | O(n) | O(1)* | O(n) | O(n) |
| Linked List | O(n) | O(n) | O(1)† | O(1)† | O(n) |
| Stack | O(n) | O(n) | O(1) | O(1) | O(n) |
| Queue | O(n) | O(n) | O(1) | O(1) | O(n) |
| Hash Table | N/A | O(1)* | O(1)* | O(1)* | O(n) |
| BST (balanced) | O(log n) | O(log n) | O(log n) | O(log n) | O(n) |
| Heap | O(1)‡ | O(n) | O(log n) | O(log n) | O(n) |
| Trie | O(m) | O(m) | O(m) | O(m) | O(n·m) |
| Segment Tree | N/A | O(log n) | O(log n) | O(log n) | O(n) |
| Union-Find | N/A | O(α(n)) | O(α(n)) | N/A | O(n) |

\* Amortized &nbsp;&nbsp; † At known position &nbsp;&nbsp; ‡ Min or Max only &nbsp;&nbsp; m = key length
