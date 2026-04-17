# Recursion: A Complete Progressive Tutorial

---

## 1. What & Why

Recursion is a technique where a function solves a problem by calling itself with a smaller version of that same problem. The function keeps calling itself until it reaches a case small enough to solve directly — the base case — then unwinds, combining results as it returns.

Why does this exist? Because many problems are naturally self-similar: a tree is a node with smaller trees attached. A sorted list is a value plus a smaller sorted list. A directory contains files and smaller directories. Expressing solutions to self-similar problems recursively often produces code that directly mirrors the problem's structure — making it shorter, clearer, and easier to verify correct.

The tradeoff: recursion uses the call stack for memory, and each function call has overhead. Problems with millions of recursive calls need iterative reformulations or stack-explicit implementations. But for tree traversal, divide-and-conquer algorithms, backtracking, and dynamic programming, recursion is the most natural expression of the solution.

---

## 2. Mental Model

Think of recursion as delegation: "I'll handle this one step, then hand the rest to a smaller version of myself."

```
Problem: calculate factorial(4)

factorial(4) says: "I'll multiply 4 by whatever factorial(3) returns."
factorial(3) says: "I'll multiply 3 by whatever factorial(2) returns."
factorial(2) says: "I'll multiply 2 by whatever factorial(1) returns."
factorial(1) says: "That's the base case. I return 1."

Now the answers unwind back up:
factorial(2) = 2 × 1 = 2
factorial(3) = 3 × 2 = 6
factorial(4) = 4 × 6 = 24
```

The call stack holds each "pending" computation. Each frame stores: the local variables of that call, and where to return when done. Stack depth equals recursion depth. This is why deep recursion risks stack overflow: you run out of stack memory before reaching the base case.

```
Call stack (growing downward):
[ factorial(4) ] <- waiting for factorial(3) to return
[ factorial(3) ] <- waiting for factorial(2) to return
[ factorial(2) ] <- waiting for factorial(1) to return
[ factorial(1) ] <- returns 1 (base case)
```

---

## 3. Progressive Examples

### Level 1: The Classic Starting Points

```python
# Pattern: base case first, recursive case second.
# Every recursive call must move toward the base case.

def factorial(n):
    """n! = n × (n-1) × (n-2) × ... × 1"""
    if n <= 1:          # base case: factorial(0) = factorial(1) = 1
        return 1
    return n * factorial(n - 1)   # recursive case: n! = n × (n-1)!

print(factorial(5))   # 120
print(factorial(0))   # 1

# Fibonacci: F(n) = F(n-1) + F(n-2)
def fib_naive(n):
    """Correct but SLOW — exponential time O(2^n). Shown to illustrate the pattern."""
    if n <= 1:          # base cases: F(0)=0, F(1)=1
        return n
    return fib_naive(n - 1) + fib_naive(n - 2)

# Fix with memoization (cache results to avoid recomputation)
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
    """O(n) time with memoization — each subproblem computed once."""
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)

print(fib(50))    # 12586269025 — fast with memoization

# Sum of a list — recursive decomposition
def sum_list(arr):
    """A list's sum = first element + sum of the rest."""
    if not arr:             # base case: empty list sums to 0
        return 0
    return arr[0] + sum_list(arr[1:])   # recursive case

print(sum_list([1, 2, 3, 4, 5]))   # 15

# Power function: x^n = x × x^(n-1)
def power(x, n):
    """Fast exponentiation: O(log n) instead of O(n)."""
    if n == 0:
        return 1
    if n % 2 == 0:
        half = power(x, n // 2)
        return half * half      # x^n = (x^(n/2))^2 — avoids two recursive calls
    return x * power(x, n - 1)

print(power(2, 10))   # 1024
```

### Level 2: Divide and Conquer

