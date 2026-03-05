# The Complete Big O Notation Cheat Sheet
### From Zero to Expert

---

## 1. What is Big O?

Big O Notation describes how the **time or space requirements of an algorithm grow** as the input size `n` grows. It answers the question: *"If I double my input, what happens to my running time?"*

It is **not** about exact runtime in seconds — it's about the **rate of growth**. Constants and small terms are dropped because as `n` approaches infinity, they become irrelevant.

### Why It Matters

```
Input n = 1,000,000

O(log n)  →       20 operations   ← blazing fast
O(n)      →  1,000,000 operations ← fine
O(n log n)→ 20,000,000 operations ← acceptable
O(n²)     →  10^12 operations     ← will take years
O(2ⁿ)     →  10^301302 operations ← computationally impossible
```

### Formal Definition

`f(n) = O(g(n))` means there exist constants `c > 0` and `n₀` such that:

```
f(n) ≤ c · g(n)    for all n ≥ n₀
```

In plain English: `g(n)` is an **upper bound** on the growth of `f(n)` after some point. Big O describes the *worst case* unless otherwise stated.

---

## 2. The Complexity Classes

### Growth Rate Hierarchy (slowest → fastest growth)

```
O(1) < O(log log n) < O(log n) < O(√n) < O(n) < O(n log n) < O(n²) < O(n³) < O(2ⁿ) < O(n!) < O(nⁿ)
```

---

### O(1) — Constant Time

The algorithm takes the **same amount of time regardless of input size**.

```python
def get_first(arr):
    return arr[0]          # Always 1 operation

def is_even(n):
    return n % 2 == 0      # Always 1 operation

# Hash map lookup, array access by index, push/pop from stack
```

**Real world:** Looking up a word in a dictionary if you know the exact page number.

**Growth:** If `n` doubles → runtime stays the same.

---

### O(log n) — Logarithmic Time

The algorithm **halves (or divides by k) the problem size** with each step. Extremely efficient.

```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1       # Eliminate left half
        else:
            right = mid - 1      # Eliminate right half
    return -1
```

**Intuition:** `n = 1,024` takes ~10 steps because `log₂(1024) = 10`. Each step eliminates half.

**Real world:** Finding a name in a phone book by repeatedly opening to the middle.

**Growth:** If `n` doubles → runtime increases by just 1 step.

---

### O(√n) — Square Root Time

Appears in primality tests and some number theory algorithms.

```python
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n**0.5) + 1):  # Only need to check up to √n
        if n % i == 0:
            return False
    return True
```

**Growth:** If `n` goes from 100 to 10,000 → runtime goes from 10 to 100.

---

### O(n) — Linear Time

The algorithm visits **each input element a constant number of times**.

```python
def find_max(arr):
    max_val = arr[0]
    for x in arr:          # Visits every element once
        if x > max_val:
            max_val = x
    return max_val

def linear_search(arr, target):
    for i, x in enumerate(arr):
        if x == target:
            return i
    return -1
```

**Real world:** Reading every word on a page.

**Growth:** If `n` doubles → runtime doubles.

---

### O(n log n) — Linearithmic Time

The "sweet spot" for **comparison-based sorting**. Think of it as doing a log n operation n times, or doing a linear scan log n times.

```python
# Merge sort achieves O(n log n)
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    mid = len(arr) // 2
    left  = merge_sort(arr[:mid])   # Divide: log n levels
    right = merge_sort(arr[mid:])
    return merge(left, right)        # Conquer: O(n) per level

def merge(left, right):
    result = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i]); i += 1
        else:
            result.append(right[j]); j += 1
    return result + left[i:] + right[j:]
```

**Growth:** If `n` doubles → runtime slightly more than doubles.

---

### O(n²) — Quadratic Time

Typically appears with **nested loops** iterating over the same input. Gets slow fast.

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):           # Outer loop: n times
        for j in range(n - i - 1):  # Inner loop: ~n times
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]

def find_all_pairs(arr):
    pairs = []
    for i in range(len(arr)):      # n
        for j in range(len(arr)):  # n
            pairs.append((arr[i], arr[j]))
    return pairs
