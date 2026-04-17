# NumPy: A Complete Progressive Tutorial

---

## 1. What & Why

NumPy (Numerical Python) is the foundation of Python's scientific computing ecosystem. It provides the `ndarray` — an N-dimensional array of homogeneous data — and a large library of mathematical functions that operate on those arrays at C speed.

Why NumPy instead of Python lists? Because Python lists are arrays of pointers to Python objects. Every arithmetic operation on a list requires Python's interpreter overhead, type checking, and object allocation. NumPy arrays store data in contiguous blocks of memory as raw C types (float64, int32, etc.). Operations on arrays are implemented in compiled C, applied to the entire array in a single call. The result is often 10-100x faster, and the code is more concise.

NumPy is not a standalone data analysis tool — it is the layer everything else is built on. Pandas DataFrames are dictionaries of NumPy arrays. Scikit-learn models take NumPy arrays as input. Matplotlib plots NumPy arrays. OpenCV represents images as NumPy arrays. PyTorch and TensorFlow tensor APIs are designed around NumPy's API. If you work with numerical Python, you work with NumPy.

---

## 2. Mental Model

A NumPy array (ndarray) has three attributes that fully describe it:

```
arr = np.array([[1, 2, 3],
                [4, 5, 6]])

arr.shape  → (2, 3)     — 2 rows, 3 columns
arr.dtype  → dtype('int64')  — type of each element
arr.ndim   → 2               — number of dimensions

Memory layout (row-major / C order by default):
[1, 2, 3, 4, 5, 6]   ← contiguous block of 6 × 8 bytes
 ↑                 ↑
 arr[0,0]          arr[1,2]

Each element is accessed via:
  address = base + row * ncols * itemsize + col * itemsize
  arr[1, 2] = base + 1 * 3 * 8 + 2 * 8 = base + 40
```

The key insight: arithmetic operations like `arr * 2` are NOT Python loops — they call a single C function that processes the entire array in one pass. This is called "vectorization." Your goal when writing NumPy code is to express operations as vectorized array operations, not Python loops.

---

## 3. Progressive Examples

### Level 1: Creating Arrays and Basic Operations

```python
import numpy as np

# --- Creating arrays ---

# From Python lists
a = np.array([1, 2, 3, 4, 5])          # 1D array, dtype inferred (int64)
b = np.array([[1, 2, 3], [4, 5, 6]])    # 2D array, shape (2, 3)
c = np.array([1.0, 2, 3])               # float64 (one float promotes all)

# Specify dtype explicitly
d = np.array([1, 2, 3], dtype=np.float32)  # 32-bit float (half memory of float64)
e = np.array([1, 2, 3], dtype=np.int8)     # 8-bit int (-128 to 127)

# Built-in constructors
zeros = np.zeros((3, 4))          # 3×4 array of 0.0
ones  = np.ones((2, 2))           # 2×2 array of 1.0
full  = np.full((3, 3), 7.0)      # 3×3 array of 7.0
eye   = np.eye(4)                  # 4×4 identity matrix
empty = np.empty((2, 3))           # uninitialized (random garbage values, slightly faster)

# Ranges
arange = np.arange(0, 10, 2)      # [0, 2, 4, 6, 8] — like Python range, returns array
linspace = np.linspace(0, 1, 5)   # [0.0, 0.25, 0.5, 0.75, 1.0] — 5 evenly-spaced points
logspace = np.logspace(0, 3, 4)   # [1, 10, 100, 1000] — log-spaced

# Random arrays (reproducible with seed)
rng = np.random.default_rng(seed=42)   # modern random generator API
rand_uniform = rng.random((3, 4))      # uniform [0, 1)
rand_normal  = rng.standard_normal((3, 4))  # standard normal (mean=0, std=1)
rand_int     = rng.integers(0, 100, size=(5, 5))  # integers in [0, 100)

# --- Array attributes ---
arr = np.array([[1.0, 2.0, 3.0], [4.0, 5.0, 6.0]])
print(arr.shape)     # (2, 3)
print(arr.ndim)      # 2
print(arr.dtype)     # float64
print(arr.size)      # 6 — total number of elements
print(arr.itemsize)  # 8 — bytes per element (float64 = 8 bytes)
print(arr.nbytes)    # 48 — total bytes (6 × 8)

# --- Arithmetic — fully vectorized ---
a = np.array([1.0, 2.0, 3.0, 4.0])
b = np.array([10.0, 20.0, 30.0, 40.0])

print(a + b)         # [11, 22, 33, 44]
print(a * b)         # [10, 40, 90, 160]
print(a ** 2)        # [1, 4, 9, 16]
print(np.sqrt(a))    # [1, 1.414, 1.732, 2]
print(a > 2)         # [False, False, True, True] — boolean array

# Scalar broadcasting: scalar applied to every element
print(a * 3)         # [3, 6, 9, 12]
print(a + 100)       # [101, 102, 103, 104]
```

