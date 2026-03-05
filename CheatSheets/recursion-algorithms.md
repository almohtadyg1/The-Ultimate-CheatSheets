# Recursion Algorithms Cheat Sheet
### From Zero to Expert

---

## What is Recursion?

**Recursion** is a technique where a function solves a problem by calling itself on a smaller version of the same problem. Every recursive solution consists of two parts:

- **Base case** — the condition where the function stops calling itself and returns a direct answer
- **Recursive case** — the part where the function reduces the problem and calls itself

```
factorial(4)
  └── 4 × factorial(3)
          └── 3 × factorial(2)
                  └── 2 × factorial(1)
                              └── 1   ← base case, unwind begins
                  └── 2 × 1 = 2
          └── 3 × 2 = 6
  └── 4 × 6 = 24
```

**The golden rule:** every recursive call must move toward the base case. If it doesn't, you get infinite recursion and a stack overflow.

---

## The Call Stack

Each function call is pushed onto the **call stack** as a **stack frame** containing local variables, parameters, and the return address. Recursion builds a chain of frames that unwind once the base case is hit.

```
factorial(4)  ← frame 4  (waiting)
factorial(3)  ← frame 3  (waiting)
factorial(2)  ← frame 2  (waiting)
factorial(1)  ← frame 1  returns 1  ← top of stack
```

**Stack depth = O(recursion depth).** Deep recursion risks a **stack overflow**. Python's default limit is 1,000 frames; this is why iterative or tail-recursive solutions are preferred for large inputs.

---

## Anatomy of a Recursive Function

```python
def solve(problem):
    # 1. Base case — stop condition
    if problem is trivial:
        return direct_answer

    # 2. Reduce — shrink the problem
    smaller = reduce(problem)

    # 3. Recurse — solve the smaller version
    sub_result = solve(smaller)

    # 4. Combine — build the answer from sub-results
    return combine(sub_result, problem)
```

---

## Complexity Analysis: The Recurrence

The time complexity of a recursive algorithm is described by a **recurrence relation**.

```
T(n) = a · T(n/b) + f(n)

  a = number of recursive subproblems
  b = factor by which input shrinks
  f(n) = work done outside recursive calls
```

### Master Theorem

Solves divide-and-conquer recurrences of the form T(n) = aT(n/b) + f(n):

| Condition | Result | Example |
|---|---|---|
| f(n) = O(n^(log_b(a) − ε)) | T(n) = Θ(n^log_b(a)) | Binary search: T(n)=T(n/2)+O(1) → O(log n) |
| f(n) = Θ(n^log_b(a)) | T(n) = Θ(n^log_b(a) · log n) | Merge sort: T(n)=2T(n/2)+O(n) → O(n log n) |
| f(n) = Ω(n^(log_b(a) + ε)) | T(n) = Θ(f(n)) | T(n)=T(n/2)+O(n) → O(n) |

### Common Recurrences

| Recurrence | Algorithm | Complexity |
|---|---|---|
| T(n) = T(n−1) + O(1) | Factorial | O(n) |
| T(n) = T(n−1) + O(n) | Selection sort | O(n²) |
| T(n) = 2T(n−1) + O(1) | Tower of Hanoi | O(2ⁿ) |
| T(n) = T(n/2) + O(1) | Binary search | O(log n) |
| T(n) = 2T(n/2) + O(n) | Merge sort | O(n log n) |
| T(n) = T(n−1) + T(n−2) | Naïve Fibonacci | O(φⁿ) ≈ O(1.618ⁿ) |
| T(n) = 2T(n/2) + O(1) | Tree traversal | O(n) |

---

## Linear Recursion

A single recursive call per invocation. The simplest and most common form.

### Factorial

```python
def factorial(n):
    if n == 0:              # base case
        return 1
    return n * factorial(n - 1)   # recursive case
```

**T(n) = T(n−1) + O(1) → O(n) time, O(n) space**

### Sum of Array

```python
def array_sum(arr, i=0):
    if i == len(arr):
        return 0
    return arr[i] + array_sum(arr, i + 1)
```