```

**Growth:** If `n` doubles → runtime **quadruples**. `n=10,000` → 100,000,000 ops.

---

### O(n³) — Cubic Time

Three nested loops. Appears in naive matrix multiplication, some DP problems.

```python
def matrix_multiply(A, B, C, n):
    for i in range(n):
        for j in range(n):
            for k in range(n):
                C[i][j] += A[i][k] * B[k][j]
```

**Growth:** If `n` doubles → runtime increases **8×**.

---

### O(2ⁿ) — Exponential Time

**Doubles with each added element**. Usually means trying all subsets. Only feasible for very small `n` (<30).

```python
# Naive recursive Fibonacci
def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)  # Branches into 2 at every call

# Generating all subsets (power set)
def power_set(arr):
    if not arr:
        return [[]]
    rest = power_set(arr[1:])
    return rest + [[arr[0]] + s for s in rest]
```

**Growth:** `n=20` → ~1M ops. `n=40` → ~1T ops. Each +1 to n **doubles** the work.

---

### O(n!) — Factorial Time

**Every permutation** of the input. Utterly infeasible beyond n ≈ 12.

```python
# Brute force Travelling Salesman Problem
# Try every possible route order
import itertools

def tsp_brute_force(cities, distances):
    best_cost = float('inf')
    for perm in itertools.permutations(cities):   # n! permutations
        cost = sum(distances[perm[i]][perm[i+1]] for i in range(len(perm)-1))
        best_cost = min(best_cost, cost)
    return best_cost
```

**Growth:** `n=10` → 3,628,800. `n=15` → 1,307,674,368,000.

---

### Complexity Growth Table

| n | O(1) | O(log n) | O(n) | O(n log n) | O(n²) | O(2ⁿ) | O(n!) |
|---|------|----------|------|------------|-------|--------|-------|
| 1 | 1 | 1 | 1 | 1 | 1 | 2 | 1 |
| 8 | 1 | 3 | 8 | 24 | 64 | 256 | 40,320 |
| 16 | 1 | 4 | 16 | 64 | 256 | 65,536 | 2.1 × 10¹³ |
| 32 | 1 | 5 | 32 | 160 | 1,024 | 4.3 × 10⁹ | 2.6 × 10³⁵ |
| 128 | 1 | 7 | 128 | 896 | 16,384 | 3.4 × 10³⁸ | 💀 |
| 1,024 | 1 | 10 | 1,024 | 10,240 | 1,048,576 | 💀 | 💀 |
| 10⁶ | 1 | 20 | 10⁶ | 2×10⁷ | 10¹² | 💀 | 💀 |

---

## 3. Rules for Calculating Big O

### Rule 1: Drop Constants

Constants don't affect growth rate.

```python
# O(2n) → O(n)
for x in arr:   # n
    print(x)
for x in arr:   # n again
    print(x)

# O(500) → O(1)
for i in range(500):
    print(i)
```

### Rule 2: Drop Non-Dominant Terms

Keep only the fastest-growing term.

```python
# O(n² + n) → O(n²)
for i in arr:         # O(n²)
    for j in arr:
        print(i, j)

for x in arr:         # O(n)  — irrelevant next to n²
    print(x)

# O(n³ + 2ⁿ) → O(2ⁿ)
# O(n + log n) → O(n)
# O(n² + n log n) → O(n²)
```

### Rule 3: Different Variables Stay Separate

If loops iterate over **different** inputs, use different variables.

```python
# NOT O(n²) — it's O(a · b)
def print_pairs(arr_a, arr_b):
    for x in arr_a:     # a iterations
        for y in arr_b: # b iterations
            print(x, y)
```

### Rule 4: Sequential Steps Add

```python
def example(arr):
    # Step 1: O(n)
    for x in arr:
        print(x)

    # Step 2: O(n²)
    for i in arr:
        for j in arr:
            print(i, j)

# Total: O(n + n²) = O(n²)
```

### Rule 5: Nested Steps Multiply

```python
def example(arr):
    for i in arr:         # O(n)
        for j in arr:     #   × O(n)
            print(i, j)   # = O(n²)
