# Sorting Algorithms Cheat Sheet
### From Zero to Expert

---

## What is Sorting?

**Sorting** is the process of arranging elements in a defined order — typically ascending or descending. It is one of the most fundamental operations in computer science, underpinning search, databases, compression, rendering, and almost every non-trivial algorithm.

```
Unsorted: [38, 27, 43, 3, 9, 82, 10]
Sorted:   [ 3,  9, 10, 27, 38, 43, 82]
```

---

## Core Concepts & Vocabulary

| Term | Meaning |
|---|---|
| **In-place** | Sorts using O(1) or O(log n) extra memory (no auxiliary array) |
| **Stable** | Equal elements preserve their original relative order |
| **Adaptive** | Runs faster on already partially sorted input |
| **Online** | Can sort a stream — processes elements one at a time without seeing the full input |
| **Comparison-based** | Uses only `<`, `>`, `==` to determine order |
| **Non-comparison** | Exploits structure of the data (digits, bytes) to sort faster than O(n log n) |
| **Internal sort** | All data fits in RAM |
| **External sort** | Data is too large for RAM; lives on disk |

### The Comparison Sort Lower Bound
Any algorithm that sorts purely by comparing elements **cannot do better than O(n log n)** in the worst case. This is a provable information-theoretic limit — there are n! possible orderings, and you need at least log₂(n!) ≈ n log n comparisons to distinguish them all. Non-comparison sorts break this barrier by using extra information about the data.

---

## Complexity Master Table

| Algorithm | Best | Average | Worst | Space | Stable | Notes |
|---|---|---|---|---|---|---|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ | Adaptive with early exit |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | ❌ | Minimal swaps |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ | Best for small / nearly sorted |
| Shell Sort | O(n log n) | O(n^1.5) | O(n²) | O(1) | ❌ | Gap-sequence dependent |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | ✅ | Guaranteed O(n log n) |
| Quicksort | O(n log n) | O(n log n) | O(n²) | O(log n) | ❌ | Fastest in practice |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | ❌ | Guaranteed, in-place |
| Tim Sort | O(n) | O(n log n) | O(n log n) | O(n) | ✅ | Python/Java default |
| Introsort | O(n log n) | O(n log n) | O(n log n) | O(log n) | ❌ | C++ STL default |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(k) | ✅ | Integer keys in range k |
| Radix Sort | O(nk) | O(nk) | O(nk) | O(n+k) | ✅ | k = number of digits |
| Bucket Sort | O(n+k) | O(n+k) | O(n²) | O(n+k) | ✅ | Uniform distribution |
| Cycle Sort | O(n²) | O(n²) | O(n²) | O(1) | ❌ | Minimum writes to memory |
| Cocktail Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ | Bidirectional bubble sort |
| Gnome Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ | Simplest possible sort |
| Comb Sort | O(n log n) | O(n²/2^p) | O(n²) | O(1) | ❌ | Improved bubble sort |
| Pigeonhole Sort | O(n+k) | O(n+k) | O(n+k) | O(n+k) | ✅ | Similar to counting sort |
| Strand Sort | O(n) | O(n²) | O(n²) | O(n) | ✅ | Natural on sorted sublists |
| Patience Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | ❌ | LIS foundation |
| Tree Sort | O(n log n) | O(n log n) | O(n²)* | O(n) | ✅ | *O(n log n) with balanced BST |
| Library Sort | O(n log n) | O(n log n) | O(n²) | O(n) | ✅ | Gapped insertion sort |
| Block Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | ✅ | In-place merge sort |

---

## Bubble Sort

The simplest sorting algorithm. Repeatedly steps through the list, **bubbles** the largest unsorted element to its correct position by swapping adjacent pairs.