```python
# Divide and conquer: split the problem, solve each part, combine results.
# Template: divide → conquer → combine

def merge_sort(arr):
    """
    Split array in half, sort each half recursively, merge.
    O(n log n) time, O(n) space.
    """
    if len(arr) <= 1:           # base case: single element is sorted
        return arr

    mid = len(arr) // 2
    left = merge_sort(arr[:mid])    # conquer left half
    right = merge_sort(arr[mid:])   # conquer right half
    return merge(left, right)       # combine

def merge(left, right):
    """Merge two sorted arrays into one sorted array. O(n)."""
    result = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i]); i += 1
        else:
            result.append(right[j]); j += 1
    result.extend(left[i:])
    result.extend(right[j:])
    return result

print(merge_sort([38, 27, 43, 3, 9, 82, 10]))
# [3, 9, 10, 27, 38, 43, 82]

def binary_search(arr, target, lo=0, hi=None):
    """
    Recursive binary search.
    Divide: eliminate half the array each call.
    O(log n) time, O(log n) space (call stack depth).
    """
    if hi is None:
        hi = len(arr) - 1

    if lo > hi:         # base case: search space is empty
        return -1

    mid = (lo + hi) // 2

    if arr[mid] == target:
        return mid
    elif arr[mid] < target:
        return binary_search(arr, target, mid + 1, hi)   # right half
    else:
        return binary_search(arr, target, lo, mid - 1)   # left half

sorted_arr = [3, 9, 10, 27, 38, 43, 82]
print(binary_search(sorted_arr, 27))   # 3
print(binary_search(sorted_arr, 5))    # -1

def count_inversions(arr):
    """
    Count pairs (i, j) where i < j but arr[i] > arr[j].
    Solved in O(n log n) using a modified merge sort.
    """
    if len(arr) <= 1:
        return arr, 0

    mid = len(arr) // 2
    left, left_inv = count_inversions(arr[:mid])
    right, right_inv = count_inversions(arr[mid:])

    merged = []
    inv_count = left_inv + right_inv
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            merged.append(left[i]); i += 1
        else:
            merged.append(right[j]); j += 1
            inv_count += len(left) - i   # all remaining left elements are > right[j]
    merged.extend(left[i:])
    merged.extend(right[j:])
    return merged, inv_count

_, inversions = count_inversions([6, 3, 5, 2, 4, 1])
print(f"Inversions: {inversions}")   # 10
```

### Level 3: Tree Recursion

```python
# Trees are the natural domain of recursion: a tree is a node + subtrees.
# Every tree algorithm follows the pattern: handle node, recurse on children.

class TreeNode:
    def __init__(self, val, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

# Build a sample tree:
#        10
#       /  \
#      5    15
#     / \     \
#    3   7    20
root = TreeNode(10,
    TreeNode(5, TreeNode(3), TreeNode(7)),
    TreeNode(15, None, TreeNode(20)))

def tree_height(node):
    """Height of tree = 1 + max(height(left), height(right))."""
    if node is None:
        return 0    # base case: empty tree has height 0
    return 1 + max(tree_height(node.left), tree_height(node.right))

print(tree_height(root))   # 3

def tree_sum(node):
    """Sum all values in the tree."""
    if node is None:
        return 0
    return node.val + tree_sum(node.left) + tree_sum(node.right)

print(tree_sum(root))   # 60

def inorder(node):
    """Left → Node → Right: gives sorted output for a BST."""
    if node is None:
        return []
    return inorder(node.left) + [node.val] + inorder(node.right)

print(inorder(root))   # [3, 5, 7, 10, 15, 20]

def is_balanced(node):
    """
    A tree is balanced if for every node, the height difference
    between left and right subtrees is at most 1.
    Returns (is_balanced, height).
    """
    if node is None:
        return True, 0

    left_balanced, left_h = is_balanced(node.left)
    right_balanced, right_h = is_balanced(node.right)

    balanced = (left_balanced and right_balanced
                and abs(left_h - right_h) <= 1)
    return balanced, 1 + max(left_h, right_h)

balanced, height = is_balanced(root)
print(f"Balanced: {balanced}, Height: {height}")   # True, 3

def path_sum(node, target, current=0):
    """Check if any root-to-leaf path has a sum equal to target."""
    if node is None:
        return False
    current += node.val
    if node.left is None and node.right is None:    # leaf node
        return current == target
    return path_sum(node.left, target, current) or path_sum(node.right, target, current)

print(path_sum(root, 18))   # True (10 → 5 → 3 is 18)
print(path_sum(root, 100))  # False
```

### Level 4: Backtracking

