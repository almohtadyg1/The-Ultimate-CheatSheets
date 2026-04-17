# Sorting Algorithms: A Complete Progressive Tutorial

---

## 1. What & Why

Sorting is the process of rearranging a collection into a defined order — ascending, descending, or by some custom criterion. It is one of the most studied problems in computer science, and for good reason: sorted data unlocks dramatically faster operations. Binary search only works on sorted arrays. Database indexes are sorted structures. Merge operations are linear on sorted inputs but quadratic on unsorted ones.

You will rarely implement a sorting algorithm from scratch in production. Every modern language has a battle-tested sort in its standard library. What you need to understand is: which algorithm does your language use, when does it perform poorly, and what are the trade-offs between stability, space, and speed — because these determine whether you should reach for the standard sort or something specialized.

The key concepts you need to internalize: **stability** (do equal elements preserve their original order?), **in-place vs auxiliary space**, **comparison-based vs non-comparison-based**, and the proven lower bound of O(n log n) for any comparison sort.

---

## 2. Mental Model

Think of sorting algorithms as different strategies for organizing a hand of playing cards.

**Insertion sort**: pick up cards one at a time, slide each into its correct position among the already-sorted cards in your hand. Natural for humans.

**Selection sort**: scan all remaining cards, find the smallest, put it in position. Then repeat. Minimal number of swaps.

**Merge sort**: split the deck in half, sort each half independently, then merge two sorted decks into one sorted deck — merge is easy because both halves are already sorted.

**Quick sort**: pick a card as a pivot, put all smaller cards to its left and larger to its right. Now the pivot is in its final position. Recursively sort each side.

**Counting sort**: count how many of each value exist, then reconstruct. Works only if you know the range of values in advance.

```
n=7: [38, 27, 43, 3, 9, 82, 10]

Insertion sort:  look at each, slide left until in place
Selection sort:  find min(3), swap to front. find min(9), swap. etc.
Merge sort:      [38,27,43,3] [9,82,10] → sort each → merge
Quick sort:      pivot=38, left=[27,3,9,10], right=[43,82] → recurse
Counting sort:   count each value, rebuild in order
```

---

## 3. Progressive Examples

### Level 1: Bubble Sort — Understand Before Moving On

Bubble sort is the "hello world" of sorting algorithms. It's rarely used in production, but it illustrates the key concepts: comparison, swap, and loop invariants.

```python
def bubble_sort(arr):
    """
    Repeatedly passes through the array, swapping adjacent elements that are
    out of order. Each full pass 'bubbles' the largest unsorted element to its
    final position.

    After pass k, the last k elements are in their final sorted positions.
    This is the loop invariant.
    """
    n = len(arr)
    arr = arr.copy()    # don't modify the original

    for i in range(n):
        swapped = False
        # Each pass shrinks the unsorted region by 1 (the right i elements are sorted)
        for j in range(n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                swapped = True
        # Optimization: if no swaps occurred, the array is already sorted
        if not swapped:
            break    # best case: O(n) for already-sorted input

    return arr

# Example trace:
# [38, 27, 43, 3]
# Pass 1: compare(38,27)->swap, compare(38,43)->no, compare(43,3)->swap
#         [27, 38, 3, 43]   <- 43 is now in its final position
# Pass 2: compare(27,38)->no, compare(38,3)->swap
#         [27, 3, 38, 43]   <- 38 and 43 in final positions
# Pass 3: compare(27,3)->swap
#         [3, 27, 38, 43]   <- sorted

print(bubble_sort([38, 27, 43, 3, 9, 82, 10]))
# [3, 9, 10, 27, 38, 43, 82]
```

### Level 2: Insertion Sort — The Algorithm Your Brain Uses

