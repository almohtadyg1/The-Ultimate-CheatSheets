# Data Structures: A Complete Progressive Tutorial

---

## 1. What & Why

A data structure is a way of organizing data in memory so that specific operations — look up a value, insert a new one, delete one, find the minimum — can be performed efficiently.

The word "efficiently" is doing all the work in that definition. The same problem solved with different data structures can run in 1 millisecond or 10 minutes, depending on the input size. This is not a minor optimization concern. It is the central engineering decision in algorithm design.

Consider: you have a list of 10 million user IDs and you need to check whether a given ID exists. Using an array and scanning it takes 10 million comparisons in the worst case. Using a hash set, it takes roughly 1 comparison. At 10,000 lookups per second, the array approach takes 1,000 seconds. The hash set takes 1 second. Same problem, same data, same hardware — orders of magnitude difference from one structural choice.

Why do different structures have different performance characteristics? Because each makes different trade-offs in how data is laid out in memory, how pointers connect nodes, and which operations are treated as first-class. Understanding those trade-offs is this tutorial.

---

## 2. Mental Model

Think of data structures as different types of storage containers, each with different rules for how you can access what's inside:

```
Array        = numbered slots in a parking lot
              Jump directly to slot 42 in O(1).
              Moving cars to insert between slots: O(n).

Linked List  = a scavenger hunt
              Each note tells you where the next clue is.
              To reach clue 42: follow 42 links. O(n).
              But adding a clue: just update two pointers. O(1).

Hash Table   = a library card catalog
              The card (hash function) tells you exactly which shelf (bucket).
              Lookup is nearly instant regardless of collection size. O(1) avg.

Binary Tree  = a sorted filing cabinet that halves the search space every decision
              Is it left or right of this divider? Repeat. O(log n).

Heap         = a sorted pile where you only need to see the top
              You care about the min/max, not the rest. O(1) peek, O(log n) insert.

Graph        = a road network
              Cities are nodes, roads are edges. Routing algorithms navigate it.
```

The right container depends on which operations you perform most. Optimize for your bottleneck.

---

## 3. Progressive Examples

### Level 1: Array — The Baseline

```python
# An array is a contiguous block of memory. Every element is the same size,
# so the address of element[i] = base_address + i * element_size.
# This makes indexing O(1) — no traversal needed.

temperatures = [72, 68, 75, 80, 65, 71]

# O(1): direct index access
today = temperatures[3]          # 80 — direct memory calculation, no traversal

# O(n): search requires scanning (no shortcut in unsorted data)
def contains(arr, target):
    for val in arr:              # must check each element
        if val == target:
            return True
    return False

# O(n): inserting in the middle requires shifting everything right
temperatures.insert(2, 73)       # insert 73 at index 2
# Before: [72, 68, 75, 80, 65, 71]
# After:  [72, 68, 73, 75, 80, 65, 71]
# Every element from index 2 onward moved one position right.

# O(1) amortized: append at the end — Python's list handles resizing
temperatures.append(79)

# When to use: indexed access is frequent, insertions/deletions are rare or at the end.
# Real-world: storing pixel RGB values, time-series data, lookup tables.
```

### Level 2: Linked List — When Insertions Are Cheap

```python
class Node:
    def __init__(self, value):
        self.value = value
        self.next = None         # pointer to the next node

class LinkedList:
    def __init__(self):
        self.head = None         # the entry point to the list

    def prepend(self, value):
        """O(1) — insert at the front. Just rewire two pointers."""
        new_node = Node(value)
        new_node.next = self.head   # new node points to old head
        self.head = new_node        # head now points to new node

    def append(self, value):
        """O(n) without a tail pointer — must walk to the end."""
        new_node = Node(value)
        if not self.head:
            self.head = new_node
            return
        current = self.head
        while current.next:          # walk until we reach the last node
            current = current.next
        current.next = new_node      # attach new node

    def delete(self, value):
        """O(n) to find the node, O(1) to delete once found."""
        if not self.head:
            return
        if self.head.value == value:  # special case: deleting the head
            self.head = self.head.next
            return
        current = self.head
        while current.next and current.next.value != value:
            current = current.next
        if current.next:              # node found — bypass it
            current.next = current.next.next
            # The node is now unreachable — Python GC will collect it

    def to_list(self):
        result = []
        current = self.head
        while current:
            result.append(current.value)
            current = current.next
        return result

# Usage
ll = LinkedList()
ll.prepend(3)
ll.prepend(2)
ll.prepend(1)
print(ll.to_list())   # [1, 2, 3]
ll.delete(2)
print(ll.to_list())   # [1, 3]

# Key insight: linked lists shine when you frequently insert/delete at known positions.
# They suffer when you need to access element[42] — that's 42 pointer dereferences.
```