```

### Rule 6: Logarithm Base Doesn't Matter

`log₂n` and `log₁₀n` differ only by a constant factor, so both are written `O(log n)`.

```
log₂n = log₁₀n / log₁₀2 = log₁₀n × 3.32
```

---

## 4. Data Structure Complexities

### Arrays

| Operation | Average | Worst |
|-----------|---------|-------|
| Access by index | O(1) | O(1) |
| Search (unsorted) | O(n) | O(n) |
| Search (sorted, binary) | O(log n) | O(log n) |
| Insert at end | O(1) amortized | O(n) |
| Insert at beginning/middle | O(n) | O(n) |
| Delete at end | O(1) | O(1) |
| Delete at beginning/middle | O(n) | O(n) |

### Hash Map / Hash Table / Dictionary

| Operation | Average | Worst |
|-----------|---------|-------|
| Get/Lookup | O(1) | O(n) |
| Insert | O(1) | O(n) |
| Delete | O(1) | O(n) |
| Search for value | O(n) | O(n) |

> Worst case O(n) occurs with many hash collisions (rare with good hash functions).

### Linked List

| Operation | Singly Linked | Doubly Linked |
|-----------|--------------|---------------|
| Access by index | O(n) | O(n) |
| Search | O(n) | O(n) |
| Insert at head | O(1) | O(1) |
| Insert at tail (with tail pointer) | O(1) | O(1) |
| Insert at arbitrary position | O(n) | O(n) |
| Delete at head | O(1) | O(1) |
| Delete at tail | O(n) | O(1) |
| Delete arbitrary (given node) | O(n) to find, O(1) to delete | O(1) to delete |

### Stack

| Operation | Time |
|-----------|------|
| Push | O(1) |
| Pop | O(1) |
| Peek/Top | O(1) |
| Search | O(n) |

### Queue

| Operation | Time |
|-----------|------|
| Enqueue | O(1) |
| Dequeue | O(1) |
| Peek/Front | O(1) |
| Search | O(n) |

### Binary Search Tree (BST)

| Operation | Average (balanced) | Worst (degenerate/linear) |
|-----------|-------------------|--------------------------|
| Access | O(log n) | O(n) |
| Search | O(log n) | O(n) |
| Insert | O(log n) | O(n) |
| Delete | O(log n) | O(n) |

### Balanced BSTs (AVL, Red-Black Tree)

| Operation | Time (guaranteed) |
|-----------|-------------------|
| Access | O(log n) |
| Search | O(log n) |
| Insert | O(log n) |
| Delete | O(log n) |

### Heap (Binary Heap)

| Operation | Time |
|-----------|------|
| Find min/max | O(1) |
| Insert | O(log n) |
| Delete min/max | O(log n) |
| Heapify (build from array) | O(n) |
| Search | O(n) |

### Trie (Prefix Tree)

| Operation | Time |
|-----------|------|
| Search | O(m) where m = key length |
| Insert | O(m) |
| Delete | O(m) |

> Tries beat hash maps when you need prefix operations or sorted iteration.

### Segment Tree

| Operation | Time |
|-----------|------|
| Build | O(n) |
| Range query | O(log n) |
| Point update | O(log n) |
| Range update | O(log n) |

### Disjoint Set (Union-Find)

| Operation | Time |
|-----------|------|
| Find | O(α(n)) ≈ O(1) amortized |
| Union | O(α(n)) ≈ O(1) amortized |

> α is the inverse Ackermann function — effectively constant for all practical n.

---

## 5. Sorting Algorithm Complexities

| Algorithm | Best | Average | Worst | Space | Stable? |
|-----------|------|---------|-------|-------|---------|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | ❌ |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | ✅ |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | ❌ |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | ❌ |
| Tim Sort | O(n) | O(n log n) | O(n log n) | O(n) | ✅ |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(k) | ✅ |
| Radix Sort | O(nk) | O(nk) | O(nk) | O(n+k) | ✅ |
| Bucket Sort | O(n+k) | O(n+k) | O(n²) | O(n+k) | ✅ |
| Shell Sort | O(n log n) | O(n log²n) | O(n²) | O(1) | ❌ |

> **k** = range of values, **Stable** = equal elements maintain original order.

**Key insights:**
- `O(n log n)` is the **theoretical lower bound** for comparison-based sorting.
- Counting/Radix sort beat this by not comparing elements (trade space for speed).
- QuickSort's O(n²) worst case is avoided in practice with random pivots or 3-way partition.
- TimSort (Python's and Java's built-in sort) is optimized for real-world data.

---

## 6. Graph & Tree Algorithm Complexities

> **V** = number of vertices (nodes), **E** = number of edges.

### Graph Traversal

| Algorithm | Time | Space | Use Case |
|-----------|------|-------|----------|
| BFS | O(V + E) | O(V) | Shortest path (unweighted), level-order |
| DFS | O(V + E) | O(V) | Cycle detection, topological sort, connectivity |

### Shortest Path

| Algorithm | Time | Space | Works With |
|-----------|------|-------|-----------|
| Dijkstra (binary heap) | O((V + E) log V) | O(V) | Non-negative weights |
| Dijkstra (Fibonacci heap) | O(E + V log V) | O(V) | Non-negative weights (theoretical) |
| Bellman-Ford | O(V · E) | O(V) | Negative weights, detects negative cycles |
| Floyd-Warshall | O(V³) | O(V²) | All-pairs shortest path |
| A* | O(E log V) | O(V) | Heuristic-guided, single target |
| SPFA | O(V · E) worst | O(V) | Avg faster than Bellman-Ford |

### Minimum Spanning Tree

| Algorithm | Time | Space |
|-----------|------|-------|
| Prim's (binary heap) | O((V + E) log V) | O(V) |
| Kruskal's | O(E log E) | O(V) |
| Borůvka's | O(E log V) | O(V) |

### Other Graph Algorithms

| Algorithm | Time | Space | Purpose |
|-----------|------|-------|---------|
| Topological Sort (Kahn's) | O(V + E) | O(V) | DAG ordering |
| Topological Sort (DFS) | O(V + E) | O(V) | DAG ordering |
| Tarjan's SCC | O(V + E) | O(V) | Strongly connected components |
| Kosaraju's SCC | O(V + E) | O(V) | Strongly connected components |
| Bipartite Check (BFS/DFS) | O(V + E) | O(V) | 2-colorability |
| Articulation Points | O(V + E) | O(V) | Bridge/cut vertex finding |
| Max Flow (Ford-Fulkerson) | O(E · max_flow) | O(V) | Network flow |
| Max Flow (Edmonds-Karp) | O(V · E²) | O(V) | Network flow (BFS-based) |

### Tree Algorithms

| Algorithm | Time | Space |
|-----------|------|-------|
| Tree traversal (pre/in/post/level) | O(n) | O(h) |
| Find height | O(n) | O(h) |
| LCA (naive) | O(n) per query | O(1) |
| LCA (binary lifting) | O(n log n) preprocessing, O(log n) query | O(n log n) |
| LCA (Euler tour + RMQ) | O(n) preprocessing, O(1) query | O(n) |

> **h** = tree height. For balanced trees h = O(log n); for skewed trees h = O(n).

---

## 7. Recognizing Complexity from Code

### Pattern Recognition Guide

```
Single loop over n        → O(n)
Two nested loops over n   → O(n²)
Three nested loops        → O(n³)
Halving the problem       → O(log n)
Loop + halving inside     → O(n log n)
Recursive calls × 2       → O(2ⁿ) unless memoized
All subsets               → O(2ⁿ)
All permutations          → O(n!)
```

### Common Code Patterns

```python
# O(log n) — input shrinks by fraction each iteration
while n > 1:
    n = n // 2