```python
# Backtracking: build a solution incrementally, abandon it (backtrack)
# as soon as you determine it cannot lead to a valid complete solution.
# Template: choose → explore → unchoose

def generate_permutations(nums):
    """
    Generate all permutations of nums.
    At each step: choose an unused number, recurse, unchoose.
    O(n! × n) time — unavoidable since there are n! permutations.
    """
    result = []

    def backtrack(current, remaining):
        if not remaining:           # base case: used all numbers
            result.append(current[:])   # copy current — don't store reference
            return

        for i in range(len(remaining)):
            current.append(remaining[i])               # choose
            backtrack(current, remaining[:i] + remaining[i+1:])  # explore
            current.pop()                              # unchoose (backtrack)

    backtrack([], nums)
    return result

print(generate_permutations([1, 2, 3]))
# [[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1]]

def n_queens(n):
    """
    Place n queens on an n×n chessboard so none attack each other.
    Classic backtracking: place queen in each row, check validity.
    """
    solutions = []
    board = [-1] * n   # board[row] = column of queen in that row

    def is_safe(row, col):
        for r in range(row):
            c = board[r]
            if c == col:            # same column
                return False
            if abs(r - row) == abs(c - col):    # same diagonal
                return False
        return True

    def solve(row):
        if row == n:               # placed all n queens
            solutions.append(board[:])
            return
        for col in range(n):
            if is_safe(row, col):
                board[row] = col   # place queen
                solve(row + 1)     # recurse to next row
                board[row] = -1    # remove queen (backtrack)

    solve(0)
    return solutions

solutions = n_queens(4)
print(f"4-Queens solutions: {len(solutions)}")   # 2
for sol in solutions:
    for row in sol:
        print("." * row + "Q" + "." * (4 - row - 1))
    print()

def subset_sum(nums, target):
    """Find if any subset of nums sums to target."""
    def backtrack(index, remaining):
        if remaining == 0:
            return True     # found a valid subset
        if index >= len(nums) or remaining < 0:
            return False    # pruned: overshot or exhausted elements

        # Choice 1: include nums[index]
        if backtrack(index + 1, remaining - nums[index]):
            return True
        # Choice 2: exclude nums[index]
        return backtrack(index + 1, remaining)

    return backtrack(0, target)

print(subset_sum([3, 1, 4, 2, 6], 7))   # True (3+4 or 1+6)
print(subset_sum([3, 1, 4, 2, 6], 8))   # True (2+6 or 3+1+4)
print(subset_sum([3, 1, 4, 2, 6], 99))  # False
```

### Level 5: Tail Recursion and Iteration Conversion

```python
# Tail recursion: the recursive call is the LAST operation in the function.
# No pending computation after the recursive call returns.
# Many languages optimize tail calls to avoid growing the stack.
# Python does NOT optimize tail calls — convert to iteration for large inputs.

# NOT tail-recursive (multiplication happens after recursive call returns):
def factorial_head(n):
    if n <= 1: return 1
    return n * factorial_head(n - 1)   # n * ... is pending

# Tail-recursive (accumulator carries the state):
def factorial_tail(n, acc=1):
    if n <= 1: return acc
    return factorial_tail(n - 1, n * acc)   # all computation in the args

# Convert to iterative (essential in Python for large n):
def factorial_iter(n):
    acc = 1
    while n > 1:
        acc *= n
        n -= 1
    return acc

# Python default recursion limit is 1000 frames
import sys
# sys.setrecursionlimit(10000)  # increase if needed, but prefer iteration

# Converting recursive DFS to iterative using explicit stack
def dfs_iterative(graph, start):
    """Iterative DFS — avoids Python's recursion limit for deep graphs."""
    visited = set()
    stack = [start]     # explicit stack replaces the call stack
    result = []

    while stack:
        node = stack.pop()    # pop from the RIGHT (LIFO — like recursion)
        if node in visited:
            continue
        visited.add(node)
        result.append(node)
        # Push neighbors in reverse to maintain left-to-right order
        for neighbor in reversed(graph.get(node, [])):
            if neighbor not in visited:
                stack.append(neighbor)

    return result

# Memoization as an alternative to deep recursion — O(n) space, O(n) time
def fib_memo(n, memo={}):
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fib_memo(n - 1, memo) + fib_memo(n - 2, memo)
    return memo[n]

# Bottom-up dynamic programming — best for Python (no stack depth risk)
def fib_dp(n):
    if n <= 1:
        return n
    prev, curr = 0, 1
    for _ in range(2, n + 1):
        prev, curr = curr, prev + curr
    return curr

print(fib_dp(1000))  # works fine — no recursion at all
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Missing or unreachable base case**

```python
# WRONG: infinite recursion — n never reaches the base case
def countdown(n):
    print(n)
    return countdown(n - 1)   # no base case!