```
Pass 1: [38, 27, 43, 3] → [27, 38, 3, 43]
Pass 2: [27, 38, 3, 43] → [27, 3, 38, 43]
Pass 3: [27, 3, 38, 43] → [3, 27, 38, 43]
```

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        swapped = False
        for j in range(n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                swapped = True
        if not swapped:       # already sorted — adaptive O(n) best case
            break
```

**Why it matters:** Simple to understand; demonstrates how loop invariants work. Almost never used in production.

---

## Selection Sort

Finds the **minimum element** in the unsorted region and places it at the front. Exactly n-1 swaps — ideal when writes to memory are expensive.

```
[64, 25, 12, 22, 11]
 ↓ find min (11)
[11, 25, 12, 22, 64]
     ↓ find min (12)
[11, 12, 25, 22, 64]
         ↓ find min (22)
[11, 12, 22, 25, 64]
```

```python
def selection_sort(arr):
    n = len(arr)
    for i in range(n):
        min_idx = i
        for j in range(i + 1, n):
            if arr[j] < arr[min_idx]:
                min_idx = j
        arr[i], arr[min_idx] = arr[min_idx], arr[i]
```

**Not stable** — swapping the minimum past equal elements can change their relative order.

---

## Insertion Sort

Builds the sorted array **one element at a time**, inserting each new element into its correct position among already-sorted elements — like sorting a hand of playing cards.

```
[5, 2, 4, 6, 1, 3]
[2, 5, 4, 6, 1, 3]   ← insert 2
[2, 4, 5, 6, 1, 3]   ← insert 4
[2, 4, 5, 6, 1, 3]   ← insert 6 (already in place)
[1, 2, 4, 5, 6, 3]   ← insert 1
[1, 2, 3, 4, 5, 6]   ← insert 3
```

```python
def insertion_sort(arr):
    for i in range(1, len(arr)):
        key = arr[i]
        j = i - 1
        while j >= 0 and arr[j] > key:
            arr[j + 1] = arr[j]
            j -= 1
        arr[j + 1] = key
```

**Golden rule:** Use insertion sort for arrays with n ≤ 16. It beats quicksort on tiny inputs due to its tiny constant factor. Tim Sort uses it for small subarrays for exactly this reason.

---

## Shell Sort

A generalization of insertion sort. Instead of comparing adjacent elements, compares elements **h positions apart**, then progressively reduces h to 1 (a final insertion sort pass). Large early gaps move elements far quickly.

```
Gap sequence (Knuth): 1, 4, 13, 40, 121, …  →  h = (3h + 1)

Array: [8, 7, 6, 1, 3, 5, 2, 4]

Gap=4:  compare/swap elements 4 apart → [3, 5, 2, 1, 8, 7, 6, 4]
Gap=1:  final insertion sort (nearly sorted) → [1, 2, 3, 4, 5, 6, 7, 8]
```

Performance depends heavily on the **gap sequence**. Best known: Ciura's sequence `[1, 4, 10, 23, 57, 132, 301, 701]` giving O(n^(4/3)) experimentally.

---

## Merge Sort

A classic **divide and conquer** algorithm. Recursively splits the array in half, sorts each half, then **merges** the two sorted halves.

```
         [38, 27, 43, 3, 9, 82]
              /            \
      [38, 27, 43]       [3, 9, 82]
        /       \          /      \
   [38, 27]   [43]      [3, 9]   [82]
    /     \              /    \
 [38]    [27]          [3]    [9]

Merge back up:
[27, 38] + [43] → [27, 38, 43]
[3, 9]   + [82] → [3, 9, 82]
[27, 38, 43] + [3, 9, 82] → [3, 9, 27, 38, 43, 82]
```

```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    mid = len(arr) // 2
    left  = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    return merge(left, right)

def merge(left, right):
    result, i, j = [], 0, 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i]); i += 1
        else:
            result.append(right[j]); j += 1
    return result + left[i:] + right[j:]
```

**Guaranteed O(n log n)** regardless of input. The algorithm of choice for **linked lists** (no random access needed) and **external sorting** (can merge sorted chunks from disk).

### Bottom-Up Merge Sort
Iterative variant — avoids recursion by merging runs of size 1, then 2, then 4, then 8… Uses O(1) stack space.

---

## Quicksort

The fastest general-purpose sorting algorithm in practice. Choose a **pivot**, partition the array so all elements less than the pivot are left of it and all greater are right, then recursively sort each side.

```
Pivot = 3 (last element)
[10, 80, 30, 90, 40, 50, 70, 3]

Partition:
[ 3 | 80, 30, 90, 40, 50, 70, 10]
  ↑ pivot placed in final position

Recurse on left [] and right [80, 30, 90, 40, 50, 70, 10]
```

```python
def quicksort(arr, lo, hi):
    if lo < hi:
        pivot_idx = partition(arr, lo, hi)
        quicksort(arr, lo, pivot_idx - 1)
        quicksort(arr, pivot_idx + 1, hi)

def partition(arr, lo, hi):         # Lomuto scheme
    pivot = arr[hi]
    i = lo - 1
    for j in range(lo, hi):
        if arr[j] <= pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]
    arr[i + 1], arr[hi] = arr[hi], arr[i + 1]
    return i + 1