```python
def insertion_sort(arr):
    """
    Maintains a sorted 'left portion' of the array.
    For each new element, shifts sorted elements right until finding
    the correct insertion position.

    Excellent for: small arrays (< 64 elements), nearly sorted data.
    This is why Python's Timsort uses insertion sort for small runs.
    """
    arr = arr.copy()

    for i in range(1, len(arr)):
        key = arr[i]        # the element to be inserted into the sorted portion
        j = i - 1

        # Shift elements that are greater than key one position to the right
        while j >= 0 and arr[j] > key:
            arr[j + 1] = arr[j]   # shift right (not a swap — fewer operations)
            j -= 1

        arr[j + 1] = key           # place key in its correct position

    return arr

# Why this beats bubble sort on nearly sorted data:
# For [1, 2, 3, 4, 5, 6, 0]:
# Only the last element (0) needs to travel far.
# All others: the while loop terminates in 0 or 1 iterations.
# Total work: ~n + n = O(n) — the adaptive best case.

# Insertion sort is also stable:
# Equal elements are not moved past each other — original order preserved.
students = [("Alice", 90), ("Bob", 85), ("Carol", 90), ("Dave", 85)]
# After sorting by score: Alice before Carol (same 90), Bob before Dave (same 85)
def insertion_sort_stable(arr, key=lambda x: x):
    arr = arr.copy()
    for i in range(1, len(arr)):
        cur = arr[i]
        j = i - 1
        while j >= 0 and key(arr[j]) > key(cur):  # strictly greater, not >=
            arr[j + 1] = arr[j]
            j -= 1
        arr[j + 1] = cur
    return arr
```

### Level 3: Merge Sort — Guaranteed O(n log n)

```python
def merge_sort(arr):
    """
    Divide-and-conquer: split in half, sort each half recursively,
    merge two sorted halves into one sorted result.

    The merge step is the key insight: merging two sorted arrays is O(n).
    At each level of recursion, O(n) merge work is done.
    There are O(log n) levels. Total: O(n log n).
    """
    if len(arr) <= 1:
        return arr      # base case: a single element is already sorted

    mid = len(arr) // 2
    left = merge_sort(arr[:mid])    # sort left half
    right = merge_sort(arr[mid:])   # sort right half
    return merge(left, right)

def merge(left, right):
    """
    Merge two sorted arrays into one sorted array.
    O(n) time, O(n) space.
    """
    result = []
    i = j = 0

    while i < len(left) and j < len(right):
        if left[i] <= right[j]:     # <= preserves stability (equal elements keep order)
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1

    result.extend(left[i:])     # append any remaining elements
    result.extend(right[j:])
    return result

# Trace on [38, 27, 43, 3]:
#
# merge_sort([38, 27, 43, 3])
# ├── merge_sort([38, 27])
# │   ├── merge_sort([38]) → [38]
# │   ├── merge_sort([27]) → [27]
# │   └── merge([38],[27]) → [27, 38]
# ├── merge_sort([43, 3])
# │   ├── merge_sort([43]) → [43]
# │   ├── merge_sort([3])  → [3]
# │   └── merge([43],[3])  → [3, 43]
# └── merge([27,38],[3,43]) → [3, 27, 38, 43]

print(merge_sort([38, 27, 43, 3, 9, 82, 10]))
# [3, 9, 10, 27, 38, 43, 82]

# Real-world use: merge sort is the algorithm of choice when:
# - You need a guaranteed O(n log n) worst case (quicksort is O(n²) worst case)
# - Stability is required
# - You're sorting linked lists (doesn't require random access)
# - External sorting (data too large for RAM — merge in chunks)
```

### Level 4: Quicksort — The Fastest in Practice