# ALSO WRONG: base case unreachable with float inputs
def bad_factorial(n):
    if n == 0:      # for n=0.5: never equals 0, recurses forever
        return 1
    return n * bad_factorial(n - 1)

# CORRECT: guard against unexpected inputs
def factorial(n):
    if not isinstance(n, int) or n < 0:
        raise ValueError(f"factorial requires non-negative integer, got {n}")
    if n <= 1:
        return 1
    return n * factorial(n - 1)
```

**Mistake 2: Exponential redundant computation in tree recursion**

```python
# WRONG: fib(50) makes ~2^50 calls — takes many seconds
def fib_slow(n):
    if n <= 1: return n
    return fib_slow(n-1) + fib_slow(n-2)

# The problem: fib_slow(48) is called from BOTH fib_slow(50) AND fib_slow(49)
# and from each of their callers, exponentially.

# CORRECT: memoize or use bottom-up DP
from functools import lru_cache

@lru_cache(maxsize=None)
def fib_fast(n):
    if n <= 1: return n
    return fib_fast(n-1) + fib_fast(n-2)
# Now fib_fast(48) is computed once and cached. O(n) calls total.
```

**Mistake 3: Mutating shared state in backtracking without undoing**

```python
# WRONG: appending to result without removing — ruins future branches
def permutations_broken(nums, current, result):
    if not nums:
        result.append(current)   # appends reference, not copy
        return
    for i in range(len(nums)):
        current.append(nums[i])
        permutations_broken(nums[:i] + nums[i+1:], current, result)
        # FORGOT: current.pop()   <- backtrack missing!

# Also wrong: appending without copying
result = []
permutations_broken([1,2,3], [], result)
# All entries in result point to the same list (which is now empty)

# CORRECT: copy when adding to result, AND undo state changes
def permutations_correct(nums, current, result):
    if not nums:
        result.append(current[:])   # copy the current state
        return
    for i in range(len(nums)):
        current.append(nums[i])
        permutations_correct(nums[:i] + nums[i+1:], current, result)
        current.pop()              # undo — backtrack
```

**Mistake 4: Stack overflow on large inputs in Python**

```python
# Python's default recursion limit is 1000 frames.
# This fails for n > ~500 depending on system:
def sum_to_n(n):
    if n <= 0: return 0
    return n + sum_to_n(n - 1)

sum_to_n(10000)   # RecursionError: maximum recursion depth exceeded

# Fix 1: convert to iteration
def sum_to_n_iter(n):
    return n * (n + 1) // 2   # closed-form is even better

# Fix 2: use sys.setrecursionlimit (increases limit, doesn't eliminate risk)
import sys
sys.setrecursionlimit(50000)   # still uses stack memory

# Fix 3: tail-call simulation with a trampoline
def trampoline(f, *args):
    result = f(*args)
    while callable(result):    # if result is a function, call it
        result = result()
    return result
```

**Mistake 5: Confusing recursion depth with time complexity**

```python
# merge_sort has O(log n) recursion depth but O(n log n) time
# The DEPTH of recursion tells you SPACE usage (call stack), not time.

# Binary search: O(log n) depth, O(log n) time
# Merge sort: O(log n) depth, O(n log n) time
# Fibonacci naive: O(n) depth, O(2^n) time — the depth is misleading here