### Power Function

```python
def power(base, exp):
    if exp == 0:
        return 1
    return base * power(base, exp - 1)   # O(n)
```

Better with fast exponentiation (see below): **O(log n)**.

---

## Tail Recursion

A recursive call where the recursive invocation is the **very last operation** — nothing is waiting on the result to do further work. The compiler (or interpreter) can optimize this into a loop, using **O(1) stack space**.

```python
# NOT tail recursive — multiplication waits on the result
def factorial(n):
    if n == 0: return 1
    return n * factorial(n - 1)   # ← 'n *' is pending after the call

# Tail recursive — accumulator carries the result forward
def factorial_tail(n, acc=1):
    if n == 0: return acc
    return factorial_tail(n - 1, n * acc)   # ← nothing pending, just return
```

**Languages with TCO (Tail Call Optimization):** Haskell, Scala, Erlang, Scheme, Swift, Kotlin (with `tailrec` keyword).  
**Python does NOT optimize tail calls** — use explicit iteration for deep recursion.

### Converting Tail Recursion to Iteration

Any tail-recursive function translates mechanically into a loop:

```python
def factorial_iter(n):
    acc = 1
    while n > 0:
        acc *= n
        n -= 1
    return acc
```

---

## Binary Recursion

Two recursive calls per invocation. Often arises in divide-and-conquer and tree-based problems.

### Fibonacci (Naïve)

```python
def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)
```

**O(2ⁿ) time** — catastrophic. Each call branches into two, creating an exponential tree of redundant work. Fixed with memoization (see Dynamic Programming section).

### Binary Search (Recursive)

```python
def binary_search(arr, target, lo, hi):
    if lo > hi:
        return -1
    mid = lo + (hi - lo) // 2
    if arr[mid] == target:
        return mid
    elif arr[mid] < target:
        return binary_search(arr, target, mid + 1, hi)
    else:
        return binary_search(arr, target, lo, mid - 1)
```

Only **one** branch is actually taken per call — making this effectively linear recursion. **O(log n) time, O(log n) space** (stack depth).

---

## Divide and Conquer

Split the problem into **independent subproblems**, solve each recursively, then **combine** the results. The three-step pattern: **Divide → Conquer → Combine**.

### Merge Sort

```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr

    mid = len(arr) // 2
    left  = merge_sort(arr[:mid])     # conquer left half
    right = merge_sort(arr[mid:])     # conquer right half
    return merge(left, right)         # combine

def merge(left, right):
    result, i, j = [], 0, 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i]); i += 1
        else:
            result.append(right[j]); j += 1
    return result + left[i:] + right[j:]
```

**T(n) = 2T(n/2) + O(n) → O(n log n)**

### Quicksort

```python
def quicksort(arr):
    if len(arr) <= 1:
        return arr
    pivot = arr[len(arr) // 2]
    left   = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right  = [x for x in arr if x > pivot]
    return quicksort(left) + middle + quicksort(right)
```

**Average: O(n log n). Worst case (sorted input, bad pivot): O(n²)**

### Fast Power (Exponentiation by Squaring)

```python
def fast_power(base, exp):
    if exp == 0:
        return 1
    if exp % 2 == 0:
        half = fast_power(base, exp // 2)
        return half * half               # square the result — don't call twice!
    return base * fast_power(base, exp - 1)
```

**T(n) = T(n/2) + O(1) → O(log n)**. The key insight: `base^8 = (base^4)^2` — compute `base^4` once, square it.

### Karatsuba Multiplication

Multiplies two n-digit numbers faster than the schoolbook O(n²) algorithm.

```
Schoolbook: a×b requires n² single-digit multiplications
Karatsuba:  split a = a1·10^m + a0,  b = b1·10^m + b0

Compute only 3 multiplications instead of 4:
  z0 = a0 × b0
  z2 = a1 × b1
  z1 = (a0+a1) × (b0+b1) − z0 − z2

Result: z2·10^(2m) + z1·10^m + z0
```

**T(n) = 3T(n/2) + O(n) → O(n^log₂3) ≈ O(n^1.585)**