### Level 3: Stack and Queue — Behavioral Contracts

```python
# Stack: Last In, First Out (LIFO)
# Think: a stack of plates. You add and remove from the top.
# Implemented efficiently with a Python list (append/pop are both O(1) at the end).

class Stack:
    def __init__(self):
        self._data = []

    def push(self, item):
        self._data.append(item)     # O(1) amortized

    def pop(self):
        if not self._data:
            raise IndexError("Stack is empty")
        return self._data.pop()     # O(1)

    def peek(self):
        if not self._data:
            raise IndexError("Stack is empty")
        return self._data[-1]       # O(1) — look without removing

    def is_empty(self):
        return len(self._data) == 0

# Real-world stack use: validating bracket pairs
def is_balanced(expression):
    """Check if brackets are properly nested: ({[]}) is valid, ({[}) is not."""
    stack = Stack()
    matching = {')': '(', ']': '[', '}': '{'}

    for char in expression:
        if char in '([{':
            stack.push(char)              # opening bracket: push it
        elif char in ')]}':
            if stack.is_empty():
                return False              # closing bracket with nothing open
            if stack.pop() != matching[char]:
                return False              # mismatched pair
    return stack.is_empty()              # all opened brackets were closed

print(is_balanced("({[]})"))   # True
print(is_balanced("({[})"))    # False

# Queue: First In, First Out (FIFO)
# Think: a line at a coffee shop. First customer served first.
# Use collections.deque — O(1) append and popleft (list.pop(0) is O(n)).

from collections import deque

class Queue:
    def __init__(self):
        self._data = deque()

    def enqueue(self, item):
        self._data.append(item)          # add to the right (back of queue)

    def dequeue(self):
        if not self._data:
            raise IndexError("Queue is empty")
        return self._data.popleft()      # remove from the left (front) — O(1)

    def peek(self):
        return self._data[0]

    def is_empty(self):
        return len(self._data) == 0

# Real-world queue use: BFS (Breadth-First Search)
from collections import defaultdict

def bfs(graph, start):
    """Visit all nodes level by level — closest nodes first."""
    visited = set()
    queue = Queue()
    queue.enqueue(start)
    visited.add(start)
    order = []

    while not queue.is_empty():
        node = queue.dequeue()
        order.append(node)
        for neighbor in graph[node]:     # add unvisited neighbors to back of queue
            if neighbor not in visited:
                visited.add(neighbor)
                queue.enqueue(neighbor)
    return order

graph = {'A': ['B', 'C'], 'B': ['D'], 'C': ['D', 'E'], 'D': [], 'E': []}
print(bfs(graph, 'A'))   # ['A', 'B', 'C', 'D', 'E']
```

### Level 4: Hash Table — The O(1) Lookup Machine

```python
# Python's dict IS a hash table. Understanding how it works explains its behavior.

# Basic usage — you already know this
user_scores = {}
user_scores["alice"] = 95
user_scores["bob"] = 82
print(user_scores.get("carol", 0))   # 0 — default if key missing

# What a hash table does internally:
# 1. Compute hash("alice") -> some integer, e.g., 7391846234
# 2. Take integer % table_size (e.g., % 8) -> index 6
# 3. Store (key, value) pair at index 6
# 4. On lookup: same hash -> same index -> direct array access -> O(1)

# Common hash table patterns

# Frequency counting — O(n) to build, O(1) to query
def char_frequency(text):
    freq = {}
    for char in text:
        freq[char] = freq.get(char, 0) + 1   # increment or start at 0
    return freq

print(char_frequency("mississippi"))
# {'m': 1, 'i': 4, 's': 4, 'p': 2}

# Alternatively, use Counter (optimized hash table for counting)
from collections import Counter
print(Counter("mississippi"))

# Grouping by key — hash table maps key -> list of values
def group_by_length(words):
    groups = defaultdict(list)
    for word in words:
        groups[len(word)].append(word)
    return dict(groups)

words = ["cat", "dog", "elephant", "rat", "cow", "ant"]
print(group_by_length(words))
# {3: ['cat', 'dog', 'rat', 'cow', 'ant'], 8: ['elephant']}

# Two-sum problem: find two numbers that add to target — O(n) with hash table
def two_sum(nums, target):
    seen = {}   # value -> index
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:              # O(1) lookup
            return [seen[complement], i]
        seen[num] = i                       # O(1) insert
    return []

print(two_sum([2, 7, 11, 15], 9))   # [0, 1] — nums[0] + nums[1] == 9

# When hash tables hurt: they use more memory than arrays, have poor cache behavior
# for large tables, and worst-case O(n) if many keys collide.
# Never use a dict when an array with integer keys will do.
```