```python
import random

def quicksort(arr, low=0, high=None):
    """
    In-place quicksort with random pivot selection.
    
    Partition: choose a pivot, move all smaller elements left,
    all larger elements right. The pivot is now in its final position.
    Recurse on both sides.

    Random pivot makes worst-case O(n²) astronomically unlikely in practice.
    O(n log n) average, O(log n) stack space.
    """
    if high is None:
        high = len(arr) - 1
        arr = arr.copy()   # avoid modifying input

    if low < high:
        pivot_idx = partition(arr, low, high)
        quicksort(arr, low, pivot_idx - 1)      # sort left of pivot
        quicksort(arr, pivot_idx + 1, high)     # sort right of pivot

    return arr if high == len(arr) - 1 and low == 0 else None

def partition(arr, low, high):
    """
    Lomuto partition scheme.
    Choose random pivot, move elements < pivot to the left.
    Returns the final index of the pivot.
    """
    # Randomize pivot to avoid O(n²) on sorted input
    rand_idx = random.randint(low, high)
    arr[rand_idx], arr[high] = arr[high], arr[rand_idx]

    pivot = arr[high]
    i = low - 1     # i tracks the boundary of the "less than pivot" region

    for j in range(low, high):
        if arr[j] <= pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]   # expand the "less than" region

    arr[i + 1], arr[high] = arr[high], arr[i + 1]   # place pivot in final position
    return i + 1

arr = [38, 27, 43, 3, 9, 82, 10]
print(quicksort(arr))   # [3, 9, 10, 27, 38, 43, 82]

# Why quicksort beats merge sort in practice despite same Big O:
# 1. In-place: O(log n) stack space vs O(n) auxiliary array for merge sort
# 2. Cache efficiency: accesses memory sequentially in partition, cache-friendly
# 3. Smaller constant factor: fewer memory allocations
# C++ STL uses Introsort (quicksort + heapsort fallback + insertion sort for small n)
```

### Level 5: Linear-Time Sorts — Breaking the O(n log n) Barrier

```python
def counting_sort(arr, max_val=None):
    """
    Counting sort: O(n + k) time and space, where k = range of values.
    Only works on non-negative integers with a known bounded range.
    
    Works by counting occurrences of each value, then reconstructing.
    Stable — maintains relative order of equal elements.
    """
    if not arr:
        return []

    max_val = max_val or max(arr)
    counts = [0] * (max_val + 1)

    for val in arr:
        counts[val] += 1        # count each value

    result = []
    for val, count in enumerate(counts):
        result.extend([val] * count)   # output each value count times

    return result

print(counting_sort([4, 2, 2, 8, 3, 3, 1]))   # [1, 2, 2, 3, 3, 4, 8]

# Radix sort: sort integers digit by digit, least significant to most significant
def radix_sort(arr):
    """
    O(nk) where k = number of digits. For fixed-size integers, this is O(n).
    Uses counting sort as a stable subroutine for each digit position.
    """
    if not arr:
        return arr

    arr = arr.copy()
    max_val = max(arr)
    exp = 1   # current digit position (1 = ones, 10 = tens, etc.)

    while max_val // exp > 0:
        arr = counting_sort_by_digit(arr, exp)
        exp *= 10

    return arr

def counting_sort_by_digit(arr, exp):
    n = len(arr)
    output = [0] * n
    count = [0] * 10   # digits 0-9

    for val in arr:
        digit = (val // exp) % 10
        count[digit] += 1

    # Cumulative count — tells us the final position of each digit
    for i in range(1, 10):
        count[i] += count[i - 1]

    # Build output (traverse backwards for stability)
    for i in range(n - 1, -1, -1):
        digit = (arr[i] // exp) % 10
        output[count[digit] - 1] = arr[i]
        count[digit] -= 1

    return output

data = [170, 45, 75, 90, 802, 24, 2, 66]
print(radix_sort(data))   # [2, 24, 45, 66, 75, 90, 170, 802]

# When to use radix sort:
# - Sorting millions of integers with a known small range (IDs, phone numbers, ages)
# - When you need guaranteed linear time and your data fits the constraints
# - NOT appropriate for floating-point numbers, strings of varying length, or when
#   the key range k >> n (counting sort's O(k) space becomes a problem)
```

### Level 6: Python's Timsort — What You Actually Use