```

### Pivot Strategies
| Strategy | Notes |
|---|---|
| First / Last element | Simple; O(n²) on sorted input |
| Random element | O(n²) worst case becomes statistically negligible |
| Median-of-three | Pick median of first, middle, last — good in practice |
| Ninther | Median of medians — used in Introsort |

### Hoare Partition
The original partition scheme. Uses two pointers converging inward — ~3× fewer swaps than Lomuto on average.

### 3-Way Quicksort (Dutch National Flag)
Partitions into three regions: `< pivot`, `== pivot`, `> pivot`.  
Reduces O(n²) to O(n) on arrays with many duplicate keys.

---

## Heap Sort

Uses a **max-heap** to sort in place. First builds a heap from the array, then repeatedly extracts the maximum and places it at the end.

```
Phase 1 — Build max-heap (heapify):
[4, 10, 3, 5, 1] → [10, 5, 3, 4, 1]

Phase 2 — Extract max repeatedly:
Swap root (10) with last → [1, 5, 3, 4, | 10]
Heapify down           → [5, 4, 3, 1, | 10]
Swap root (5)  with last → [1, 4, 3, | 5, 10]
… continue until sorted  → [1, 3, 4, 5, 10]
```

```python
def heap_sort(arr):
    n = len(arr)
    for i in range(n // 2 - 1, -1, -1):   # build max-heap
        heapify(arr, n, i)
    for i in range(n - 1, 0, -1):          # extract elements
        arr[0], arr[i] = arr[i], arr[0]
        heapify(arr, i, 0)

def heapify(arr, n, i):
    largest = i
    l, r = 2*i+1, 2*i+2
    if l < n and arr[l] > arr[largest]: largest = l
    if r < n and arr[r] > arr[largest]: largest = r
    if largest != i:
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, n, largest)
```

**Guaranteed O(n log n), in-place, not stable.**  
Slower than quicksort in practice due to poor cache locality (heap jumps around memory). Used as the fallback in Introsort.

---

## Tim Sort

The **de facto standard** sorting algorithm for general-purpose use. Powers Python's `sorted()`, Java's `Arrays.sort()` for objects, and Android's sort.

### How It Works
1. **Scan** the input for natural **runs** (already sorted or reverse-sorted sequences). Reverse any descending runs to make them ascending.
2. **Extend** runs shorter than `minrun` (~32–64 elements) using insertion sort.
3. **Merge** runs using a stack with rules that ensure runs are merged in near-optimal order (maintaining the invariants: `|Z| > |Y| + |X|` and `|Y| > |X|`).

```
Input: [5,4,3,2,1, 6,7,8,9,10, 3,1,2]

Run 1 (descending, reversed): [1,2,3,4,5]
Run 2 (ascending):            [6,7,8,9,10]
Run 3 (short, insertion sort):[1,2,3]

Merge runs → [1,1,2,2,3,3,4,5,6,7,8,9,10]
```

**O(n) on already-sorted or reverse-sorted data.** The merge uses **galloping mode** — when one run is consistently winning the merge, it switches to exponential search to skip ahead faster.

---

## Introsort

The algorithm behind **C++ `std::sort`**. A hybrid that starts as quicksort and switches strategy to avoid worst-case behavior.

```
if depth_limit exceeded:
    → switch to Heap Sort     (O(n log n) guarantee)
elif partition size ≤ 16:
    → switch to Insertion Sort (fastest for small n)
else:
    → continue Quicksort
```

`depth_limit = 2 × floor(log₂(n))`

This gives quicksort's average-case speed, heap sort's worst-case guarantee, and insertion sort's efficiency on tiny subarrays — all in one.

---

## Counting Sort

A **non-comparison** sort for integers. Counts the occurrences of each value, then reconstructs the sorted array from the counts.

```
Input: [4, 2, 2, 8, 3, 3, 1]
Range: 1–8

Count array:
Index: [1, 2, 3, 4, 5, 6, 7, 8]
Count: [1, 2, 2, 1, 0, 0, 0, 1]

Prefix sums (positions):
       [1, 3, 5, 6, 6, 6, 6, 7]

Place each element → [1, 2, 2, 3, 3, 4, 8]
```

```python
def counting_sort(arr, max_val):
    count = [0] * (max_val + 1)
    for x in arr:
        count[x] += 1
    result = []
    for val, freq in enumerate(count):
        result.extend([val] * freq)
    return result
```

**O(n + k)** where k = range of values. Impractical when k >> n (e.g., sorting 10 integers with values up to 10⁹).

---

## Radix Sort

Sorts integers **digit by digit**, from least significant to most significant (LSD) or vice versa (MSD), using a stable sort (typically counting sort) at each digit position.

```
Input: [170, 45, 75, 90, 802, 24, 2, 66]