### Level 5: Binary Search Tree and Heap

```python
# Binary Search Tree: sorted structure for O(log n) search, insert, delete
# Invariant: left subtree < node < right subtree

class BSTNode:
    def __init__(self, value):
        self.value = value
        self.left = None
        self.right = None

class BST:
    def __init__(self):
        self.root = None

    def insert(self, value):
        self.root = self._insert(self.root, value)

    def _insert(self, node, value):
        if node is None:
            return BSTNode(value)           # base case: found the empty slot
        if value < node.value:
            node.left = self._insert(node.left, value)    # go left
        elif value > node.value:
            node.right = self._insert(node.right, value)  # go right
        # if equal: ignore (or handle duplicates per your policy)
        return node

    def search(self, value):
        return self._search(self.root, value)

    def _search(self, node, value):
        if node is None:
            return False                    # reached a dead end: not found
        if value == node.value:
            return True
        if value < node.value:
            return self._search(node.left, value)
        return self._search(node.right, value)

    def inorder(self):
        """Inorder traversal yields elements in sorted order."""
        result = []
        self._inorder(self.root, result)
        return result

    def _inorder(self, node, result):
        if node is None:
            return
        self._inorder(node.left, result)    # visit left subtree first
        result.append(node.value)           # then current node
        self._inorder(node.right, result)   # then right subtree

bst = BST()
for val in [8, 3, 10, 1, 6, 14, 4, 7]:
    bst.insert(val)

print(bst.inorder())    # [1, 3, 4, 6, 7, 8, 10, 14] — sorted automatically
print(bst.search(6))    # True
print(bst.search(5))    # False

# WARNING: if you insert sorted data [1, 2, 3, 4, 5...] into a plain BST,
# it degenerates into a linked list and all operations become O(n).
# Solution: use a self-balancing BST (AVL, Red-Black) or Python's sortedcontainers.

# Heap: get the minimum (or maximum) in O(1), maintain order in O(log n)
import heapq   # Python's built-in min-heap

tasks = []
heapq.heappush(tasks, (3, "low priority task"))
heapq.heappush(tasks, (1, "urgent task"))
heapq.heappush(tasks, (2, "normal task"))

while tasks:
    priority, task = heapq.heappop(tasks)   # always pops the MINIMUM priority
    print(f"Processing [{priority}]: {task}")
# Processing [1]: urgent task
# Processing [2]: normal task
# Processing [3]: low priority task

# Top-k elements (the most practical heap use case)
def top_k_frequent(nums, k):
    counts = Counter(nums)
    # heapq.nlargest uses a heap internally — O(n log k)
    return [num for num, _ in heapq.nlargest(k, counts.items(), key=lambda x: x[1])]

print(top_k_frequent([1,1,1,2,2,3], 2))   # [1, 2]
```

### Level 6: Graph Representation and Traversal

```python
from collections import defaultdict, deque

class Graph:
    """Adjacency list representation — efficient for sparse graphs (most real-world graphs)."""

    def __init__(self, directed=False):
        self.adj = defaultdict(list)
        self.directed = directed

    def add_edge(self, u, v, weight=1):
        self.adj[u].append((v, weight))
        if not self.directed:
            self.adj[v].append((u, weight))

    def bfs(self, start):
        """
        Breadth-First Search: explores all nodes at distance 1, then 2, then 3...
        Guarantees shortest path (in terms of number of edges) in unweighted graphs.
        """
        visited = {start: True}
        queue = deque([start])
        distances = {start: 0}

        while queue:
            node = queue.popleft()
            for neighbor, _ in self.adj[node]:
                if neighbor not in visited:
                    visited[neighbor] = True
                    distances[neighbor] = distances[node] + 1
                    queue.append(neighbor)
        return distances

    def dfs(self, start):
        """
        Depth-First Search: dives as deep as possible before backtracking.
        Used for: topological sort, cycle detection, finding connected components.
        """
        visited = set()
        order = []

        def _dfs(node):
            visited.add(node)
            order.append(node)
            for neighbor, _ in self.adj[node]:
                if neighbor not in visited:
                    _dfs(neighbor)

        _dfs(start)
        return order

    def dijkstra(self, start):
        """
        Dijkstra's algorithm: shortest path in a weighted graph (non-negative weights only).
        Uses a min-heap as a priority queue for O((V + E) log V).
        """
        import heapq
        distances = {node: float('inf') for node in self.adj}
        distances[start] = 0
        heap = [(0, start)]    # (distance, node)

        while heap:
            dist, node = heapq.heappop(heap)

            if dist > distances[node]:    # stale entry — skip
                continue

            for neighbor, weight in self.adj[node]:
                new_dist = dist + weight
                if new_dist < distances[neighbor]:
                    distances[neighbor] = new_dist
                    heapq.heappush(heap, (new_dist, neighbor))
        return distances

# Example: city road network
city_map = Graph(directed=False)
city_map.add_edge("Cairo", "Alexandria", 225)
city_map.add_edge("Cairo", "Suez", 134)
city_map.add_edge("Alexandria", "Marsa Matruh", 290)
city_map.add_edge("Suez", "Hurghada", 195)

distances = city_map.dijkstra("Cairo")
print(distances)
# {'Cairo': 0, 'Alexandria': 225, 'Suez': 134, 'Marsa Matruh': 515, 'Hurghada': 329}
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Using a list for O(1) lookup when you need a set or dict**

```python
# WRONG: O(n) membership test every iteration
valid_users = ["alice", "bob", "carol", "dave"]  # a list
def is_valid(user):
    return user in valid_users   # scans the entire list every time