# O(log n) — same but recursive
def f(n):
    if n <= 1: return
    f(n // 2)

# O(n log n) — loop with halving inside
for i in range(n):
    j = n
    while j > 1:
        j //= 2

# O(n log n) — divide and conquer with O(n) merge
def merge_sort(arr): ...

# O(2ⁿ) — two recursive calls both on n-1
def fib(n):
    return fib(n-1) + fib(n-2)

# O(n) after memoization!
@lru_cache
def fib(n):
    return fib(n-1) + fib(n-2)

# O(2ⁿ) — generating all subsets
def subsets(arr, i=0):
    if i == len(arr): return [[]]
    rest = subsets(arr, i+1)
    return rest + [[arr[i]] + s for s in rest]

# O(n!) — generating permutations
from itertools import permutations
list(permutations(arr))  # n! items
```

### Tricky Cases

```python
# This looks O(n²) but is actually O(n)
# The inner loop doesn't reset to 0 each time
i, j = 0, 0
while i < n:
    while j < n and arr[j] < arr[i]:
        j += 1       # j only ever moves forward to n total
    i += 1
# Total moves of j: at most n → O(n) overall (two-pointer technique)

# This looks O(n log n) but depends on input
for i in range(n):
    if sorted_condition:
        binary_search(...)   # O(log n) per iteration → O(n log n)
    else:
        break                # May terminate early

# Careful with string concatenation in loops!
# This is O(n²) in languages where strings are immutable (Python, Java, JS)
result = ""
for x in arr:
    result += str(x)   # Creates a new string each time!
# Fix: use "".join(arr) → O(n)
```

### Recursion Tree Method

For recursive algorithms, draw the call tree:

```
fib(4)
├── fib(3)
│   ├── fib(2)
│   │   ├── fib(1) → 1
│   │   └── fib(0) → 0
│   └── fib(1) → 1
└── fib(2)
    ├── fib(1) → 1
    └── fib(0) → 0

Nodes at level k: 2^k
Total levels: n
Total nodes ≈ 2^n → O(2ⁿ)
```

---

## 8. Space Complexity

Space complexity measures **extra memory used** as a function of input size (not counting the input itself, unless specified).

### Space Complexity Examples

```python
# O(1) space — only a few variables
def sum_array(arr):
    total = 0          # one variable
    for x in arr:
        total += x
    return total

# O(n) space — stores output proportional to n
def double_array(arr):
    return [x * 2 for x in arr]   # New list of size n

# O(n) space — recursion stack depth is n
def factorial(n):
    if n <= 1: return 1
    return n * factorial(n - 1)   # n frames on call stack

# O(log n) space — binary search recursion depth
def binary_search(arr, target, l, r):
    mid = (l + r) // 2
    if arr[mid] == target: return mid
    elif arr[mid] < target: return binary_search(arr, target, mid+1, r)
    else: return binary_search(arr, target, l, mid-1)
    # Recursion depth = log n

# O(n) space — memo table
from functools import lru_cache
@lru_cache(maxsize=None)
def fib(n):
    if n <= 1: return n
    return fib(n-1) + fib(n-2)   # Stores n results in cache

# O(n²) space — 2D DP table
def edit_distance(s1, s2):
    dp = [[0] * (len(s2)+1) for _ in range(len(s1)+1)]  # n×m table
    ...
```

### Space-Time Tradeoffs

Many optimizations trade space for speed:

| Technique | Time Improvement | Space Cost |
|-----------|-----------------|------------|
| Memoization / DP | O(2ⁿ) → O(n) | O(n) or O(n²) |
| Hash map for lookups | O(n) → O(1) per lookup | O(n) |
| Prefix sum array | O(n) → O(1) per range query | O(n) |
| Precomputed lookup table | O(n) → O(1) | O(n) |
| BFS vs DFS | Same time | BFS uses O(w), DFS uses O(h) |

---

## 9. Recurrence Relations & Divide-and-Conquer

Recursive algorithms have **recurrence relations** that describe their runtime.

### Master Theorem

For recurrences of the form: `T(n) = a·T(n/b) + f(n)`

Where `a ≥ 1`, `b > 1`, and `f(n)` is the cost of work done outside recursive calls:

```
Let c = log_b(a)  (critical exponent)

Case 1: f(n) = O(n^(c-ε)) for some ε > 0
        → T(n) = Θ(n^c)         [Recursive work dominates]

Case 2: f(n) = Θ(n^c · log^k n) for k ≥ 0
        → T(n) = Θ(n^c · log^(k+1) n)  [Work evenly split]

Case 3: f(n) = Ω(n^(c+ε)) for some ε > 0, and regularity holds
        → T(n) = Θ(f(n))        [Combining work dominates]
```

### Master Theorem Examples

```
Merge Sort: T(n) = 2·T(n/2) + O(n)
  a=2, b=2, c = log₂(2) = 1, f(n) = O(n) = O(n¹)
  Case 2 (f(n) = Θ(n^c)) → T(n) = Θ(n log n)  ✅

Binary Search: T(n) = 1·T(n/2) + O(1)
  a=1, b=2, c = log₂(1) = 0, f(n) = O(1) = O(n⁰)
  Case 2 → T(n) = Θ(log n)  ✅

Strassen's Matrix Mult: T(n) = 7·T(n/2) + O(n²)
  a=7, b=2, c = log₂(7) ≈ 2.81, f(n) = O(n²)
  Since 2 < 2.81 → Case 1 → T(n) = Θ(n^log₂7) ≈ Θ(n^2.81)  ✅
  (Better than naive O(n³)!)

Naive Matrix Mult: T(n) = 8·T(n/2) + O(n²)
  a=8, b=2, c = log₂(8) = 3, f(n) = O(n²)
  Case 1 → T(n) = Θ(n³)  ✅
```

### Common Recurrences

| Recurrence | Solution | Algorithm |
|------------|----------|-----------|
| T(n) = T(n/2) + O(1) | O(log n) | Binary search |
| T(n) = T(n-1) + O(1) | O(n) | Linear recursion |
| T(n) = 2T(n/2) + O(n) | O(n log n) | Merge sort |
| T(n) = 2T(n/2) + O(1) | O(n) | Tree traversal |
| T(n) = T(n-1) + O(n) | O(n²) | Insertion sort |
| T(n) = 2T(n-1) + O(1) | O(2ⁿ) | Naive Fibonacci |
| T(n) = T(n/3) + T(2n/3) + O(n) | O(n log n) | Quicksort average |
| T(n) = 4T(n/2) + O(n²) | O(n² log n) | — |

### Recursion Tree Method (Manual)

For `T(n) = 2T(n/2) + n`:

```
Level 0:        n                   cost: n
Level 1:    n/2   n/2               cost: n
Level 2:  n/4 n/4 n/4 n/4          cost: n
...
Level k:  2^k nodes of size n/2^k  cost: n
...
Level log n: n leaves of size 1    cost: n

Total levels: log n
Cost per level: n
Total: n · log n = O(n log n)  ✅
```

---

## 10. Common Patterns & Techniques

### Two Pointers — O(n)

```python
# Find pair summing to target in sorted array
def two_sum_sorted(arr, target):
    left, right = 0, len(arr) - 1
    while left < right:
        s = arr[left] + arr[right]
        if s == target:   return (left, right)
        elif s < target:  left += 1
        else:             right -= 1
    return None
# O(n) instead of O(n²) brute force
```

### Sliding Window — O(n)

```python
# Max sum subarray of size k
def max_sliding_window(arr, k):
    window_sum = sum(arr[:k])
    max_sum = window_sum
    for i in range(k, len(arr)):
        window_sum += arr[i] - arr[i - k]   # Slide: add new, remove old
        max_sum = max(max_sum, window_sum)
    return max_sum
# O(n) instead of O(n·k) brute force
```

### Binary Search on Answer — O(log(max) · n)

```python
# "Find minimum X such that condition(X) is true"
def find_minimum(arr):
    lo, hi = min(arr), max(arr)
    while lo < hi:
        mid = (lo + hi) // 2
        if feasible(mid):   # O(n) check
            hi = mid
        else:
            lo = mid + 1
    return lo
# O(n log(max_val)) instead of O(n²) or worse
```

### Prefix Sums — O(1) range queries after O(n) build

```python
def build_prefix(arr):
    prefix = [0] * (len(arr) + 1)
    for i, x in enumerate(arr):
        prefix[i+1] = prefix[i] + x
    return prefix

def range_sum(prefix, l, r):   # inclusive [l, r]
    return prefix[r+1] - prefix[l]  # O(1)!
```

### Memoization / Top-Down DP — O(states)

```python
from functools import lru_cache

# Naive: O(2ⁿ) → Memoized: O(n)
@lru_cache(maxsize=None)
def fib(n):
    if n <= 1: return n
    return fib(n-1) + fib(n-2)
```

### Monotonic Stack — O(n)

```python
# Next greater element for each position
def next_greater(arr):
    result = [-1] * len(arr)
    stack = []   # Indices, monotonically decreasing values
    for i, x in enumerate(arr):
        while stack and arr[stack[-1]] < x:
            result[stack.pop()] = x
        stack.append(i)
    return result
# O(n) — each element pushed/popped at most once
```

### Fast Exponentiation — O(log n)

```python
# a^n mod m in O(log n) instead of O(n)
def power(a, n, m):
    result = 1
    a %= m
    while n > 0:
        if n % 2 == 1:
            result = result * a % m
        a = a * a % m
        n //= 2
    return result
```

---

## 11. Amortized Analysis

Amortized analysis gives the **average cost per operation** over a sequence of operations, even if some individual operations are expensive.

### Three Methods

**1. Aggregate Method** — Total cost / number of operations

```
Dynamic array doubling:
- n insertions total
- Doublings happen at size: 1, 2, 4, 8, ..., n
- Total copy work: 1 + 2 + 4 + ... + n = 2n (geometric series)
- n insertions, 2n total copy cost
- Amortized cost per insert: 2n/n = O(1)  ✅
```

**2. Accounting Method** — "Charge" extra credits on cheap ops, spend on expensive ops

```
Dynamic array: Charge 3 units per insert
- 1 unit: actual insertion
- 2 units: saved to pay for future copying
When we double: each element uses its 2 saved credits → fully paid!
Amortized cost: O(1) per insert  ✅
```

**3. Potential Method** — Define a "potential" function Φ representing stored energy

```
Φ = 2 × (elements after last doubling)
Amortized cost = actual cost + ΔΦ
Regular insert: 1 + 2 = 3 = O(1)
Doubling insert: n + (−n) + 2 = 2 = O(1)
```

### Common Amortized Results

| Structure/Operation | Worst Case | Amortized |
|--------------------|------------|-----------|
| Dynamic array append | O(n) | O(1) |
| Stack multi-pop | O(n) | O(1) per element |
| Queue from two stacks | O(n) | O(1) |
| Union-Find (with path compression + union by rank) | O(log n) | O(α(n)) ≈ O(1) |
| Splay tree operations | O(n) | O(log n) |
| Binary counter increment | O(n) | O(1) |

---

## 12. Big O vs Big Θ vs Big Ω

These are the three asymptotic notations, each describing a different kind of bound.

```
O  (Big O)       — UPPER bound    — "at most this fast"  (worst case)
Ω  (Big Omega)   — LOWER bound    — "at least this fast" (best case)
Θ  (Big Theta)   — TIGHT bound    — "exactly this fast"  (both)
```

### Formal Definitions

```
f(n) = O(g(n))    if f grows NO FASTER than g
                  ∃ c, n₀: f(n) ≤ c·g(n) for all n ≥ n₀

f(n) = Ω(g(n))    if f grows NO SLOWER than g
                  ∃ c, n₀: f(n) ≥ c·g(n) for all n ≥ n₀

f(n) = Θ(g(n))    if f(n) = O(g(n)) AND f(n) = Ω(g(n))
                  g is both an upper and lower bound
```

### Practical Examples

```
Binary Search:
  Best case (found at middle): O(1) = Ω(1)
  Worst case (not found):      O(log n)
  Overall: O(log n) [upper bound], Ω(1) [lower bound]
  NOT Θ(log n) because best case is O(1)

Merge Sort:
  Best case: Θ(n log n)  — always divides and merges
  Worst case: Θ(n log n) — same
  Overall: Θ(n log n)  — tight bound!

QuickSort:
  Best/Average case: Θ(n log n)
  Worst case (sorted input, bad pivot): Θ(n²)
  Overall: O(n²) [upper], Ω(n log n) [average lower]
```

### Little-o and Little-ω

```
f(n) = o(g(n))  — f grows STRICTLY SLOWER than g
                 lim[n→∞] f(n)/g(n) = 0

f(n) = ω(g(n))  — f grows STRICTLY FASTER than g
                 lim[n→∞] f(n)/g(n) = ∞

Example: n = o(n²)  because n/n² = 1/n → 0
         n² = ω(n)  because n²/n = n → ∞
```

### In Industry vs Academia

In **industry and interviews**, "Big O" is used loosely to mean "tight bound" (what academics call Big Theta). When someone says "merge sort is O(n log n)", they usually mean it's *tight* at Θ(n log n), not just an upper bound.

---

## 13. Interview Cheat Sheet & Quick Reference

### Complexity at a Glance

| Complexity | Name | Example |
|------------|------|---------|
| O(1) | Constant | Hash map lookup, array index |
| O(log n) | Logarithmic | Binary search, balanced BST |
| O(n) | Linear | Single loop, linear search |
| O(n log n) | Linearithmic | Merge sort, heap sort |
| O(n²) | Quadratic | Nested loops, bubble sort |
| O(n³) | Cubic | Triple nested loops |
| O(2ⁿ) | Exponential | Subsets, naive recursion |
| O(n!) | Factorial | Permutations, brute-force TSP |

---

### "What's the Best Possible?" Reference

| Problem | Lower Bound | Why |
|---------|-------------|-----|
| Comparison-based sorting | Ω(n log n) | Decision tree has n! leaves |
| Searching unsorted array | Ω(n) | Must check every element |
| Searching sorted array | Ω(log n) | Binary search optimal |
| Matrix multiplication | Ω(n²) | Must read n² entries |
| Finding max of n elements | Ω(n) | Must inspect each |

---

### Complexity Optimization Strategies

| If you have | Try | Expected result |
|-------------|-----|-----------------|
| O(n²) nested loops | Two pointers, sliding window | O(n) |
| O(n²) repeated lookups | Hash map | O(n) |
| O(n²) sorted pairs | Sort + binary search | O(n log n) |
| O(2ⁿ) recursion | Memoization / DP | O(n) or O(n²) |
| O(n) per range query | Prefix sums | O(1) per query |
| O(n log n) repeated sorts | Heap | O(n log k) |
| O(n) connectivity queries | Union-Find | O(α(n)) ≈ O(1) |

---

### Interview Problem Patterns

```
Array/String problems:    → Two pointers, sliding window, hash map
Sorted array:             → Binary search
Tree problems:            → DFS (recursion), BFS (queue)
Graph problems:           → BFS (shortest path), DFS (connectivity, cycles)
Optimization:             → Dynamic programming, greedy
Counting subsets:         → Bitmask, backtracking
Scheduling / intervals:   → Sort by start/end, greedy
Top K elements:           → Heap (priority queue)
Range queries:            → Prefix sums, segment tree
String matching:          → KMP O(n+m), Rabin-Karp O(n+m) avg
```

---

### Quick Sanity Check for Big O

Ask yourself:
1. **Does the loop variable double/halve?** → log n
2. **Simple loop over n?** → n
3. **Two nested independent loops?** → n²
4. **Does the function call itself twice on n-1?** → 2ⁿ
5. **Does the function call itself twice on n/2?** → n (geometric series)
6. **Divide + linear work to combine?** → n log n

---

### Time Budget for Competitive Programming

| n | Max Complexity |
|---|---------------|
| n ≤ 10 | O(n!) or O(n^6) |
| n ≤ 20 | O(2ⁿ) |
| n ≤ 100 | O(n³) |
| n ≤ 1,000 | O(n²) |
| n ≤ 100,000 | O(n log n) |
| n ≤ 1,000,000 | O(n) or O(n log n) |
| n ≤ 10⁹ | O(log n) or O(√n) |
| n ≤ 10¹⁸ | O(log n) |

> Assume ~10⁸–10⁹ simple operations per second on modern hardware.

---

### Pitfalls & Common Mistakes

```
❌ Forgetting that string concatenation in a loop is O(n²)
✅ Use join() or StringBuilder

❌ Treating hash map as always O(1) — worst case is O(n)
✅ Acknowledge amortized O(1) in analysis

❌ Assuming sorted input for quicksort performance claims
✅ Use random pivot or state O(n log n) average

❌ Counting the wrong variable as n
✅ Clearly define what n represents (nodes, characters, edges...)

❌ Ignoring space complexity
✅ Always analyze both time AND space

❌ O(n²) in disguise — inner loop with variable bound
✅ Count total iterations across all loop executions

❌ Thinking memoized recursion is O(n) if the function has 2 parameters
✅ Memoized complexity = number of unique subproblems × work per subproblem
```

---

*Master Big O by analyzing code you write every day. Ask: "If I 10× the input, what happens?" For deeper study, read: CLRS (Introduction to Algorithms), Algorithm Design by Kleinberg & Tardos, or Skiena's Algorithm Design Manual.*