### Closest Pair of Points

```
Divide:   Split n points by x-coordinate into left and right halves
Conquer:  Recursively find closest pair in each half → distance d
Combine:  Check the strip of width 2d around the dividing line
          At most 8 points need checking per strip point (geometry proof)
          → O(n) combine step

T(n) = 2T(n/2) + O(n log n) → O(n log² n)
With careful sorting: O(n log n)
```

### Strassen's Matrix Multiplication

Multiplies n×n matrices using **7 recursive multiplications** instead of 8.

```
Standard: C = A×B requires 8 n/2×n/2 multiplications → O(n³)
Strassen: 7 multiplications with more additions

T(n) = 7T(n/2) + O(n²) → O(n^log₂7) ≈ O(n^2.807)
```

---

## Tree Recursion

Problems that naturally decompose into **tree-shaped** recursive calls. Any tree traversal is inherently recursive.

### Tree Traversals

```python
class Node:
    def __init__(self, val, left=None, right=None):
        self.val, self.left, self.right = val, left, right

def inorder(node):                    # Left → Root → Right (sorted for BST)
    if node is None: return []
    return inorder(node.left) + [node.val] + inorder(node.right)

def preorder(node):                   # Root → Left → Right (copy/serialize tree)
    if node is None: return []
    return [node.val] + preorder(node.left) + preorder(node.right)

def postorder(node):                  # Left → Right → Root (delete tree, evaluate)
    if node is None: return []
    return postorder(node.left) + postorder(node.right) + [node.val]
```

### Tree Height

```python
def height(node):
    if node is None:
        return 0
    return 1 + max(height(node.left), height(node.right))
```

### Diameter of a Tree

```python
def diameter(node):
    def depth(node):
        if node is None: return 0, 0    # (max_diameter_in_subtree, depth)
        ld, lh = depth(node.left)
        rd, rh = depth(node.right)
        return max(ld, rd, lh + rh), 1 + max(lh, rh)
    return depth(node)[0]
```

### Lowest Common Ancestor (LCA)

```python
def lca(root, p, q):
    if root is None or root == p or root == q:
        return root
    left  = lca(root.left, p, q)
    right = lca(root.right, p, q)
    if left and right:
        return root          # p and q are in different subtrees
    return left or right     # both in same subtree
```

---

## Backtracking

A systematic technique that builds a solution **incrementally** and **abandons** ("prunes") partial solutions as soon as it determines they cannot lead to a valid complete solution. It explores a virtual decision tree, backtracking when a dead end is reached.

```
Pattern:
  def backtrack(state, choices):
      if is_solution(state):
          record(state)
          return
      for choice in choices:
          if is_valid(state, choice):
              make(state, choice)         # explore
              backtrack(state, choices)
              undo(state, choice)         # backtrack ← KEY step
```

### Permutations

```python
def permutations(nums):
    results = []

    def backtrack(path, remaining):
        if not remaining:
            results.append(path[:])
            return
        for i, num in enumerate(remaining):
            path.append(num)
            backtrack(path, remaining[:i] + remaining[i+1:])
            path.pop()                    # undo

    backtrack([], nums)
    return results
```

**O(n! · n)** — unavoidable since there are n! permutations of length n.

### Subsets (Power Set)

```python
def subsets(nums):
    results = []

    def backtrack(start, path):
        results.append(path[:])           # every partial path is a valid subset
        for i in range(start, len(nums)):
            path.append(nums[i])
            backtrack(i + 1, path)
            path.pop()

    backtrack(0, [])
    return results
```

**O(2ⁿ · n)** — 2ⁿ subsets, each taking O(n) to copy.

### Combinations

```python
def combinations(nums, k):
    results = []

    def backtrack(start, path):
        if len(path) == k:
            results.append(path[:])
            return
        for i in range(start, len(nums)):
            path.append(nums[i])
            backtrack(i + 1, path)
            path.pop()

    backtrack(0, [])
    return results
```

### N-Queens Problem

Place N queens on an N×N chessboard so no two queens attack each other.