Sort by 1s digit:  [170, 90, 802,  2, 24, 45, 75, 66]
Sort by 10s digit: [802,  2, 24, 45, 66, 170, 75, 90]
Sort by 100s digit:[  2, 24, 45, 66, 75,  90, 170, 802]
```

**O(nk)** where k = number of digits. For fixed-width integers (32-bit, 64-bit), k is constant, making it effectively **O(n)**.

**LSD Radix Sort** — stable, processes from rightmost digit. Standard for fixed-width integers.  
**MSD Radix Sort** — starts from the leftmost digit; can be parallelized; needed for variable-length strings.

---

## Bucket Sort

Distributes elements into **k buckets** spanning sub-ranges, sorts each bucket (typically with insertion sort), then concatenates.

```
Input (floats in [0,1)): [0.78, 0.17, 0.39, 0.26, 0.72, 0.94, 0.21, 0.12]
Buckets (size 0.1 each):
  [0.0–0.2): [0.17, 0.12]  → sort → [0.12, 0.17]
  [0.2–0.4): [0.39, 0.26, 0.21] → sort → [0.21, 0.26, 0.39]
  [0.7–0.8): [0.78, 0.72]  → sort → [0.72, 0.78]
  [0.9–1.0): [0.94]

Concatenate → [0.12, 0.17, 0.21, 0.26, 0.39, 0.72, 0.78, 0.94]
```

**O(n + k)** average when input is **uniformly distributed**. Degrades to O(n²) if all elements fall into one bucket.

---

## Cycle Sort

Minimizes the number of **writes to memory** — performs at most n-1 writes. Ideal when write operations are expensive (flash memory, EEPROM).

The algorithm decomposes the permutation into cycles and rotates each cycle into its sorted position with the minimum writes.

```
Array: [3, 1, 2]
Cycle: 3→position 2, 2→position 1, 1→position 0
Writes: only 2 writes to sort 3 elements
```

O(n²) comparisons but O(n) writes in the worst case.

---

## Cocktail Shaker Sort (Bidirectional Bubble Sort)

An improvement on bubble sort. Each pass alternates direction — forward (bubbling the max to the right) then backward (bubbling the min to the left). Reduces **turtles** (small values far to the right that bubble sort moves very slowly).

```
→ pass: [2, 3, 4, 5, 1] → [2, 3, 4, 1, 5]
← pass: [2, 3, 4, 1, 5] → [1, 2, 3, 4, 5]
```

Same asymptotic complexity as bubble sort, but faster on inputs with turtles.

---

## Comb Sort

Improves bubble sort by using a **gap > 1** initially (like shell sort), shrinking by a factor of ~1.3 each pass until gap = 1. Eliminates turtles efficiently.

```
gap = floor(n / 1.3) repeatedly until gap = 1
Compare arr[i] with arr[i + gap], swap if needed
```

Typical performance O(n² / 2^p) for p passes — much faster than bubble sort in practice.

---

## Gnome Sort

Perhaps the **simplest sorting algorithm** to implement. Moves forward when elements are in order, swaps and steps back when they're not — like a garden gnome sorting flower pots.

```python
def gnome_sort(arr):
    i = 0
    while i < len(arr):
        if i == 0 or arr[i] >= arr[i - 1]:
            i += 1
        else:
            arr[i], arr[i - 1] = arr[i - 1], arr[i]
            i -= 1
```

Equivalent to insertion sort but expressed differently. Useful only for education.

---

## Pigeonhole Sort

Like counting sort but physically places elements into **pigeonhole buckets**, one bucket per possible value. Only practical when the range of values n equals or is close to the number of elements.

```
Input range [0..9], elements: [8, 3, 1, 5, 3]
Holes: [_, 1, _, 3, 3, 5, _, _, 8, _]
Output: [1, 3, 3, 5, 8]
```

O(n + k). Used in scenarios with very small value ranges.

---

## Strand Sort

Repeatedly pulls out **sorted subsequences** (strands) from the input and merges them into the output. Natural O(n) on already-sorted input.

```
Input: [3, 1, 4, 2, 5]
Strand 1: pull [3, 4, 5]  → remaining: [1, 2]
Strand 2: pull [1, 2]
Merge [3,4,5] with [1,2] → [1, 2, 3, 4, 5]
```

---

## Patience Sort

Inspired by the **patience card game**. Deals cards into piles such that each pile's top card is greater than the card being placed — the number of piles equals the **Longest Increasing Subsequence (LIS)** length.

```
Input: [3, 1, 4, 1, 5, 9, 2, 6]