```python
# Python's built-in sort is Timsort, a hybrid of merge sort and insertion sort.
# Understanding it makes you a better user of sorted() and list.sort().

# Key properties of Timsort:
# - Stable: equal elements maintain original order
# - O(n) best case (already sorted or nearly sorted data)
# - O(n log n) average and worst case
# - O(n) space
# - Adaptive: exploits existing order in real-world data

# Basic usage
numbers = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3]

sorted_copy = sorted(numbers)           # returns new list, original unchanged
numbers.sort()                          # sorts in-place, returns None

# Custom sort key — the key function is called once per element
students = [
    {"name": "Alice", "gpa": 3.8, "age": 22},
    {"name": "Bob",   "gpa": 3.6, "age": 21},
    {"name": "Carol", "gpa": 3.8, "age": 20},
]

# Sort by GPA descending, then age ascending (for ties)
sorted_students = sorted(students, key=lambda s: (-s["gpa"], s["age"]))
for s in sorted_students:
    print(f"{s['name']}: GPA {s['gpa']}, Age {s['age']}")
# Carol: GPA 3.8, Age 20     <- lower age first among tied GPAs (stable)
# Alice: GPA 3.8, Age 22
# Bob:   GPA 3.6, Age 21

# Stability in action: sort by name, then by GPA — name sort is stable
by_name = sorted(students, key=lambda s: s["name"])
by_gpa = sorted(by_name, key=lambda s: s["gpa"])
# Among students with the same GPA, they'll be in name order (stable behavior)

# Sorting complex objects with functools
from functools import cmp_to_key

def compare_version(a, b):
    """Compare semantic version strings: '1.10.0' > '1.9.0'"""
    parts_a = list(map(int, a.split('.')))
    parts_b = list(map(int, b.split('.')))
    for pa, pb in zip(parts_a, parts_b):
        if pa != pb:
            return (pa > pb) - (pa < pb)   # -1, 0, or 1
    return len(parts_a) - len(parts_b)

versions = ["1.10.0", "1.9.0", "2.0.0", "1.9.1"]
print(sorted(versions, key=cmp_to_key(compare_version)))
# ['1.9.0', '1.9.1', '1.10.0', '2.0.0']
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Using the wrong sort for the data type**

```python
# WRONG: lexicographic sort on numbers (string comparison, not numeric)
numbers = [10, 9, 100, 2, 20]
numbers.sort(key=str)   # treats numbers as strings
print(numbers)           # [10, 100, 2, 20, 9] — alphabetical, not numeric

# CORRECT: default numeric sort
numbers.sort()
print(numbers)           # [2, 9, 10, 20, 100]

# Same trap with version strings
versions = ["1.10.0", "1.9.0", "2.0.0"]
versions.sort()         # lexicographic: "1.10.0" < "1.9.0" because "1" < "9" — WRONG
```

**Mistake 2: Assuming sort() returns the sorted list**

```python
# WRONG: common mistake in Python
result = [3, 1, 2].sort()    # sort() returns None!
print(result)                 # None

# CORRECT:
arr = [3, 1, 2]
arr.sort()        # modifies arr in-place
print(arr)        # [1, 2, 3]

# OR: use sorted() which returns a new list
result = sorted([3, 1, 2])
print(result)     # [1, 2, 3]
```

**Mistake 3: Using an unstable sort when stability matters**

```python
# You have records sorted by date. You want to sort by user ID.
# If the sort is stable, records with the same user ID stay in date order.
# If not stable, same-ID records may be scrambled.

# Python's sorted() is always stable — safe to use for multi-key sorting.
# Quicksort (the classic algorithm) is NOT stable.
# C++ std::sort is NOT guaranteed stable — use std::stable_sort if needed.

# The canonical pattern for multi-key sort using stability:
records = [
    ("Alice", "2024-01-03"),
    ("Bob",   "2024-01-01"),
    ("Alice", "2024-01-01"),
    ("Bob",   "2024-01-02"),
]
# Sort by user, then by date — use tuple key for clean multi-key sort
sorted_records = sorted(records, key=lambda r: (r[0], r[1]))
# Alice 2024-01-01, Alice 2024-01-03, Bob 2024-01-01, Bob 2024-01-02
```

**Mistake 4: Using quicksort with a bad pivot on sorted data**

```python
# WRONG: naive quicksort always picks first element as pivot
def bad_quicksort(arr):
    if len(arr) <= 1:
        return arr
    pivot = arr[0]              # always first element
    left  = [x for x in arr[1:] if x <= pivot]
    right = [x for x in arr[1:] if x > pivot]
    return bad_quicksort(left) + [pivot] + bad_quicksort(right)