```python
def n_queens(n):
    solutions = []
    cols = set()
    diag1 = set()     # top-left to bottom-right diagonal: row - col
    diag2 = set()     # top-right to bottom-left diagonal: row + col

    def backtrack(row, board):
        if row == n:
            solutions.append(board[:])
            return
        for col in range(n):
            if col in cols or (row-col) in diag1 or (row+col) in diag2:
                continue                  # pruning — skip invalid placements
            cols.add(col); diag1.add(row-col); diag2.add(row+col)
            board.append(col)
            backtrack(row + 1, board)
            board.pop()
            cols.remove(col); diag1.remove(row-col); diag2.remove(row+col)

    backtrack(0, [])
    return solutions
```

### Sudoku Solver

```python
def solve_sudoku(board):
    def is_valid(r, c, num):
        box_r, box_c = 3 * (r // 3), 3 * (c // 3)
        for i in range(9):
            if board[r][i] == num: return False
            if board[i][c] == num: return False
            if board[box_r + i//3][box_c + i%3] == num: return False
        return True

    def backtrack():
        for r in range(9):
            for c in range(9):
                if board[r][c] == '.':
                    for num in '123456789':
                        if is_valid(r, c, num):
                            board[r][c] = num
                            if backtrack(): return True
                            board[r][c] = '.'   # undo
                    return False                # no valid digit → backtrack
        return True                             # all cells filled

    backtrack()
```

### Word Search in Grid

```python
def word_search(board, word):
    rows, cols = len(board), len(board[0])

    def dfs(r, c, idx):
        if idx == len(word): return True
        if not (0 <= r < rows and 0 <= c < cols): return False
        if board[r][c] != word[idx]: return False

        temp, board[r][c] = board[r][c], '#'   # mark visited
        found = any(dfs(r+dr, c+dc, idx+1) for dr,dc in [(0,1),(0,-1),(1,0),(-1,0)])
        board[r][c] = temp                      # restore (backtrack)
        return found

    return any(dfs(r, c, 0) for r in range(rows) for c in range(cols))
```

### Rat in a Maze

```python
def rat_in_maze(maze, n):
    path = [[0]*n for _ in range(n)]
    solutions = []

    def solve(r, c):
        if r == n-1 and c == n-1:
            solutions.append([row[:] for row in path])
            return
        for dr, dc in [(1,0),(0,1),(-1,0),(0,-1)]:
            nr, nc = r+dr, c+dc
            if 0<=nr<n and 0<=nc<n and maze[nr][nc]==1 and path[nr][nc]==0:
                path[nr][nc] = 1
                solve(nr, nc)
                path[nr][nc] = 0   # backtrack

    if maze[0][0] == 1:
        path[0][0] = 1
        solve(0, 0)
    return solutions
```

---

## Memoization (Top-Down Dynamic Programming)

**Memoization** caches the result of each unique call so it is computed only once. It transforms exponential recursion into polynomial time by eliminating redundant subproblems.

```
Without memo: fib(5) computes fib(2) three times, fib(3) twice, etc.
With memo:    each fib(k) computed exactly once.
```

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
    if n <= 1: return n
    return fib(n-1) + fib(n-2)

# Or manually:
memo = {}
def fib(n):
    if n <= 1: return n
    if n in memo: return memo[n]
    memo[n] = fib(n-1) + fib(n-2)
    return memo[n]
```

**O(n) time and space** — each of the n subproblems is solved once.

### Coin Change (Fewest Coins)

```python
@lru_cache(maxsize=None)
def coin_change(coins, amount):
    if amount == 0: return 0
    if amount < 0:  return float('inf')
    return 1 + min(coin_change(coins, amount - c) for c in coins)
```

### Longest Common Subsequence (LCS)

```python
@lru_cache(maxsize=None)
def lcs(s1, s2, i=0, j=0):
    if i == len(s1) or j == len(s2): return 0
    if s1[i] == s2[j]:
        return 1 + lcs(s1, s2, i+1, j+1)
    return max(lcs(s1, s2, i+1, j), lcs(s1, s2, i, j+1))