Piles:
[3]     → place 3
[1][4]  → place 1 (new pile), place 4 on 3? No → new pile
[1][3][4]... (etc.)

Number of piles = LIS length
```

The merge phase uses a k-way merge with a min-heap. Total: O(n log n).  
Used in practice as the basis for **computing LIS in O(n log n)**.

---

## Tree Sort

Inserts all elements into a **Binary Search Tree**, then performs an **in-order traversal** to extract them in sorted order.

```python
# Build BST: insert each element → O(n log n) average
# Inorder traversal → O(n)
# Total: O(n log n) average, O(n²) with unbalanced tree
```

Becomes O(n log n) guaranteed with a **self-balancing BST** (AVL, Red-Black). Not commonly used in practice — the overhead of pointer-based nodes makes it slower than array-based sorts.

---

## External Sort (Merge Sort on Disk)

When data is **too large for RAM**, the standard approach is:

```
Phase 1 — Create sorted runs:
  Load chunk into RAM → sort in memory → write sorted run to disk
  Repeat until all data is in sorted runs on disk

Phase 2 — K-way merge:
  Open all runs simultaneously
  Use a min-heap of size k to merge: always output the smallest current element
  Write merged output to disk
```

Used by: databases (ORDER BY on huge tables), Hadoop MapReduce (shuffle phase), file system utilities.

---

## Parallel Sorting

### Bitonic Sort
A sorting network designed for **parallel hardware**. Sorts by building bitonic sequences (first ascending, then descending) and merging them.  
O(log²n) parallel steps — used in GPU sorting (CUDA).

### Sample Sort
A parallel generalization of quicksort. Samples elements to choose splitters that divide data evenly among processors, then each processor sorts its partition independently.

### Merge Sort on GPU
Odd-even merge sort, bitonic merge, and radix sort on 32-bit keys all map efficiently to GPU SIMD operations. Libraries: **CUB** (CUDA), **Thrust**, **rocPRIM**.

---

## How to Choose a Sorting Algorithm

```
General-purpose, unknown input?
  └── Use Timsort (Python) or Introsort (C++) — they're hybrids that win in practice.

Need guaranteed O(n log n) worst-case + in-place?
  └── Heap Sort.

Sorting a linked list?
  └── Merge Sort (no random access needed; O(1) extra space for iterative version).

Many duplicate keys?
  └── 3-Way Quicksort (Dutch National Flag).

Sorting small integers in a known range [0..k]?
  └── Counting Sort if k is small.
  └── Radix Sort if k is large but elements have fixed-width representation.

Floating-point numbers uniformly distributed in [0,1)?
  └── Bucket Sort.

Nearly sorted data?
  └── Insertion Sort (O(n) on nearly sorted) or Timsort.

Memory writes are expensive?
  └── Cycle Sort (minimizes writes) or Selection Sort.

Parallel / GPU environment?
  └── Bitonic Sort or Radix Sort.

External (disk-based) data?
  └── External Merge Sort.

Need stable sort + O(n log n)?
  └── Merge Sort or Timsort.
```

---

## Sorting in Practice — Language Defaults

| Language | Algorithm | Stable |
|---|---|---|
| Python | Timsort | ✅ |
| Java (primitives) | Dual-pivot Quicksort | ❌ |
| Java (objects) | Timsort | ✅ |
| C++ `std::sort` | Introsort | ❌ |
| C++ `std::stable_sort` | Merge Sort | ✅ |
| JavaScript V8 | Timsort | ✅ |
| Rust | Pattern-defeating Quicksort (pdqsort) | ❌ |
| Go | Pattern-defeating Quicksort (pdqsort) | ❌ |
| Swift | Introsort | ❌ |

---

## Quick Reference Card

```
SIMPLE (O(n²))                EFFICIENT (O(n log n))
  Bubble     stable  in-place   Merge      stable  O(n) space
  Selection  —       in-place   Quick      —       O(log n) stack
  Insertion  stable  in-place   Heap       —       in-place
  Gnome      stable  in-place   Shell      —       in-place
  Cocktail   stable  in-place   Timsort    stable  O(n) space
  Comb       —       in-place   Introsort  —       O(log n) stack
  Cycle      —       in-place   Patience   —       O(n) space

LINEAR (non-comparison)        HYBRID / PRODUCTION
  Counting   O(n+k)  stable     Timsort    Python, Java objects
  Radix      O(nk)   stable     Introsort  C++ std::sort
  Bucket     O(n+k)  stable     pdqsort    Rust, Go
  Pigeonhole O(n+k)  stable     Dual-pivot Java primitives
```