### Level 2: Indexing and Slicing

```python
arr = np.array([[1, 2, 3, 4],
                [5, 6, 7, 8],
                [9, 10, 11, 12]])

# Basic indexing: [row, col] — zero-indexed
print(arr[0, 0])     # 1
print(arr[1, 2])     # 7
print(arr[-1, -1])   # 12 — last row, last column

# Slicing: [row_start:row_stop, col_start:col_stop]
print(arr[0, :])     # [1, 2, 3, 4] — entire first row
print(arr[:, 1])     # [2, 6, 10] — entire second column
print(arr[0:2, 1:3]) # [[2, 3], [6, 7]] — rows 0-1, cols 1-2
print(arr[::2, :])   # rows 0 and 2 (every other row)

# KEY: numpy slices return VIEWS, not copies
row_0 = arr[0, :]    # view of row 0
row_0[0] = 99        # modifies arr too!
print(arr[0])        # [99, 2, 3, 4]

# To get an independent copy:
row_0_copy = arr[0, :].copy()
row_0_copy[0] = 0    # does NOT affect arr

# Boolean indexing
arr = np.array([1, 5, 3, 8, 2, 9, 4])
mask = arr > 4
print(mask)          # [False, True, False, True, False, True, False]
print(arr[mask])     # [5, 8, 9] — elements where mask is True

# Combined conditions
print(arr[(arr > 2) & (arr < 8)])   # [5, 3, 4] — use & not 'and'
print(arr[arr % 2 == 0])             # [8, 2, 4] — even numbers

# Fancy indexing: select with an array of indices
idx = np.array([0, 2, 4])
print(arr[idx])      # [1, 3, 2] — elements at positions 0, 2, 4

# np.where — conditional selection
result = np.where(arr > 4, arr, 0)   # keep values > 4, zero out rest
# [0, 5, 0, 8, 0, 9, 0]

result = np.where(arr > 4, "high", "low")
# ['low', 'high', 'low', 'high', 'low', 'high', 'low']
```

### Level 3: Reshaping and Broadcasting

```python
# --- Reshaping ---
arr = np.arange(12)   # [0, 1, 2, ..., 11]

# reshape — total elements must stay the same
mat = arr.reshape(3, 4)      # 3×4 matrix
mat = arr.reshape(2, 2, 3)   # 3D: 2 layers of 2×3
mat = arr.reshape(4, -1)     # -1 means "figure it out": 4×3

# flatten vs ravel
flat = mat.flatten()    # always returns a copy
flat = mat.ravel()      # returns a view if possible (faster)

# Transpose
mat = np.array([[1, 2, 3], [4, 5, 6]])  # shape (2, 3)
print(mat.T)                              # shape (3, 2) — rows become columns
print(mat.transpose())                    # same thing

# --- Broadcasting: operating on arrays of different shapes ---
# Rule: dimensions are compared from the trailing (rightmost) dimension.
# Dimensions must match OR one of them must be 1.
# Size-1 dimensions are "stretched" to match the other.

a = np.array([[1, 2, 3],   # shape (2, 3)
              [4, 5, 6]])
b = np.array([10, 20, 30]) # shape    (3,) — treated as (1, 3) then stretched to (2, 3)

result = a + b
# [[11, 22, 33],
#  [14, 25, 36]]

# Broadcasting with column vectors
col = np.array([[100], [200]])  # shape (2, 1) — stretched to (2, 3)
result = a + col
# [[101, 102, 103],
#  [204, 205, 206]]

# Classic use: normalize each column by its mean
data = np.array([[1.0, 2.0, 3.0],
                 [4.0, 5.0, 6.0],
                 [7.0, 8.0, 9.0]])

col_means = data.mean(axis=0)    # shape (3,) — mean of each column
normalized = data - col_means    # broadcasting: (3,3) - (3,) → each row minus the means

# --- Stacking arrays ---
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

np.vstack([a, b])   # [[1,2,3],[4,5,6]] — stack vertically (new rows)
np.hstack([a, b])   # [1,2,3,4,5,6]    — stack horizontally (extend columns)
np.stack([a, b])    # [[1,2,3],[4,5,6]] — create new axis
np.concatenate([a, b])   # [1,2,3,4,5,6] — along existing axis
```