# On [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]:
# Pivot=1, left=[], right=[2..10] — perfectly unbalanced every time
# Recursion depth: n — stack overflow on large inputs
# Time: O(n²)

# CORRECT: random pivot, or median-of-three
# Python's built-in sort handles this — use it for production code
```

**Mistake 5: Using counting sort when k >> n**

```python
# Counting sort for ages (0-120): k=121, n=whatever. Fine.
# Counting sort for user IDs (0-2,147,483,647): k=2 billion, allocates 2GB. Catastrophic.

# Rule: use counting sort only when max_value is not much larger than n
def should_use_counting_sort(arr):
    n = len(arr)
    k = max(arr) - min(arr) + 1
    # General guideline: counting sort makes sense when k ≤ 2n
    return k <= 2 * n
```

---

## 5. The "Why Does This Work" Layer

### Why O(n log n) Is the Lower Bound for Comparison Sorts

Any sorting algorithm that only uses comparisons (a < b, a == b, a > b) faces a fundamental information-theoretic limit. To sort n elements, the algorithm must distinguish between all n! possible orderings of the input. Each comparison gives one bit of information — it eliminates roughly half the remaining possibilities.

To distinguish n! outcomes with binary decisions, you need at least log₂(n!) decisions. By Stirling's approximation: log₂(n!) ≈ n·log₂(n). This is a mathematical lower bound — no comparison sort can do better.

Merge sort and Timsort achieve this bound. They are optimal among comparison-based algorithms.

### Why Quicksort Is Faster Than Merge Sort in Practice

Both are O(n log n), but quicksort has smaller constant factors for several reasons:

**Memory locality**: quicksort's partition step accesses array elements sequentially, which is cache-friendly. Merge sort needs to copy data into an auxiliary array, causing more cache misses.

**No auxiliary allocation**: the in-place partition needs only O(log n) stack space. Merge sort needs O(n) extra memory for the merge buffer — this allocation and the copying add overhead.

**Adaptability**: modern quicksort implementations (Introsort, Timsort) switch to insertion sort for small subarrays (typically < 16 elements), where insertion sort's tight inner loop beats the overhead of recursive calls.

### How Timsort Exploits Real-World Data Structure

Real-world data is almost never random. Logs are nearly chronological. Lists of names may be partially alphabetized. Databases return results in clustered order.

Timsort exploits this by finding "runs" — already-sorted subsequences in the input. It extends short runs using insertion sort (which is O(n) on nearly sorted data), then merges runs using merge sort's merge operation.

For data that is already sorted: Timsort finds one run of length n, no merges needed → O(n). For reverse-sorted: one reversed run, one reversal → O(n). For random data: behaves like merge sort → O(n log n).

---

## 6. Quick Reference

### Algorithm Comparison

| Algorithm | Best | Average | Worst | Space | Stable | Use When |
|-----------|------|---------|-------|-------|--------|----------|
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Yes | n < 64, nearly sorted |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes | Stability required, linked lists |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | No | General purpose, speed |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No | Worst-case guarantee + in-place |
| Timsort | O(n) | O(n log n) | O(n log n) | O(n) | Yes | Python/Java default — real-world data |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(k) | Yes | Small integer range |
| Radix Sort | O(nk) | O(nk) | O(nk) | O(n+k) | Yes | Fixed-width integers at scale |

### Decision Guide

```
Need to sort in Python or Java?      → Use built-in sort (Timsort)
Need in-place + worst-case O(n log n)?  → Heapsort
Need stable + guaranteed O(n log n)?    → Merge sort
Need fast average case, don't care about stability? → Quicksort
Integers with small range [0..k]?       → Counting sort (if k is not >> n)
Large volume of fixed-width integers?   → Radix sort
n < 64 or nearly sorted?                → Insertion sort
```

### Stability Decision

Use a **stable** sort when:
- You're sorting by multiple keys in succession (sort by date, then by name)
- Equal elements have meaningful order that must be preserved
- The algorithm must be deterministic and predictable on all inputs

Java: `Arrays.sort()` is stable. C++: `std::stable_sort()` is stable, `std::sort()` is not. Python: `sorted()` and `list.sort()` are always stable.