# For 1000 users and 100,000 lookups: 100,000,000 comparisons

# CORRECT: O(1) membership test with a set
valid_users = {"alice", "bob", "carol", "dave"}   # a set (hash table internally)
def is_valid(user):
    return user in valid_users   # O(1) hash lookup
```

**Mistake 2: Inserting sorted data into an unbalanced BST**

```python
# WRONG: creates a degenerate "linked list" tree
bst = BST()
for i in range(1, 1001):    # sorted input
    bst.insert(i)
# The tree is now 1000 nodes deep, all going right
# bst.search(500) takes 499 steps — O(n), not O(log n)

# CORRECT: use Python's sortedcontainers for a balanced sorted structure
from sortedcontainers import SortedList
sl = SortedList(range(1, 1001))
sl.index(500)    # O(log n) guaranteed
```

**Mistake 3: Using `list.pop(0)` to implement a queue**

```python
# WRONG: O(n) per dequeue — every element shifts left
queue = [1, 2, 3, 4, 5]
queue.pop(0)   # removes 1, but shifts [2,3,4,5] left — O(n)

# For a queue of 100,000 elements: 100,000 × 50,000 avg shifts = 5 billion ops

# CORRECT: use collections.deque
from collections import deque
queue = deque([1, 2, 3, 4, 5])
queue.popleft()   # O(1) — deque is designed for this
```

**Mistake 4: Assuming hash table operations are always O(1)**

```python
# Hash tables have worst-case O(n) when many keys collide into the same bucket.
# This is rare with good hash functions but critical to understand.

# Also: dict operations are NOT free. For tight inner loops:
d = {}
for i in range(10_000_000):
    d[i] = i * 2    # each is O(1) but hashing has real constant-time overhead

# For numeric indexes, an array is faster
arr = [0] * 10_000_000
for i in range(10_000_000):
    arr[i] = i * 2  # no hashing — direct memory write

# Profile your actual bottleneck before assuming dict is "good enough"
```

**Mistake 5: Modifying a data structure while iterating over it**

```python
# WRONG: undefined behavior (or RuntimeError in Python)
scores = {"alice": 10, "bob": 0, "carol": 5, "dave": 0}
for user, score in scores.items():
    if score == 0:
        del scores[user]   # RuntimeError: dictionary changed size during iteration

# CORRECT: iterate over a copy
for user, score in list(scores.items()):   # list() materializes the snapshot
    if score == 0:
        del scores[user]

# OR: build a new dict
scores = {user: score for user, score in scores.items() if score != 0}
```

**Mistake 6: Forgetting that heap order is not full sort order**

```python
import heapq

nums = [3, 1, 4, 1, 5, 9, 2, 6]
heapq.heapify(nums)
print(nums)     # [1, 1, 2, 6, 5, 9, 4, 3]
# This is NOT sorted. The heap property only guarantees:
# heap[0] is the minimum (the root)
# For any i: heap[i] <= heap[2i+1] and heap[i] <= heap[2i+2]
# The rest of the array is NOT in sorted order