### Level 4: Mathematical Operations and Linear Algebra

```python
# Aggregation functions — all support axis parameter
arr = np.array([[1.0, 2.0, 3.0],
                [4.0, 5.0, 6.0]])

np.sum(arr)          # 21.0 — sum of all elements
np.sum(arr, axis=0)  # [5., 7., 9.] — sum down rows (result has 3 elements)
np.sum(arr, axis=1)  # [6., 15.] — sum across columns (result has 2 elements)
np.mean(arr, axis=0)
np.std(arr, axis=0)
np.min(arr, axis=1)
np.max(arr)
np.argmin(arr)       # index of minimum element (in flattened array)
np.argmax(arr, axis=0)  # index of max along rows, per column

# Universal functions (ufuncs) — element-wise, C-speed
x = np.linspace(0, 2 * np.pi, 1000)
y = np.sin(x)         # sin of every element
z = np.exp(-x**2)     # Gaussian bell curve
log = np.log(np.abs(y) + 1e-8)    # avoid log(0)

# Linear algebra
A = np.array([[2.0, 1.0], [1.0, 3.0]])
b = np.array([5.0, 10.0])

# Matrix multiplication
x = np.array([1, 2, 3])
y = np.array([4, 5, 6])
print(x @ y)                      # dot product: 1*4 + 2*5 + 3*6 = 32
print(np.dot(x, y))               # same thing

A = np.random.rand(3, 4)
B = np.random.rand(4, 5)
C = A @ B                          # (3,4) @ (4,5) → (3,5) matrix multiply

# Solve linear system: Ax = b
A = np.array([[3.0, 1.0], [1.0, 2.0]])
b = np.array([9.0, 8.0])
x = np.linalg.solve(A, b)   # x = [2, 3]

# Eigenvalues and eigenvectors
eigenvalues, eigenvectors = np.linalg.eig(A)

# Matrix decompositions
U, S, Vt = np.linalg.svd(A)    # singular value decomposition
Q, R = np.linalg.qr(A)          # QR decomposition

# Inverse, determinant, rank
print(np.linalg.det(A))     # determinant
print(np.linalg.matrix_rank(A))
print(np.linalg.inv(A))     # inverse (use solve() instead for Ax=b — more stable)
```

### Level 5: Practical Vectorization Patterns

```python
import numpy as np
import time

# --- Replacing loops with vectorization ---

# Problem: compute pairwise Euclidean distances between n points
n_points = 1000
points = np.random.rand(n_points, 2)   # n_points × 2 (x, y coordinates)

# SLOW: Python loop (avoid this)
def pairwise_slow(pts):
    n = len(pts)
    dist = np.zeros((n, n))
    for i in range(n):
        for j in range(n):
            diff = pts[i] - pts[j]
            dist[i, j] = np.sqrt(diff @ diff)
    return dist

# FAST: vectorized using broadcasting
def pairwise_fast(pts):
    # pts[:, np.newaxis, :] has shape (n, 1, 2)
    # pts[np.newaxis, :, :] has shape (1, n, 2)
    # diff has shape (n, n, 2) — all pairwise differences
    diff = pts[:, np.newaxis, :] - pts[np.newaxis, :, :]
    return np.sqrt((diff**2).sum(axis=-1))   # sum over last axis (the 2 coordinates)

t0 = time.perf_counter()
d_slow = pairwise_slow(points)
print(f"Slow: {time.perf_counter() - t0:.3f}s")  # ~2-5 seconds

t0 = time.perf_counter()
d_fast = pairwise_fast(points)
print(f"Fast: {time.perf_counter() - t0:.3f}s")  # ~0.01 seconds

# --- Structured arrays for mixed data ---
dtype = np.dtype([
    ("name", "U50"),          # Unicode string, max 50 chars
    ("age", np.int32),
    ("salary", np.float64),
])
employees = np.array([
    ("Alice", 30, 95000.0),
    ("Bob", 25, 72000.0),
    ("Carol", 35, 110000.0),
], dtype=dtype)

print(employees["name"])      # ['Alice' 'Bob' 'Carol']
print(employees[employees["age"] > 28])   # filter by age

# --- Memory-mapped arrays for datasets larger than RAM ---
# Creates a file-backed array — reads only what's needed
import tempfile, os
with tempfile.NamedTemporaryFile(suffix=".dat", delete=False) as f:
    fname = f.name

# Create a 1 GB array (doesn't actually allocate 1 GB in RAM)
large = np.memmap(fname, dtype="float32", mode="w+", shape=(250_000_000,))
large[:1000] = np.arange(1000, dtype="float32")
print(large[:5])   # [0, 1, 2, 3, 4]
del large
os.unlink(fname)   # cleanup

# --- Efficient boolean operations ---
n = 10_000_000
arr = rng.standard_normal(n)

# Count elements in range
count = np.sum((arr > -1) & (arr < 1))
# Approximate: ~68% of a normal distribution within 1 std dev

# Replace outliers with clipped values
arr_clipped = np.clip(arr, -3, 3)   # cap at ±3 std dev

# Cumulative operations
cumsum = np.cumsum(arr)
cumprod = np.cumprod(np.abs(arr[:100]) + 1)  # avoid overflow
running_max = np.maximum.accumulate(arr)
```