```

### 0/1 Knapsack

```python
@lru_cache(maxsize=None)
def knapsack(weights, values, capacity, i=0):
    if i == len(weights) or capacity == 0: return 0
    if weights[i] > capacity:
        return knapsack(weights, values, capacity, i+1)
    return max(
        values[i] + knapsack(weights, values, capacity - weights[i], i+1),
        knapsack(weights, values, capacity, i+1)
    )
```

### Edit Distance (Levenshtein)

```python
@lru_cache(maxsize=None)
def edit_distance(s1, s2, i=0, j=0):
    if i == len(s1): return len(s2) - j
    if j == len(s2): return len(s1) - i
    if s1[i] == s2[j]:
        return edit_distance(s1, s2, i+1, j+1)
    return 1 + min(
        edit_distance(s1, s2, i+1, j),    # delete from s1
        edit_distance(s1, s2, i, j+1),    # insert into s1
        edit_distance(s1, s2, i+1, j+1)   # replace
    )
```

---

## Mutual Recursion

Two or more functions that call each other. Useful for problems with naturally alternating states.

```python
def is_even(n):
    if n == 0: return True
    return is_odd(n - 1)

def is_odd(n):
    if n == 0: return False
    return is_even(n - 1)
```

### Expression Parser (Recursive Descent)

A classic mutual recursion — each grammar rule becomes a function that calls the functions for other rules.

```python
# Grammar: expr → term (('+' | '-') term)*
#          term → factor (('*' | '/') factor)*
#          factor → NUMBER | '(' expr ')'

def parse_expr(tokens, pos):
    val, pos = parse_term(tokens, pos)
    while pos < len(tokens) and tokens[pos] in ('+', '-'):
        op = tokens[pos]; pos += 1
        right, pos = parse_term(tokens, pos)
        val = val + right if op == '+' else val - right
    return val, pos

def parse_term(tokens, pos):
    val, pos = parse_factor(tokens, pos)
    while pos < len(tokens) and tokens[pos] in ('*', '/'):
        op = tokens[pos]; pos += 1
        right, pos = parse_factor(tokens, pos)
        val = val * right if op == '*' else val // right
    return val, pos

def parse_factor(tokens, pos):
    if tokens[pos] == '(':
        val, pos = parse_expr(tokens, pos + 1)
        return val, pos + 1    # skip ')'
    return int(tokens[pos]), pos + 1
```

---

## Recursion on Strings

### Palindrome Check

```python
def is_palindrome(s, lo=0, hi=None):
    if hi is None: hi = len(s) - 1
    if lo >= hi: return True
    if s[lo] != s[hi]: return False
    return is_palindrome(s, lo + 1, hi - 1)
```

### Reverse a String

```python
def reverse(s):
    if len(s) <= 1: return s
    return reverse(s[1:]) + s[0]
```

### Generate All Parentheses

```python
def generate_parentheses(n):
    results = []

    def backtrack(s, open_count, close_count):
        if len(s) == 2 * n:
            results.append(s)
            return
        if open_count < n:
            backtrack(s + '(', open_count + 1, close_count)
        if close_count < open_count:
            backtrack(s + ')', open_count, close_count + 1)

    backtrack('', 0, 0)
    return results
```

### String Permutations with Duplicates

```python
def perms_unique(s):
    results = set()

    def backtrack(path, remaining):
        if not remaining:
            results.add(''.join(path))
            return
        for i, ch in enumerate(remaining):
            path.append(ch)
            backtrack(path, remaining[:i] + remaining[i+1:])
            path.pop()

    backtrack([], list(s))
    return list(results)
```

---

## Classic Recursive Problems

### Tower of Hanoi

Move n disks from peg A to peg C using peg B as auxiliary. Only one disk may be moved at a time; a larger disk may never sit atop a smaller one.

```python
def hanoi(n, source, target, auxiliary):
    if n == 1:
        print(f"Move disk 1 from {source} to {target}")
        return
    hanoi(n-1, source, auxiliary, target)   # move n-1 disks out of the way
    print(f"Move disk {n} from {source} to {target}")
    hanoi(n-1, auxiliary, target, source)   # move n-1 disks to target