# If you need all elements sorted, use sorted() — O(n log n)
# If you only need the minimum repeatedly, use heappop() — O(log n) per pop
```

---

## 5. The "Why Does This Work" Layer

### Why Hash Table Lookup Is O(1)

The magic is that the hash function converts an arbitrary key into an integer, and modulo converts that integer into an array index. Accessing an array by index is O(1) because array memory is contiguous: `address = base + index * element_size` is a single arithmetic operation, not a traversal.

```
hash("alice") = 7391846234          # deterministic: same input, always same output
7391846234 % 8 = 2                  # modulo maps to a valid index
array[2] = ("alice", 95)            # O(1) read — single memory access
```

The O(1) average case holds only if the hash function distributes keys uniformly. If 1000 keys all hash to index 2, every operation on that bucket is O(1000). Good hash functions (MurmurHash, xxHash) are designed to minimize this. Python's `dict` resizes when the load factor (entries / buckets) exceeds 2/3, which maintains uniform distribution.

### Why Heap Insert Is O(log n), Not O(n)

A heap is stored as an array where the parent-child relationship is encoded by index arithmetic: parent of `i` is `(i-1)//2`, children of `i` are `2i+1` and `2i+2`. When you insert a new element:

1. Append it to the end of the array. O(1).
2. "Sift up": compare with parent. If it violates the heap property, swap. Repeat.

How many swaps can happen? The heap is a complete binary tree of n elements, so its height is `floor(log₂(n))`. Sifting up travels at most one path from leaf to root — at most `log₂(n)` swaps. Hence O(log n).

```
Initial heap (max-heap): [90, 70, 80, 40, 60, 30, 50]
Insert 95 at end:        [90, 70, 80, 40, 60, 30, 50, 95]
                                                         ^
Sift up: 95 > parent 40? Swap.   [90, 70, 80, 95, 60, 30, 50, 40]
Sift up: 95 > parent 70? Swap.   [90, 95, 80, 70, 60, 30, 50, 40]
Sift up: 95 > parent 90? Swap.   [95, 90, 80, 70, 60, 30, 50, 40]
Done. 3 swaps for n=8: log₂(8) = 3.
```

### Why Dynamic Array Append Is O(1) Amortized

A Python list starts with some capacity (say 4). When you append beyond capacity, it allocates a new array of ~2× the size and copies everything. This copy is O(n) — but it happens rarely. If you appended n elements total:

- You triggered resizes at sizes 4, 8, 16, 32, ..., n
- Total copy work: 4 + 8 + 16 + ... + n = 2n - 4 ≈ 2n
- Spread over n appends: 2n / n = 2 operations per append on average

The amortized cost is O(1). The occasional O(n) resize is paid for by the n cheap appends that preceded it.

---

## 6. Quick Reference

### When to Use What

| Requirement | Structure | Reason |
|------------|-----------|--------|
| Fast lookup by key | `dict` / hash table | O(1) average |
| Sorted data + range queries | BST / `SortedList` | O(log n) all ops |
| Repeated min/max access | Heap / `heapq` | O(1) peek, O(log n) update |
| LIFO (undo, call stack, DFS) | Stack | O(1) push/pop |
| FIFO (BFS, task queue) | Queue / `deque` | O(1) enqueue/dequeue |
| Prefix search / autocomplete | Trie | O(m) where m = key length |
| Range queries + point updates | Segment Tree | O(log n) |
| Connectivity / grouping | Union-Find | O(α(n)) ≈ O(1) |
| Membership, low memory | Bloom Filter | O(k) hash ops, probabilistic |
| Bounded cache with eviction | LRU Cache | O(1) get/put |
| Indexed sequence access | Array | O(1) |
| Frequent head/tail insert | Linked List / deque | O(1) |

### Complexity Master Table

| Structure | Access | Search | Insert | Delete | Space |
|-----------|--------|--------|--------|--------|-------|
| Array | O(1) | O(n) | O(n) | O(n) | O(n) |
| Dynamic Array | O(1) | O(n) | O(1)* | O(n) | O(n) |
| Linked List | O(n) | O(n) | O(1)† | O(1)† | O(n) |
| Stack | O(n) | O(n) | O(1) | O(1) | O(n) |
| Queue | O(n) | O(n) | O(1) | O(1) | O(n) |
| Hash Table | — | O(1)* | O(1)* | O(1)* | O(n) |
| BST (balanced) | O(log n) | O(log n) | O(log n) | O(log n) | O(n) |
| Heap | O(1)‡ | O(n) | O(log n) | O(log n) | O(n) |
| Trie | O(m) | O(m) | O(m) | O(m) | O(n·m) |
| Segment Tree | — | O(log n) | O(log n) | O(log n) | O(n) |
| Union-Find | — | O(α(n)) | O(α(n)) | — | O(n) |

\* Amortized &nbsp;&nbsp; † At known position &nbsp;&nbsp; ‡ Min or Max only