### Level 6: Common Scientific Computing Patterns

```python
# --- Signal processing ---
t = np.linspace(0, 1, 1000)           # 1 second, 1000 samples
signal = np.sin(2 * np.pi * 50 * t) + 0.5 * np.sin(2 * np.pi * 120 * t)  # two frequencies

# FFT — frequency domain analysis
fft = np.fft.fft(signal)
freqs = np.fft.fftfreq(len(t), d=1/1000)   # frequencies in Hz
magnitude = np.abs(fft)[:len(t)//2]         # one-sided spectrum
positive_freqs = freqs[:len(t)//2]

# The peaks at 50 Hz and 120 Hz are clearly visible in magnitude

# --- Polynomial fitting ---
x = np.array([1, 2, 3, 4, 5], dtype=float)
y = np.array([2.1, 7.9, 17.8, 32.2, 50.1])   # roughly x^2 * 2

# Fit a degree-2 polynomial
coeffs = np.polyfit(x, y, deg=2)              # [a, b, c] for ax^2 + bx + c
poly = np.poly1d(coeffs)                       # callable polynomial
print(poly(3))                                 # ~17.8 — predicted value at x=3
print(coeffs)                                  # [2.01, -0.05, 0.02] ≈ 2x^2

# Evaluate at new points
x_new = np.linspace(1, 5, 100)
y_predicted = poly(x_new)

# --- Histograms and statistics ---
data = rng.standard_normal(10000)
counts, bin_edges = np.histogram(data, bins=50)
bin_centers = (bin_edges[:-1] + bin_edges[1:]) / 2   # midpoints of bins

# Percentiles and quantiles
print(np.percentile(data, [25, 50, 75]))   # Q1, median, Q3
print(np.quantile(data, 0.95))             # 95th percentile

# Correlation
x = rng.random(100)
y = x * 0.8 + rng.random(100) * 0.2      # correlated with x
corr_matrix = np.corrcoef(x, y)           # 2×2 correlation matrix
print(f"Correlation: {corr_matrix[0, 1]:.3f}")  # ~0.97
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Views vs copies — silently modifying the wrong data**

```python
original = np.array([1, 2, 3, 4, 5])

# WRONG: slicing returns a VIEW
view = original[1:4]
view[0] = 99
print(original)   # [1, 99, 3, 4, 5] — original was modified!

# CORRECT: use .copy() when you need independence
copy = original[1:4].copy()
copy[0] = 99
print(original)   # [1, 99, 3, 4, 5] — unchanged

# Fancy indexing (with arrays of indices) DOES return copies, not views:
fancy = original[[0, 2, 4]]   # copy — modifying won't affect original
```

**Mistake 2: Using Python `and`/`or` instead of `&`/`|` for arrays**

```python
arr = np.array([1, 5, 3, 8, 2])

# WRONG: 'and' compares truthiness of entire arrays (ambiguous)
# arr > 2 and arr < 7   # ValueError: ambiguous truth value

# CORRECT: element-wise operators
mask = (arr > 2) & (arr < 7)   # [False, True, True, False, False]
mask = (arr < 2) | (arr > 6)   # [True, False, False, True, False]

# Parentheses required: & has lower precedence than >
# arr > 2 & arr < 7  → arr > (2 & arr) < 7  (wrong!)
```

**Mistake 3: Reshaping without understanding memory layout**

```python
arr = np.arange(12).reshape(3, 4)
# Row-major: [0,1,2,3,4,5,6,7,8,9,10,11]

# Transposing changes the logical layout but keeps the same memory
t = arr.T          # shape (4, 3), but NOT stored contiguously in memory

# Operations on non-contiguous arrays can be slower
# Make contiguous when performance matters:
t_contiguous = np.ascontiguousarray(arr.T)