# Moves required: 2^n - 1
```

**T(n) = 2T(n−1) + 1 → O(2ⁿ)**. Provably optimal — you cannot do better than 2ⁿ−1 moves.

### Flood Fill (Paint Bucket)

```python
def flood_fill(image, r, c, new_color, original=None):
    if original is None:
        original = image[r][c]
    if r < 0 or r >= len(image): return
    if c < 0 or c >= len(image[0])): return
    if image[r][c] != original: return
    if image[r][c] == new_color: return

    image[r][c] = new_color
    flood_fill(image, r+1, c, new_color, original)
    flood_fill(image, r-1, c, new_color, original)
    flood_fill(image, r, c+1, new_color, original)
    flood_fill(image, r, c-1, new_color, original)
```

### Flatten Nested List

```python
def flatten(lst):
    result = []
    for item in lst:
        if isinstance(item, list):
            result.extend(flatten(item))   # recurse into sublists
        else:
            result.append(item)
    return result

# flatten([1, [2, [3, 4], 5], [6]]) → [1, 2, 3, 4, 5, 6]
```

### Binary Search (Recursive)

```python
def binary_search(arr, target, lo=0, hi=None):
    if hi is None: hi = len(arr) - 1
    if lo > hi: return -1
    mid = (lo + hi) // 2
    if arr[mid] == target: return mid
    if arr[mid] < target:  return binary_search(arr, target, mid+1, hi)
    return binary_search(arr, target, lo, mid-1)
```

### GCD (Euclidean Algorithm)

```python
def gcd(a, b):
    if b == 0: return a
    return gcd(b, a % b)
```

**T(n) = T(n mod b) + O(1) → O(log(min(a,b)))**. Each step reduces the larger number by at least half every two steps.

### Power Set

```python
def power_set(s):
    if not s: return [[]]
    first = s[0]
    rest  = power_set(s[1:])
    return rest + [[first] + subset for subset in rest]