# Always analyze separately:
# Time complexity: how many total function calls, and work per call?
# Space complexity: what is the maximum recursion depth simultaneously active?
```

---

## 5. The "Why Does This Work" Layer

### How the Call Stack Executes Recursion

When a function calls itself, the CPU doesn't jump to the beginning and discard the current execution context. It pushes a new stack frame — a block of memory containing local variables, parameters, and the return address (where to continue when this call returns).

```
Stack grows downward with each call:
[factorial(4)  | n=4, return_addr=main+12  ]  <- pushed first
[factorial(3)  | n=3, return_addr=fact+8   ]
[factorial(2)  | n=2, return_addr=fact+8   ]
[factorial(1)  | n=1, return_addr=fact+8   ]  <- base case, starts returning
```

Each frame is typically 100-1000 bytes. With 1000 frames (Python's limit), that's 100KB-1MB of stack. Stack overflow occurs when the stack exceeds the OS-allocated size, which varies by system (often 1-8 MB per thread). This is why tail recursion optimization matters in functional languages: if the last thing a function does is call itself, the compiler can reuse the same stack frame.

### Why Memoization Converts Exponential to Linear

The naive Fibonacci recurrence tree looks like this:

```
fib(5)
├── fib(4)
│   ├── fib(3)          <- computed here
│   │   ├── fib(2)
│   │   └── fib(1)
│   └── fib(2)
└── fib(3)              <- computed AGAIN without memoization
    ├── fib(2)
    └── fib(1)
```

Without memoization, fib(3) is computed twice, fib(2) three times, fib(1) five times. The number of calls is Fibonacci-in-itself — O(φⁿ) where φ ≈ 1.618.

With memoization, the first time fib(k) is computed, its result is stored. Every subsequent call to fib(k) returns the stored result in O(1). Each of the n unique subproblems is computed exactly once: O(n) total calls.

### Why Backtracking Prunes the Search Space

Without backtracking, generating all permutations of n elements by choosing positions one at a time would explore nⁿ possibilities (since each of n positions could hold any of n values). With backtracking:

At depth 0: n choices for position 0
At depth 1: n-1 remaining choices (used one value)
At depth 2: n-2 remaining choices
...

Total: n × (n-1) × (n-2) × ... × 1 = n! leaves. No redundant paths are explored.

For constraint-satisfaction problems (N-Queens, Sudoku), early constraint checking prunes far more than this. An invalid placement at depth 3 of N-Queens eliminates an entire subtree without exploring it — this is why backtracking can solve N-Queens for n=50 in milliseconds despite there being ~n! potential board configurations.

---

## 6. Quick Reference

### Recursive Problem Taxonomy

| Problem Type | Pattern | Example |
|-------------|---------|---------|
| Linear recursion | One recursive call | factorial, linear search |
| Binary recursion | Two recursive calls | Fibonacci, merge sort |
| Tree recursion | Recursion on tree structure | tree height, path sum |
| Divide and conquer | Split → solve → combine | merge sort, quicksort |
| Backtracking | Choose → explore → unchoose | N-Queens, permutations |
| Memoized recursion | Cache subproblem results | Fibonacci, LCS |

### Complexity Analysis Template

```
T(n) = a × T(n/b) + O(n^d)

Master Theorem:
  log_b(a) > d → O(n^(log_b a))  (subproblems dominate)
  log_b(a) = d → O(n^d × log n)  (equal work at each level)
  log_b(a) < d → O(n^d)          (non-recursive work dominates)

Examples:
  Merge sort:     a=2, b=2, d=1 → log₂2=1=d → O(n log n)
  Binary search:  a=1, b=2, d=0 → log₂1=0=d → O(log n)
  Fibonacci:      a=2, b=1 (not divide by b) → O(2^n)
```

### Anatomy of a Correct Recursive Function

```python
def solve(problem):
    # 1. VALIDATE: guard against invalid input
    if not valid(problem):
        raise ValueError(...)

    # 2. BASE CASE: smallest solvable problem
    if is_trivial(problem):
        return base_answer

    # 3. DECOMPOSE: make the problem smaller
    subproblem = reduce(problem)

    # 4. RECURSE: solve the smaller problem
    sub_result = solve(subproblem)

    # 5. COMBINE: build the answer
    return combine(sub_result, problem)
```

### When to Choose Recursion vs Iteration

Use recursion when:
- The problem structure is naturally hierarchical (trees, graphs, nested data)
- Divide-and-conquer decomposition is cleaner recursive
- Writing backtracking or exhaustive search
- The recursion depth is bounded by `log n` or a small constant

Use iteration when:
- Linear traversal (lists, strings) — a loop is clearer
- Python + large n — avoid stack overflow
- Performance is critical — iteration has less overhead
- The recursive version requires careful tail-call conversion anyway