# Check:
print(arr.flags["C_CONTIGUOUS"])         # True — row-major
print(arr.T.flags["C_CONTIGUOUS"])       # False
print(t_contiguous.flags["C_CONTIGUOUS"]) # True
```

**Mistake 4: Integer overflow in accumulations**

```python
arr = np.array([1000, 2000, 3000], dtype=np.int16)  # int16 max is 32767

# WRONG: sum exceeds int16 range
print(arr.sum())    # -32536 (wrong! overflow!)

# CORRECT: specify accumulation dtype
print(arr.sum(dtype=np.int64))    # 6000 — correct
print(arr.sum(dtype=np.float64))  # 6000.0 — also correct
```

**Mistake 5: Forgetting that `np.where(condition)` is different from `np.where(cond, x, y)`**

```python
arr = np.array([1, 5, 3, 8, 2])

# np.where(condition) returns indices (tuple of arrays)
indices = np.where(arr > 3)    # (array([1, 3]),) — indices 1 and 3
print(arr[indices])             # [5, 8]

# np.where(condition, x, y) returns element-wise selection
result = np.where(arr > 3, arr, -1)   # keep high values, -1 for low
print(result)   # [-1, 5, -1, 8, -1]

# They're completely different — don't confuse them
```

---

## 5. The "Why Does This Work" Layer

### Why NumPy Vectorization Is So Much Faster

A Python `for` loop iterating over a list executes in the CPython interpreter: each iteration involves bytecode dispatch, type checking, attribute lookup on Python objects, and memory allocation for result objects. For 10 million elements, that's 10 million rounds of Python overhead.

A NumPy vectorized operation like `arr * 2` calls a single C function (a "universal function" or ufunc). That function:
1. Receives a pointer to the raw memory block
2. Iterates in C with no Python overhead
3. May use SIMD (Single Instruction, Multiple Data) CPU instructions that process 4-8 elements simultaneously
4. Returns a new array

The ratio of C-speed to Python-speed is typically 10-100x for arithmetic, and even more for operations that can leverage SIMD or GPU acceleration.

### How Broadcasting Works in Memory

When you compute `a + b` where `a` has shape `(3, 4)` and `b` has shape `(4,)`, NumPy doesn't actually allocate a temporary `(3, 4)` copy of `b`. Instead, it creates a "strided view" where the row dimension has stride 0 — reading `b` in the row direction always returns to the same position. The addition then operates on these strided views, with the CPU's SIMD instructions naturally handling the repetition.

This is why broadcasting is both memory-efficient (no copies of the broadcast dimensions) and fast (the strided access pattern is predictable for the CPU cache).

---

## 6. Quick Reference

### Array Creation

```python
np.array([1, 2, 3])          # from Python list
np.zeros((m, n))              # m×n zeros
np.ones((m, n))               # m×n ones
np.eye(n)                     # n×n identity
np.arange(start, stop, step)  # range as array
np.linspace(start, stop, n)   # n evenly-spaced
np.random.default_rng(seed)   # reproducible RNG
rng.random((m, n))            # uniform [0,1)
rng.standard_normal((m, n))   # standard normal
```

### Indexing

```python
arr[0]          # first element (1D)
arr[-1]         # last element
arr[1:5]        # slice positions 1-4
arr[::2]        # every other element
arr[0, 1]       # row 0, col 1 (2D)
arr[:, 1]       # all rows, col 1
arr[arr > 5]    # boolean indexing
arr[[0, 2, 4]]  # fancy indexing (copy)
```

### Aggregations

```python
arr.sum(axis=0)    # sum along rows → result per column
arr.sum(axis=1)    # sum along columns → result per row
arr.mean()         # global mean
arr.std()          # standard deviation
arr.min() / .max() # min / max
arr.argmin()       # index of min
np.percentile(arr, 95)
```

### Shape Operations

```python
arr.reshape(3, -1)   # reshape (-1: auto-compute)
arr.flatten()        # → 1D copy
arr.ravel()          # → 1D view if possible
arr.T                # transpose
arr[:, np.newaxis]   # add new axis → column vector
np.vstack([a, b])    # stack vertically
np.hstack([a, b])    # stack horizontally
np.concatenate([a, b], axis=0)
```

### Essential Functions

```python
np.where(cond, x, y)    # conditional selection
np.clip(arr, lo, hi)    # clamp values
np.sort(arr)            # sorted copy
np.argsort(arr)         # indices that would sort
np.unique(arr)          # unique values
np.isin(arr, values)    # membership test
np.linalg.solve(A, b)  # solve Ax=b
A @ B                   # matrix multiply
```