```

---

## Recursion on Numbers

### Count Digits

```python
def count_digits(n):
    if n < 10: return 1
    return 1 + count_digits(n // 10)
```

### Check if Number is Palindrome

```python
def reverse_num(n, acc=0):
    if n == 0: return acc
    return reverse_num(n // 10, acc * 10 + n % 10)

def is_palindrome_num(n):
    return n == reverse_num(n)
```

### Sum of Digits

```python
def digit_sum(n):
    if n < 10: return n
    return n % 10 + digit_sum(n // 10)
```

---

## Advanced: Structural Recursion

Recursion that mirrors the **inductive definition** of a data structure. If the structure is recursive (lists, trees, graphs), the algorithm naturally is too.

### Merge K Sorted Lists

```python
def merge_k_lists(lists):
    if not lists: return None
    if len(lists) == 1: return lists[0]
    mid = len(lists) // 2
    left  = merge_k_lists(lists[:mid])
    right = merge_k_lists(lists[mid:])
    return merge_two(left, right)           # standard 2-list merge

# T(n, k) = O(n log k) where n = total nodes, k = number of lists
```

### Serialize / Deserialize Binary Tree

```python
def serialize(root):
    if root is None: return "null,"
    return str(root.val) + "," + serialize(root.left) + serialize(root.right)

def deserialize(data):
    vals = iter(data.split(","))

    def build():
        val = next(vals)
        if val == "null": return None
        node = Node(int(val))
        node.left  = build()
        node.right = build()
        return node

    return build()
```

### Recursive JSON / Tree Builder

```python
def build_tree(preorder, inorder):
    if not inorder: return None
    root_val = preorder[0]
    root = Node(root_val)
    mid = inorder.index(root_val)
    root.left  = build_tree(preorder[1:mid+1], inorder[:mid])
    root.right = build_tree(preorder[mid+1:],  inorder[mid+1:])
    return root
```

---

## Eliminating Recursion

When recursion depth is too large or performance is critical, convert to iteration.

### Using an Explicit Stack

Any DFS / recursive algorithm can be rewritten with an explicit stack:

```python
# Recursive DFS
def dfs_rec(node):
    if node is None: return
    process(node)
    dfs_rec(node.left)
    dfs_rec(node.right)

# Iterative DFS with explicit stack
def dfs_iter(root):
    stack = [root]
    while stack:
        node = stack.pop()
        if node is None: continue
        process(node)
        stack.append(node.right)   # right pushed first → left processed first
        stack.append(node.left)
```

### Trampoline (for Tail Recursion without TCO)

A trampoline runs a function repeatedly until it returns a value rather than another function call — simulating TCO in languages that don't support it natively.

```python
def trampoline(f, *args):
    result = f(*args)
    while callable(result):
        result = result()
    return result

def factorial_tramp(n, acc=1):
    if n == 0: return acc
    return lambda: factorial_tramp(n-1, n*acc)   # return thunk, not call

trampoline(factorial_tramp, 10000)   # no stack overflow
```

### Continuation-Passing Style (CPS)

Transform recursion so that the "what to do next" logic is passed as a function (continuation), enabling iterative simulation.

```python
def factorial_cps(n, cont=lambda x: x):
    if n == 0:
        return cont(1)
    return factorial_cps(n-1, lambda result: cont(n * result))
```

---

## Pitfalls and Best Practices

**Missing or incorrect base case**
```python
# Infinite recursion — base case never reached
def countdown(n):
    print(n)
    return countdown(n - 1)   # forgot: if n == 0: return
```

**Not reducing the problem**
```python
def bad(n):
    if n == 0: return 0
    return bad(n)    # n never decreases — infinite loop
```

**Redundant computation without memoization**
```python
fib(40)   # ~2^40 ≈ 1 trillion calls without memoization
          # 40 unique subproblems — memo makes it instant
```

**Excessive copying of data in Python**
```python
# Slow: creates new lists every call — O(n²) total copies
def merge_sort(arr):
    return merge_sort(arr[:mid]) + merge_sort(arr[mid:])

# Better: pass indices, operate in-place
def merge_sort(arr, lo, hi): ...
```

**Mutating shared state across branches**
```python
# BUG: path is shared across branches, mutations persist
def backtrack(path, choices):
    path.append(choice)          # mutate
    backtrack(path, ...)
    # forgot: path.pop()         # must undo mutation on backtrack
```

---

## Quick Reference Card

```
PATTERN              SIGNATURE                    COMPLEXITY
Linear recursion     f(n) calls f(n-1)            O(n) time, O(n) space
Tail recursion       f(n) = return f(n-1,acc)     O(n) time, O(1) space (w/ TCO)
Binary recursion     f(n) calls f(n-1)+f(n-2)     O(2^n) without memo
Divide & conquer     f(n) = 2·f(n/2) + O(n)       O(n log n)
Fast power           f(n) = f(n/2)²               O(log n)
Tree traversal       f(node) = f(left)+f(right)   O(n)
Backtracking         try → recurse → undo         O(b^d): b=branching, d=depth
Memoized recursion   cache(subproblem) → reuse    O(states × transition cost)
Mutual recursion     f calls g, g calls f         depends on structure

MASTER THEOREM SHORTCUTS
  T(n) = T(n/2) + O(1)   → O(log n)       ← binary search
  T(n) = 2T(n/2) + O(n)  → O(n log n)     ← merge sort
  T(n) = T(n-1) + O(1)   → O(n)           ← factorial
  T(n) = 2T(n-1) + O(1)  → O(2^n)         ← tower of hanoi

WHEN TO MEMOIZE
  Same subproblem called more than once?  → add @lru_cache
  Call tree looks like a DAG not a tree?  → definitely memoize
  Exponential without memo?               → memo drops it to polynomial

WHEN TO AVOID RECURSION
  Depth > ~1000 (Python)                  → use iteration or trampoline
  No repeated subproblems                 → iteration is cleaner
  Performance-critical inner loop        → iteration has lower overhead
```
