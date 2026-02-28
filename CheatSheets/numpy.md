# NumPy Complete Reference Guide
> From Zero to Expert — Everything You Need to Master NumPy

---

## 1. What is NumPy & Why Use It

NumPy (Numerical Python) is the foundation of the entire Python scientific computing ecosystem. It provides an N-dimensional array object (`ndarray`) and a large library of mathematical functions that operate on arrays **very fast**.

### NumPy vs Plain Python

```python
import numpy as np
import time

n = 10_000_000

# Python list
lst = list(range(n))
t0 = time.perf_counter()
result = [x * 2 for x in lst]
print(f"Python list: {time.perf_counter() - t0:.3f}s")   # ~0.8s

# NumPy array
arr = np.arange(n)
t0 = time.perf_counter()
result = arr * 2
print(f"NumPy array: {time.perf_counter() - t0:.3f}s")   # ~0.01s
# ~80x faster!
```

### Why NumPy is Fast

- **Contiguous memory:** All elements stored in a single block of memory (C arrays), not scattered Python objects
- **Vectorized operations:** Operations apply to the whole array at the C level — no Python loop overhead
- **SIMD instructions:** Modern CPUs process multiple numbers in one instruction; NumPy exploits this
- **Optimized BLAS/LAPACK:** Linear algebra uses Intel MKL or OpenBLAS under the hood

### The Ecosystem Built on NumPy

```
NumPy (n-dimensional arrays)
  ├── Pandas     (labeled DataFrames built on NumPy arrays)
  ├── SciPy      (scientific algorithms: optimization, signal processing, etc.)
  ├── Matplotlib (plotting — takes NumPy arrays)
  ├── Scikit-learn (ML algorithms — all data as NumPy arrays)
  ├── TensorFlow / PyTorch (deep learning — GPU arrays inspired by NumPy)
  └── OpenCV     (computer vision — images as NumPy arrays)
```

---

## 2. Installation & Setup

```bash
pip install numpy
pip install numpy scipy matplotlib pandas   # Common scientific stack

# With conda
conda install numpy
conda install numpy scipy matplotlib pandas

# Check version
python -c "import numpy; print(numpy.__version__)"
```

```python
# Standard import alias — use this everywhere
import numpy as np

# Check build info (shows BLAS/LAPACK backend)
np.__version__           # e.g. "1.26.4"
np.show_config()         # Show BLAS/LAPACK configuration
np.__config__.blas_opt_info   # BLAS details
```

---

## 3. Creating Arrays

### From Python Data

```python
import numpy as np

# From a list
a = np.array([1, 2, 3, 4, 5])            # 1D array, dtype=int64
a = np.array([1.0, 2.0, 3.0])            # dtype=float64
a = np.array([1, 2, 3], dtype=np.float32)  # Explicit dtype

# From nested lists → 2D array
m = np.array([[1, 2, 3],
              [4, 5, 6]])                 # shape (2, 3)

# 3D array
t = np.array([[[1, 2], [3, 4]],
              [[5, 6], [7, 8]]])          # shape (2, 2, 2)

# From a tuple
a = np.array((1, 2, 3))

# Mixed nesting must be consistent (no ragged arrays)
# np.array([[1,2], [3]])   # DeprecationWarning in newer NumPy
```

### Zeros, Ones & Constants

```python
np.zeros(5)                   # [0. 0. 0. 0. 0.]
np.zeros((3, 4))              # 3×4 matrix of zeros (float64)
np.zeros((2, 3, 4))           # 3D array

np.ones(5)                    # [1. 1. 1. 1. 1.]
np.ones((3, 4), dtype=int)    # Integer ones

np.full(5, 7)                 # [7 7 7 7 7]
np.full((3, 3), np.pi)        # 3×3 filled with π
np.full_like(a, 99)           # Same shape/dtype as `a`, filled with 99

np.empty((3, 4))              # Uninitialized (fast — garbage values)
np.empty_like(a)              # Same shape/dtype as `a`, uninitialized

np.zeros_like(a)              # Same shape/dtype as `a`, all zeros
np.ones_like(a)               # Same shape/dtype as `a`, all ones
```

### Ranges & Sequences

```python
# arange — like Python range() but returns ndarray
np.arange(5)                  # [0 1 2 3 4]
np.arange(2, 10)              # [2 3 4 5 6 7 8 9]
np.arange(0, 1, 0.1)          # [0.  0.1 0.2 ... 0.9]
np.arange(10, 0, -2)          # [10  8  6  4  2]

# linspace — n evenly spaced points between start and stop (inclusive)
np.linspace(0, 1, 5)          # [0.   0.25 0.5  0.75 1.  ]
np.linspace(0, 1, 5, endpoint=False)  # [0.  0.2 0.4 0.6 0.8]
np.linspace(0, 1, 5, retstep=True)   # (array, step_size)

# logspace — n points on a logarithmic scale
np.logspace(0, 3, 4)          # [1. 10. 100. 1000.]  (base 10)
np.logspace(0, 8, 9, base=2)  # Powers of 2

# geomspace — geometric sequence
np.geomspace(1, 1000, 4)      # [   1.   10.  100. 1000.]
```

### Identity, Diagonal & Triangular

```python
np.eye(3)                     # 3×3 identity matrix
np.eye(3, 4)                  # 3×4 with 1s on main diagonal
np.eye(4, k=1)                # 4×4 with 1s on superdiagonal (k=1)
np.eye(4, k=-1)               # 4×4 with 1s on subdiagonal

np.identity(3)                # 3×3 identity (square only)

np.diag([1, 2, 3])            # Diagonal matrix from 1D array
np.diag(m)                    # Extract diagonal from 2D array
np.diag(m, k=1)               # Extract superdiagonal

np.diagflat([[1,2],[3,4]])     # Flatten and make diagonal matrix

np.tri(3)                     # Lower triangular all-ones matrix
np.tril(m)                    # Lower triangle of m (zero out upper)
np.triu(m)                    # Upper triangle of m (zero out lower)
np.tril(m, k=-1)              # Strictly lower triangular
```

### Meshgrids & Grid Arrays

```python
# meshgrid — create coordinate grids (essential for plotting & vectorized 2D functions)
x = np.linspace(-2, 2, 5)
y = np.linspace(-1, 1, 3)
X, Y = np.meshgrid(x, y)     # X shape (3,5), Y shape (3,5)
Z = np.sqrt(X**2 + Y**2)     # Distance from origin at each point

# mgrid — open mesh (returns indexing arrays)
grid = np.mgrid[0:3, 0:4]    # shape (2, 3, 4)
rows, cols = np.mgrid[0:3, 0:4]

# ogrid — open mesh (minimal representation)
r, c = np.ogrid[0:3, 0:4]    # r shape (3,1), c shape (1,4)
# Broadcast together to get full grid
dist = np.sqrt(r**2 + c**2)   # shape (3, 4)

# indices — like meshgrid for integer indices
np.indices((3, 4))            # shape (2, 3, 4)
```

### Copying & Converting

```python
# Convert Python types to NumPy
np.asarray([1, 2, 3])         # No copy if already ndarray
np.asarray(a, dtype=float)
np.ascontiguousarray(a)       # Ensure C-contiguous memory layout
np.asfortranarray(a)          # Ensure Fortran-contiguous layout

# Explicit copy
b = a.copy()                  # Deep copy — modifications to b don't affect a
b = np.copy(a)                # Same

# View vs copy — critical to understand!
b = a.view()                  # Shares data with a — same memory!
b = a[:]                      # Also a view!
b = a[::2]                    # Slices are views
b = a[[0,1,2]]                # Fancy indexing returns a COPY

# Check if array owns its data
a.flags["OWNDATA"]            # True if owns data
a.base is None                # True if owns data (no base array)
```

---

## 4. Array Attributes & Inspection

```python
a = np.array([[1, 2, 3],
              [4, 5, 6]], dtype=np.float32)

# Shape & dimensions
a.shape          # (2, 3)
a.ndim           # 2  (number of dimensions)
a.size           # 6  (total number of elements)
len(a)           # 2  (length of first dimension only)

# Data type
a.dtype          # dtype('float32')
a.dtype.name     # 'float32'
a.itemsize       # 4  (bytes per element for float32)
a.nbytes         # 24 (total bytes: size * itemsize)

# Memory layout
a.flags          # C_CONTIGUOUS, F_CONTIGUOUS, OWNDATA, WRITEABLE, etc.
a.strides        # (12, 4) — bytes to step in each dimension
a.data           # memoryview of raw data buffer

# Type checks
np.issubdtype(a.dtype, np.floating)   # True
np.issubdtype(a.dtype, np.integer)    # False
np.issubdtype(a.dtype, np.number)     # True

# Pretty printing
print(a)                    # Formatted array
np.set_printoptions(
    precision=4,            # Decimal places for floats
    suppress=True,          # Suppress scientific notation for small numbers
    linewidth=120,          # Characters before wrapping
    threshold=100,          # Max elements before truncating with ...
    edgeitems=3,            # Items at edges when truncating
    formatter={"float": lambda x: f"{x:.2f}"}  # Custom formatter
)
np.set_printoptions()       # Reset to defaults (pass no args)

# Describe the array
a.min(), a.max(), a.mean()
np.info(np.zeros)           # Documentation for a function
```

---

## 5. Data Types (dtype)

Understanding dtypes is critical for performance and correctness.

### NumPy Type Hierarchy

```
np.generic
  ├── np.bool_
  ├── np.number
  │     ├── np.integer
  │     │     ├── np.signedinteger:   int8, int16, int32, int64
  │     │     └── np.unsignedinteger: uint8, uint16, uint32, uint64
  │     └── np.floating
  │           ├── float16, float32, float64, float128 (platform)
  │           └── np.complexfloating: complex64, complex128
  ├── np.character: bytes_, str_
  └── np.object_
```

### Type Reference

| NumPy Type | Alias | Bytes | Range / Notes |
|------------|-------|-------|---------------|
| `np.bool_` | `bool` | 1 | `True` / `False` |
| `np.int8` | | 1 | -128 to 127 |
| `np.int16` | | 2 | -32,768 to 32,767 |
| `np.int32` | `int_` (32-bit) | 4 | ~±2.1 billion |
| `np.int64` | `int_` (64-bit) | 8 | ~±9.2 × 10¹⁸ |
| `np.uint8` | | 1 | 0 to 255 (images!) |
| `np.uint16` | | 2 | 0 to 65,535 |
| `np.uint32` | | 4 | 0 to ~4.3 billion |
| `np.uint64` | | 8 | 0 to ~1.8 × 10¹⁹ |
| `np.float16` | `half` | 2 | ~3 decimal digits (ML weights) |
| `np.float32` | `single` | 4 | ~7 decimal digits (most ML) |
| `np.float64` | `double`, `float_` | 8 | ~15 decimal digits (default) |
| `np.complex64` | | 8 | Two float32 |
| `np.complex128` | `complex_` | 16 | Two float64 (default complex) |
| `np.str_` | `unicode_` | varies | Fixed-length Unicode |
| `np.bytes_` | `string_` | varies | Fixed-length bytes |
| `np.object_` | | 8 | Python objects (avoid!) |

### Creating & Converting dtypes

```python
# Specify dtype on creation
a = np.array([1, 2, 3], dtype=np.float32)
a = np.array([1, 2, 3], dtype="f4")      # Short string code
a = np.zeros(5, dtype=np.int16)

# Convert (cast) dtype
b = a.astype(np.int32)              # New array with different dtype
b = a.astype("float64")
b = a.astype(np.float32, copy=False)  # Avoid copy if already that type

# Safe casting check
np.can_cast(np.float32, np.float64)   # True — no precision loss
np.can_cast(np.float64, np.float32)   # False — precision loss possible
np.can_cast(np.float64, np.float32, casting="same_kind")  # True

# Type limits
np.iinfo(np.int32).max         # 2147483647
np.iinfo(np.uint8).min         # 0
np.finfo(np.float32).eps       # 1.1920929e-07 (machine epsilon)
np.finfo(np.float64).max       # 1.7976931348623157e+308
np.finfo(np.float64).resolution  # 1e-15

# Result type of an operation
np.result_type(np.int32, np.float32)   # dtype('float64')
np.promote_types(np.int32, np.float32) # dtype('float64')
np.min_scalar_type(256)                # dtype('uint16') — smallest type that fits

# String dtype codes
# 'f4' = float32, 'f8' = float64
# 'i2' = int16,   'i4' = int32, 'i8' = int64
# 'u1' = uint8,   'u4' = uint32
# 'b'  = bool,    'c8' = complex64
# 'U10' = unicode string, max 10 chars
# 'S5'  = byte string, max 5 bytes
```

---

## 6. Indexing & Slicing

### 1D Indexing

```python
a = np.array([10, 20, 30, 40, 50])

a[0]          # 10    (first)
a[-1]         # 50    (last)
a[-2]         # 40    (second to last)

# Slicing: [start:stop:step]  (stop is exclusive)
a[1:4]        # [20 30 40]
a[:3]         # [10 20 30]
a[2:]         # [30 40 50]
a[::2]        # [10 30 50]  (every other)
a[::-1]       # [50 40 30 20 10]  (reversed)
a[1:4:2]      # [20 40]
```

### 2D Indexing

```python
m = np.array([[1,  2,  3,  4],
              [5,  6,  7,  8],
              [9, 10, 11, 12]])

# Single element: m[row, col]
m[0, 0]       # 1
m[1, 2]       # 7
m[-1, -1]     # 12

# Row selection
m[0]          # [1 2 3 4]      (first row — 1D array)
m[0, :]       # [1 2 3 4]      (same, explicit)
m[[0, 2]]     # rows 0 and 2   (fancy indexing)

# Column selection
m[:, 0]       # [1 5 9]        (first column — 1D array)
m[:, 1:3]     # columns 1 and 2

# Sub-matrix (slice rows AND columns)
m[0:2, 1:3]   # [[2,3],[6,7]]  (rows 0-1, cols 1-2)
m[::2, ::2]   # [[1,3],[9,11]] (every other row and col)

# All rows, specific col
m[:, 2]       # [3 7 11]

# Assign via indexing
m[0, 0]  = 99
m[:, 0]  = [10, 50, 90]       # Set entire column
m[1, :]  = 0                   # Set entire row to 0
m[0:2, 1:3] = [[20,30],[60,70]]
```

### N-Dimensional Indexing

```python
t = np.zeros((2, 3, 4))      # shape (2, 3, 4)

t[0]           # shape (3, 4) — first 2D slice
t[0, 1]        # shape (4,)   — first row of first slice
t[0, 1, 2]     # scalar       — element [0][1][2]
t[0, :, :]     # shape (3, 4) — same as t[0]
t[:, :, 0]     # shape (2, 3) — first column of every slice

# Ellipsis (...) — fill in all remaining dimensions
t[0, ...]      # shape (3, 4) — equivalent to t[0, :, :]
t[..., 0]      # shape (2, 3) — last dim = 0
t[0, ..., 0]   # shape (3,)
```

### Views vs Copies — Critical!

```python
a = np.arange(12).reshape(3, 4)

# VIEWS — same underlying data, modifications affect original
b = a[:]           # View
b = a[1:3]         # Slice → view
b = a.view()       # Explicit view
b = a.T            # Transpose → view

# COPIES — independent data
b = a.copy()       # Explicit copy
b = a[[0, 1]]      # Fancy indexing → copy
b = a[a > 5]       # Boolean indexing → copy
b = np.array(a)    # Always a copy

# Verify
b = a[1:3]
b[0, 0] = 999
print(a[1, 0])     # 999 — a was modified! It's a view.

b = a[[0, 1]]
b[0, 0] = 999
print(a[0, 0])     # 0 — a unchanged. It's a copy.

# Check
b.base is a        # True → b is a view of a
b.flags.owndata    # False → b doesn't own data (it's a view)
```

---

## 7. Reshaping & Manipulation

### Reshaping

```python
a = np.arange(12)   # [0 1 2 3 4 5 6 7 8 9 10 11]

# reshape — returns a view when possible
a.reshape(3, 4)           # shape (3, 4) — row-major (C order)
a.reshape(3, 4, order="F")  # Fortran order (column-major)
a.reshape(2, 3, 2)        # 3D: shape (2, 3, 2)
a.reshape(3, -1)          # -1 = infer this dimension: (3, 4)
a.reshape(-1, 3)          # (4, 3)
a.reshape(-1)             # Flatten to 1D

# np.reshape (same as method form)
np.reshape(a, (3, 4))

# Flatten — always returns a copy
a2d = a.reshape(3, 4)
a2d.flatten()             # [0 1 2 3 4 5 6 7 8 9 10 11] — copy
a2d.flatten(order="F")    # Fortran order (column-major)

# ravel — returns view when possible
a2d.ravel()               # [0 1 2 3 4 5 6 7 8 9 10 11] — view
a2d.ravel(order="F")      # Column-major order
```

### Adding & Removing Dimensions

```python
a = np.array([1, 2, 3])   # shape (3,)

# Add dimensions
a[:, np.newaxis]          # shape (3, 1) — column vector
a[np.newaxis, :]          # shape (1, 3) — row vector
a.reshape(1, 3)           # shape (1, 3)
np.expand_dims(a, axis=0) # shape (1, 3)
np.expand_dims(a, axis=1) # shape (3, 1)
np.expand_dims(a, axis=(0,2))  # shape (1, 3, 1)

# Remove size-1 dimensions
a = np.array([[[1, 2, 3]]])   # shape (1, 1, 3)
a.squeeze()                    # shape (3,)
a.squeeze(axis=0)              # shape (1, 3) — only remove axis 0
np.squeeze(a)
```

### Transposing & Swapping Axes

```python
m = np.arange(12).reshape(3, 4)   # shape (3, 4)

m.T                    # Transpose: shape (4, 3) — view
m.transpose()          # Same
np.transpose(m)        # Same

# For higher dimensions
t = np.arange(24).reshape(2, 3, 4)   # shape (2, 3, 4)
t.T                    # Reverses all axes: shape (4, 3, 2)
np.transpose(t, (2, 0, 1))  # Permute axes: shape (4, 2, 3)

np.moveaxis(t, 0, -1)  # Move axis 0 to last position: shape (3, 4, 2)
np.moveaxis(t, [0,1], [-1,-2])  # Move multiple axes

np.swapaxes(t, 0, 2)   # Swap axes 0 and 2: shape (4, 3, 2)
np.rollaxis(t, 2)      # Roll axis 2 backwards: shape (4, 2, 3)
```

### Joining Arrays

```python
a = np.array([[1, 2], [3, 4]])    # shape (2, 2)
b = np.array([[5, 6], [7, 8]])    # shape (2, 2)

# concatenate — join along existing axis
np.concatenate([a, b], axis=0)    # Stack rows: shape (4, 2)
np.concatenate([a, b], axis=1)    # Stack cols: shape (2, 4)
np.concatenate([a, b], axis=None) # Flatten then join: shape (8,)

# vstack — vertical stack (along axis 0) — arrays must match cols
np.vstack([a, b])                 # shape (4, 2)
np.row_stack([a, b])              # Same

# hstack — horizontal stack (along axis 1) — arrays must match rows
np.hstack([a, b])                 # shape (2, 4)
np.column_stack([a, b])           # Like hstack but handles 1D → 2D columns

# stack — create NEW axis
np.stack([a, b], axis=0)          # shape (2, 2, 2) — new first axis
np.stack([a, b], axis=1)          # shape (2, 2, 2) — new second axis
np.stack([a, b], axis=2)          # shape (2, 2, 2) — new last axis

# dstack — stack along third axis (depth)
np.dstack([a, b])                 # shape (2, 2, 2)

# 1D special cases
x = np.array([1, 2])
y = np.array([3, 4])
np.hstack([x, y])                 # [1 2 3 4]
np.vstack([x, y])                 # [[1,2],[3,4]]
np.column_stack([x, y])           # [[1,3],[2,4]] — as columns
```

### Splitting Arrays

```python
a = np.arange(12).reshape(3, 4)

# split — into equal parts
np.split(a, 3, axis=0)            # 3 arrays of shape (1, 4)
np.split(a, 2, axis=1)            # 2 arrays of shape (3, 2)
np.split(a, [1, 3], axis=1)       # Split at indices 1 and 3

# vsplit / hsplit / dsplit
np.vsplit(a, 3)                   # Split along rows
np.hsplit(a, 2)                   # Split along columns
np.hsplit(a, [1, 3])              # Split at column indices 1 and 3

# array_split — allows unequal splits
np.array_split(a, 4, axis=0)     # 3 arrays → won't error if not evenly divisible
```

### Tiling & Repeating

```python
a = np.array([1, 2, 3])

np.tile(a, 3)              # [1 2 3 1 2 3 1 2 3]
np.tile(a, (2, 3))         # 2D: [[1 2 3 1 2 3 1 2 3], ...]  shape (2, 9)

np.repeat(a, 3)            # [1 1 1 2 2 2 3 3 3]
np.repeat(a, [1, 2, 3])    # [1 2 2 3 3 3]  (different repeats per element)

m = np.array([[1, 2], [3, 4]])
np.repeat(m, 2, axis=0)    # Repeat each row twice: shape (4, 2)
np.repeat(m, 2, axis=1)    # Repeat each col twice: shape (2, 4)
```

---

## 8. Math & Arithmetic Operations

### Element-wise Arithmetic

```python
a = np.array([1, 2, 3, 4])
b = np.array([10, 20, 30, 40])

# All operations are element-wise
a + b           # [11 22 33 44]
a - b           # [-9 -18 -27 -36]
a * b           # [10 40 90 160]
b / a           # [10. 10. 10. 10.]
b // a          # [10 10 10 10]   integer division
b % a           # [0 0 0 0]       modulo
a ** 2          # [1 4 9 16]      exponentiation
-a              # [-1 -2 -3 -4]   negation

# Scalar operations (broadcast scalar to full array)
a + 100         # [101 102 103 104]
a * 2.5         # [2.5 5.  7.5 10. ]
2 ** a          # [2 4 8 16]

# In-place operations (modify array, avoid copy)
a += 10
a *= 2
a **= 2

# Functional equivalents (ufuncs)
np.add(a, b)
np.subtract(a, b)
np.multiply(a, b)
np.divide(a, b)
np.floor_divide(a, b)
np.mod(a, b)
np.power(a, 2)
np.negative(a)
np.absolute(a)        # Absolute value
np.abs(a)             # Same
```

### Universal Functions (ufuncs)

Ufuncs are the core of NumPy — they operate element-wise and are implemented in C.

```python
a = np.array([0.0, np.pi/6, np.pi/4, np.pi/2, np.pi])

# Trigonometry
np.sin(a)
np.cos(a)
np.tan(a)
np.arcsin(a)          # Inverse sin
np.arccos(a)
np.arctan(a)
np.arctan2(y, x)      # 4-quadrant arctangent
np.hypot(x, y)        # sqrt(x²+y²) — Euclidean distance
np.deg2rad(180)       # π
np.rad2deg(np.pi)     # 180.0
np.unwrap(angles)     # Unwrap radian phase angles

# Hyperbolic
np.sinh(a)
np.cosh(a)
np.tanh(a)
np.arcsinh(a)
np.arccosh(a)
np.arctanh(a)

# Exponents & logarithms
np.exp(a)             # e^x
np.exp2(a)            # 2^x
np.expm1(a)           # e^x - 1 (more accurate for small x)
np.log(a)             # Natural log (base e)
np.log2(a)            # Log base 2
np.log10(a)           # Log base 10
np.log1p(a)           # log(1+x)  (more accurate for small x)

# Rounding
np.floor(a)           # Round toward -∞
np.ceil(a)            # Round toward +∞
np.trunc(a)           # Round toward 0
np.round(a, 2)        # Round to 2 decimal places
np.around(a, 2)       # Same as round
a.round(2)            # Method form
np.rint(a)            # Round to nearest integer (returns float)
np.fix(a)             # Same as trunc

# Misc math
np.sqrt(a)            # Square root
np.cbrt(a)            # Cube root
np.square(a)          # x²
np.sign(a)            # -1, 0, or 1
np.clip(a, 0.1, 0.9)  # Clamp values to [0.1, 0.9]
np.maximum(a, b)      # Element-wise max of two arrays
np.minimum(a, b)      # Element-wise min of two arrays
np.fmax(a, b)         # Like maximum but ignores NaN
np.fmin(a, b)         # Like minimum but ignores NaN
np.heaviside(a, 0.5)  # Heaviside step function
```

### Bitwise Operations

```python
a = np.array([0b1010, 0b1100], dtype=np.uint8)
b = np.array([0b1001, 0b0110], dtype=np.uint8)

np.bitwise_and(a, b)   # [0b1000, 0b0100]
np.bitwise_or(a, b)    # [0b1011, 0b1110]
np.bitwise_xor(a, b)   # [0b0011, 0b1010]
np.invert(a)           # Bitwise NOT
np.left_shift(a, 2)    # Shift left by 2 bits
np.right_shift(a, 1)   # Shift right by 1 bit
```

### Floating-Point Utilities

```python
# Special values
np.inf                 # Infinity
np.nan                 # Not a Number
np.PINF                # +inf
np.NINF                # -inf
np.NZERO               # -0.0

# Check for special values
np.isinf(a)            # Boolean array: is element ±inf?
np.isnan(a)            # Boolean array: is element NaN?
np.isfinite(a)         # Boolean array: is element finite?
np.isneginf(a)
np.isposinf(a)

# NaN-safe operations (ignore NaN in reductions)
a = np.array([1, 2, np.nan, 4])
np.nansum(a)           # 7.0   (ignores NaN)
np.nanmean(a)          # 2.333
np.nanmax(a)           # 4.0
np.nanmin(a)           # 1.0
np.nanstd(a)
np.nanvar(a)
np.nanmedian(a)
np.nanpercentile(a, 50)

# Close comparison (instead of ==, which fails for floats)
np.isclose(0.1 + 0.2, 0.3)           # True
np.allclose(a, b, rtol=1e-5, atol=1e-8)  # All elements close?
np.isclose(a, b, equal_nan=True)     # Treat NaN==NaN as True
```

---

## 9. Aggregations & Statistics

### Basic Aggregations

```python
a = np.array([[1, 2, 3],
              [4, 5, 6],
              [7, 8, 9]])

# Whole array
a.sum()              # 45
a.min()              # 1
a.max()              # 9
a.mean()             # 5.0
a.prod()             # 362880
a.std()              # Standard deviation (population)
a.var()              # Variance (population)
a.cumsum()           # Cumulative sum (flattened)
a.cumprod()          # Cumulative product

# Along an axis
a.sum(axis=0)        # [12 15 18]  — sum of each column
a.sum(axis=1)        # [6 15 24]   — sum of each row
a.sum(axis=(0,1))    # 45          — sum everything (same as no axis)

a.min(axis=0)        # [1 2 3]     — min of each column
a.max(axis=1)        # [3 6 9]     — max of each row
a.mean(axis=0)       # [4. 5. 6.]  — mean of each column
a.std(axis=1)        # Std dev of each row

# keepdims=True — preserves dimensions for broadcasting
a.sum(axis=1, keepdims=True)    # shape (3,1) instead of (3,)
a.mean(axis=0, keepdims=True)   # shape (1,3) instead of (3,)

# Normalize rows (divide each row by its sum)
a_norm = a / a.sum(axis=1, keepdims=True)

# Functional forms
np.sum(a)
np.sum(a, axis=0)
np.min(a, axis=1)
```

### Statistical Functions

```python
a = np.array([4, 1, 7, 2, 9, 3, 5, 8, 6])

np.median(a)               # 5.0
np.percentile(a, 25)       # 3.0   (25th percentile)
np.percentile(a, [25, 50, 75])   # Quartiles
np.quantile(a, 0.25)       # Same as percentile but 0-1 scale
np.quantile(a, [0.25, 0.5, 0.75])

# Descriptive statistics
np.ptp(a)                  # Peak-to-peak (max - min): 8
np.std(a, ddof=0)          # Population std (divide by n)
np.std(a, ddof=1)          # Sample std (divide by n-1)
np.var(a, ddof=1)          # Sample variance

# Weighted statistics
weights = np.ones_like(a)
np.average(a, weights=weights)   # Weighted average
np.average(a, axis=0, weights=w) # Along axis

# Correlation and covariance
x = np.array([1, 2, 3, 4])
y = np.array([2, 4, 5, 4])
np.corrcoef(x, y)          # 2×2 correlation matrix
np.cov(x, y)               # 2×2 covariance matrix
np.cov(x, y, ddof=1)       # Sample covariance (default)
np.cov(x, y, ddof=0)       # Population covariance

# Histogram
counts, bin_edges = np.histogram(a, bins=5)
counts, bin_edges = np.histogram(a, bins=[0,3,6,9])  # Custom bins
density, bin_edges = np.histogram(a, bins=5, density=True)  # Probability density

# 2D histogram
H, xedges, yedges = np.histogram2d(x, y, bins=5)

# Bincount — count occurrences of non-negative integers
np.bincount(np.array([0, 1, 1, 2, 2, 2]))   # [1 2 3]  (count of 0s, 1s, 2s)
np.bincount(np.array([0,1,2,1]), weights=[0.5,1.0,2.0,0.3])  # Weighted
```

### Index-returning Aggregations

```python
a = np.array([[3, 1, 4],
              [1, 5, 9],
              [2, 6, 5]])

# Position of min/max
np.argmin(a)              # 1  (flat index of global minimum)
np.argmax(a)              # 5  (flat index of global maximum, i.e. 9)
np.argmin(a, axis=0)      # [1 0 0]  index of min in each column
np.argmax(a, axis=1)      # [2 2 1]  index of max in each row

# Unravel flat index to multi-dim index
np.unravel_index(5, a.shape)   # (1, 2)  ← position of 9

# Indices that would sort the array
np.argsort(a, axis=1)     # Sort indices along rows
np.argsort(a, axis=0)     # Sort indices along columns
```

---

## 10. Broadcasting

Broadcasting is NumPy's mechanism for performing operations on arrays of **different shapes**. It's one of the most powerful and often misunderstood features.

### Broadcasting Rules

1. If arrays have different numbers of dimensions, prepend 1s to the **smaller** shape.
2. Arrays with size 1 along a dimension act as if they have the size of the other array.
3. If sizes differ and neither is 1 → error.

```
Shape A:  (3, 4)      compatible
Shape B:     (4,)  →  (1, 4)  →  (3, 4)
Result:   (3, 4)

Shape A:  (3, 1)      compatible
Shape B:     (4,)  →  (1, 4)
Result:   (3, 4)

Shape A:  (3, 4)      ERROR!
Shape B:  (3, 3)  →  neither matches
```

### Broadcasting Examples

```python
# Scalar broadcast (trivial)
a = np.array([[1, 2, 3],
              [4, 5, 6]])    # shape (2, 3)
a + 10                       # 10 broadcast to (2, 3)

# Row vector broadcast
row = np.array([1, 2, 3])   # shape (3,)  →  treated as (1, 3)
a + row                      # shape (2, 3) — adds row to every row of a

# Column vector broadcast
col = np.array([[10],        # shape (2, 1)
                [20]])
a + col                      # shape (2, 3) — adds col to every column

# Outer product via broadcasting
x = np.array([1, 2, 3])      # shape (3,)
y = np.array([10, 20])       # shape (2,)
x[:, np.newaxis] * y          # shape (3, 2) — outer product
# [[10 20]
#  [20 40]
#  [30 60]]

# Standardize (zero mean, unit std) using broadcasting
data = np.random.randn(1000, 10)    # 1000 samples, 10 features
mean = data.mean(axis=0)            # shape (10,)
std  = data.std(axis=0)             # shape (10,)
standardized = (data - mean) / std  # mean/std broadcast over 1000 rows

# Distance matrix between two sets of points
A = np.random.randn(5, 2)    # 5 points in 2D
B = np.random.randn(3, 2)    # 3 points in 2D
# A[:, np.newaxis] shape (5,1,2), B shape (3,2)→(1,3,2)
diff = A[:, np.newaxis] - B         # shape (5, 3, 2)
dist = np.sqrt((diff**2).sum(axis=2))  # shape (5, 3)  — all pairwise distances

# Normalize each row to unit length
norms = np.linalg.norm(data, axis=1, keepdims=True)  # shape (1000, 1)
normalized = data / norms                              # broadcast over columns
```

---

## 11. Boolean Indexing & Filtering

```python
a = np.array([3, 1, 4, 1, 5, 9, 2, 6, 5, 3])

# Create boolean mask
mask = a > 4              # [F F F F T T F T T F]
a[mask]                   # [5 9 6 5]  — elements where mask is True

# Inline (most common usage)
a[a > 4]                  # [5 9 6 5]
a[a % 2 == 0]             # [4 2 6]   — even numbers
a[a != 1]                 # Remove all 1s

# 2D boolean indexing
m = np.arange(12).reshape(3, 4)
m[m > 5]                  # 1D array of values greater than 5 — always 1D result!

# Row selection with boolean mask
row_mask = m.sum(axis=1) > 10    # Which rows sum to > 10?
m[row_mask]                       # Selected rows

# Combine conditions
a[(a > 2) & (a < 7)]      # AND:  [3 4 5 6 5 3]
a[(a < 2) | (a > 7)]      # OR:   [1 1 9]
a[~(a > 4)]               # NOT:  [3 1 4 1 2 3]
# Use & | ~ (not 'and', 'or', 'not' — those don't work on arrays!)

# np.where — conditional selection (ternary for arrays)
np.where(a > 4, a, 0)     # Replace values ≤4 with 0: [0 0 0 0 5 9 0 6 5 0]
np.where(a > 4, 1, -1)    # [−1 −1 −1 −1 1 1 −1 1 1 −1]
np.where(a > 4)            # (array([4,5,7,8]),)  — indices where condition is True

# np.nonzero / np.argwhere
np.nonzero(a > 4)          # Tuple of arrays of indices
np.argwhere(a > 4)         # 2D array of indices: [[4],[5],[7],[8]]

# Assignment via boolean index
a[a < 0] = 0               # Clip negatives to 0
a[a > 100] = 100           # Clip large values

# np.select — multiple conditions
conditions = [a < 3, a < 6, a >= 6]
choices    = [-1,    0,     1    ]
np.select(conditions, choices, default=0)
# Like a multi-case np.where

# np.extract — like a[mask] but functional
np.extract(a > 4, a)       # [5 9 6 5]

# Counting
np.count_nonzero(a > 4)    # 4
(a > 4).sum()              # Same
np.any(a > 8)              # True — is any element > 8?
np.all(a > 0)              # True — are all elements > 0?
np.any(a > 8, axis=0)      # Along axis
```

---

## 12. Fancy Indexing

Fancy indexing uses integer arrays (not slices) as indices. It **always returns a copy**.

```python
a = np.array([10, 20, 30, 40, 50])

# Index with integer array
idx = np.array([0, 2, 4])
a[idx]                     # [10 30 50]
a[[0, 2, 4]]               # Same
a[[4, 0, 2]]               # [50 10 30] — order follows index array
a[[-1, -3]]                # [50 30] — negative indices work too

# Repeated indices
a[[0, 0, 2, 2]]            # [10 10 30 30]

# 2D fancy indexing
m = np.arange(12).reshape(3, 4)

# Select rows
m[[0, 2]]                  # Rows 0 and 2: shape (2, 4)

# Select specific elements: m[row_indices, col_indices]
rows = np.array([0, 1, 2])
cols = np.array([0, 2, 1])
m[rows, cols]              # [m[0,0], m[1,2], m[2,1]] = [0, 6, 9]

# Select a submatrix (all combinations)
rows = np.array([0, 2])
cols = np.array([1, 3])
m[np.ix_(rows, cols)]      # shape (2, 2): [[m[0,1],m[0,3]],[m[2,1],m[2,3]]]
# np.ix_ creates an open mesh for broadcasting

# Fancy assignment
a[[0, 2, 4]] = 99          # a = [99 20 99 40 99]
a[[0, 0]] = [100, 200]     # When same index appears multiple times, last wins!
                            # a[0] = 200

# np.put — assignment with flat indices
np.put(a, [0, 2], [100, 300])  # Like a[[0,2]] = [100, 300] (in-place)

# take — like fancy indexing but more flexible
np.take(a, [0, 2, 4])         # Same as a[[0,2,4]]
np.take(m, [0, 2], axis=0)    # Take rows 0 and 2
np.take_along_axis(m, idx, axis=1)   # Use 2D index array

# Fancy vs boolean vs slicing
# All three select elements, but:
# - Slices: view, fast, contiguous
# - Boolean: copy, elements where True
# - Fancy:   copy, arbitrary elements
```

---

## 13. Sorting & Searching

### Sorting

```python
a = np.array([3, 1, 4, 1, 5, 9, 2, 6])

# Sort (returns new array)
np.sort(a)                    # [1 1 2 3 4 5 6 9]
np.sort(a)[::-1]              # Descending
a.sort()                      # In-place sort (modifies a!)

# Sort 2D along axis
m = np.array([[3,1,4],[1,5,9],[2,6,5]])
np.sort(m, axis=0)            # Sort each column
np.sort(m, axis=1)            # Sort each row
np.sort(m, axis=None)         # Flatten then sort

# Algorithms
np.sort(a, kind="quicksort")  # Default (introsort)
np.sort(a, kind="mergesort")  # Stable sort
np.sort(a, kind="heapsort")
np.sort(a, kind="stable")     # Guaranteed stable (mergesort/timsort)

# argsort — indices that would sort the array
idx = np.argsort(a)           # [1 3 6 0 2 4 7 5]
a[idx]                        # [1 1 2 3 4 5 6 9]  — sorted
np.argsort(a)[::-1]           # Indices for descending sort

# argsort 2D
np.argsort(m, axis=1)         # Indices to sort each row

# Sort by a key column (structured array style for regular)
# Sort rows by second column
order = np.argsort(m[:, 1])   # Indices that sort by col 1
m[order]                       # Sorted rows

# lexsort — sort by multiple keys (last key is primary)
names  = np.array(["Alice", "Bob", "Alice", "Bob"])
scores = np.array([90, 85, 95, 88])
idx = np.lexsort((names, scores))  # Sort by score first, then name
# names[idx], scores[idx]

# partition — O(n) vs O(n log n), puts kth element in sorted position
np.partition(a, 3)            # a[3] is in sorted position; left ≤ it, right ≥ it
np.argpartition(a, 3)         # Indices version
np.partition(a, -3)           # 3rd largest in correct position (k from end)

# Top-k elements (efficiently!)
k = 3
top_k_idx = np.argpartition(a, -k)[-k:]    # Indices of top k
top_k     = a[top_k_idx]                   # Values (not necessarily sorted)
top_k_sorted = np.sort(a)[-k:][::-1]       # Sorted top k
```

### Searching

```python
a = np.array([1, 3, 5, 7, 9, 11])   # Must be sorted for searchsorted!

# Binary search — where to insert to keep sorted
np.searchsorted(a, 6)          # 3  (insert before index 3)
np.searchsorted(a, 6, side="right")  # Insert after duplicates
np.searchsorted(a, [2, 6, 8]) # [1 3 4]  — batch query

# Find elements
np.where(a > 5)                # (array([3, 4, 5]),)
np.flatnonzero(a > 5)          # [3 4 5]  — flat indices
np.argwhere(a > 5)             # [[3], [4], [5]]

# Find unique values
b = np.array([3, 1, 4, 1, 5, 9, 2, 6, 5, 3])
np.unique(b)                   # [1 2 3 4 5 6 9]  — sorted unique values
vals, counts = np.unique(b, return_counts=True)   # with occurrence counts
vals, idx    = np.unique(b, return_index=True)    # with first occurrence index
vals, inv    = np.unique(b, return_inverse=True)  # reconstruct original: vals[inv] == b

# Set operations
a = np.array([1, 2, 3, 4])
b = np.array([3, 4, 5, 6])
np.union1d(a, b)               # [1 2 3 4 5 6]
np.intersect1d(a, b)           # [3 4]
np.setdiff1d(a, b)             # [1 2]  (in a but not b)
np.setxor1d(a, b)              # [1 2 5 6] (in one but not both)
np.in1d(a, b)                  # [F F T T]  (is each a in b?)
np.isin(a, b)                  # Same, newer API (handles N-D arrays)
```

---

## 14. Linear Algebra

```python
import numpy as np
from numpy import linalg as LA   # or: from numpy.linalg import ...

A = np.array([[2, 1, -1],
              [-3, -1, 2],
              [-2, 1, 2]], dtype=float)

b = np.array([8, -11, -3], dtype=float)
```

### Matrix Operations

```python
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

# Matrix multiplication
A @ B              # [[19 22],[43 50]]
np.dot(A, B)       # Same
np.matmul(A, B)    # Same (no scalar support unlike dot)

# Dot product of 1D vectors
x = np.array([1, 2, 3])
y = np.array([4, 5, 6])
np.dot(x, y)       # 32  (scalar: 1×4 + 2×5 + 3×6)
x @ y              # Same

# Outer product
np.outer(x, y)     # shape (3,3)
np.multiply.outer(x, y)  # Same

# Cross product
np.cross(x, y)     # [−3, 6, −3]

# Kronecker product
np.kron(A, B)

# Matrix power
LA.matrix_power(A, 3)   # A³ = A @ A @ A

# Element-wise vs matrix multiply
A * B              # Element-wise
A @ B              # Matrix multiply
```

### Solving Linear Systems

```python
# Solve Ax = b  (most common need)
x = LA.solve(A, b)            # x such that A @ x == b
# Much better than: x = LA.inv(A) @ b  (slower, less accurate)

# Least squares solution (over/under-determined systems)
x, residuals, rank, sv = LA.lstsq(A, b, rcond=None)

# Multiple right-hand sides
B = np.array([[8, 0], [-11, 5], [-3, 2]])
X = LA.solve(A, B)            # Solve A @ X = B simultaneously
```

### Decompositions

```python
# LU decomposition (via scipy)
from scipy.linalg import lu
P, L, U = lu(A)    # A = P @ L @ U

# QR decomposition
Q, R = LA.qr(A)    # A = Q @ R  (Q orthogonal, R upper triangular)
Q, R = LA.qr(A, mode="reduced")   # Economy QR

# Singular Value Decomposition
U, s, Vt = LA.svd(A)           # A = U @ diag(s) @ Vt
U, s, Vt = LA.svd(A, full_matrices=False)   # Economy SVD
# Reconstruct: A ≈ U @ np.diag(s) @ Vt
# Rank-k approximation:
k = 2
A_approx = U[:,:k] @ np.diag(s[:k]) @ Vt[:k,:]

# Eigenvalue decomposition
eigenvalues, eigenvectors = LA.eig(A)       # General (complex possible)
eigenvalues, eigenvectors = LA.eigh(A)      # Symmetric/Hermitian (real, faster)
eigenvalues = LA.eigvals(A)                 # Only eigenvalues
eigenvalues = LA.eigvalsh(A)                # Only eigenvalues, symmetric

# Cholesky decomposition (A must be positive definite)
L = LA.cholesky(A)     # A = L @ L.T.conj()  (L lower triangular)
```

### Matrix Properties

```python
# Determinant
LA.det(A)              # -1.0

# Inverse
LA.inv(A)              # A⁻¹  (avoid when possible — use solve instead)
LA.pinv(A)             # Moore-Penrose pseudo-inverse (works for non-square)

# Norms
LA.norm(x)             # L2 norm of vector: √(x₁²+x₂²+...)
LA.norm(x, ord=1)      # L1 norm: sum of absolute values
LA.norm(x, ord=np.inf) # L∞ norm: max absolute value
LA.norm(x, ord=-np.inf) # min absolute value
LA.norm(A, ord="fro")  # Frobenius norm: √(sum of squared elements)
LA.norm(A, ord=1)      # Max column sum
LA.norm(A, ord=np.inf) # Max row sum
LA.norm(A, ord=2)      # Largest singular value

# Rank
LA.matrix_rank(A)      # Numerical rank

# Condition number (sensitivity to numerical errors)
LA.cond(A)             # Condition number = σ_max / σ_min
# Large condition number → matrix is nearly singular → solutions may be inaccurate

# Trace
np.trace(A)            # Sum of diagonal elements
# equals sum of eigenvalues

# Multi-dimensional dot
np.einsum("ij,jk->ik", A, B)    # Matrix multiply
np.einsum("ii->i", A)           # Diagonal
np.einsum("ii->", A)            # Trace
np.einsum("ij->ji", A)          # Transpose
np.einsum("ij,ij->", A, B)      # Frobenius inner product
np.einsum("ijk,jl->il", T, M)   # Tensor contraction
```

---

## 15. Random Number Generation

NumPy's modern interface uses `np.random.default_rng()` — always prefer this over the legacy `np.random.*` functions.

### New Interface (Recommended)

```python
rng = np.random.default_rng(seed=42)   # Reproducible
rng = np.random.default_rng()          # Random seed

# Integers
rng.integers(0, 10)              # Single integer in [0, 10)
rng.integers(0, 10, size=5)      # Array of 5 integers
rng.integers(0, 10, size=(3,4))  # 3×4 matrix

# Floats in [0, 1)
rng.random()                     # Single float
rng.random(size=5)               # Array of 5 floats
rng.random(size=(3, 4))          # 3×4 matrix

# Uniform distribution [low, high)
rng.uniform(low=-1, high=1, size=10)

# Normal (Gaussian) distribution
rng.normal(loc=0.0, scale=1.0, size=100)    # mean=0, std=1
rng.standard_normal(size=(3,4))             # N(0,1)

# Other distributions
rng.binomial(n=10, p=0.5, size=1000)
rng.poisson(lam=3, size=1000)
rng.exponential(scale=1.0, size=100)
rng.gamma(shape=2.0, scale=1.0, size=100)
rng.beta(a=2, b=5, size=100)
rng.chisquare(df=2, size=100)
rng.f(dfnum=2, dfden=5, size=100)
rng.laplace(loc=0, scale=1, size=100)
rng.logistic(loc=0, scale=1, size=100)
rng.lognormal(mean=0, sigma=1, size=100)
rng.multinomial(n=10, pvals=[0.5, 0.3, 0.2], size=5)
rng.multivariate_normal(mean=[0,0], cov=[[1,0],[0,1]], size=100)
rng.dirichlet(alpha=[1,2,3], size=10)

# Shuffle and choice
a = np.arange(10)
rng.shuffle(a)                   # In-place shuffle
rng.permutation(10)              # Return shuffled array [0..9]
rng.permutation(a)               # Return shuffled copy of a

rng.choice(10, size=5)           # Sample 5 from range(10) with replacement
rng.choice(10, size=5, replace=False)   # Without replacement
rng.choice(a, size=3, p=[0.1]*10)       # Weighted sampling
rng.choice(["a","b","c"], size=4)       # From arbitrary array
```

### Legacy Interface (Still Common)

```python
np.random.seed(42)            # Set global seed (reproducible)
np.random.rand(3, 4)          # Uniform [0,1)
np.random.randn(3, 4)         # Standard normal N(0,1)
np.random.randint(0, 10, 5)   # Random integers
np.random.shuffle(a)          # In-place shuffle
np.random.choice(a, 3)        # Sample without specification
np.random.normal(0, 1, 100)   # Normal
np.random.uniform(0, 1, 100)  # Uniform
```

### Reproducibility Patterns

```python
# Method 1: fixed seed at start
rng = np.random.default_rng(42)

# Method 2: save and restore state
state = rng.bit_generator.state
# ... generate some numbers ...
rng.bit_generator.state = state   # Reset to same state

# Method 3: spawn independent sub-generators (parallel work)
rng = np.random.default_rng(42)
child_rngs = rng.spawn(4)    # 4 independent generators for 4 threads

# Method 4: SeedSequence for advanced control
from numpy.random import SeedSequence, PCG64
ss = SeedSequence(12345)
child_seqs = ss.spawn(4)
rngs = [np.random.Generator(PCG64(s)) for s in child_seqs]
```

---

## 16. File I/O

### NumPy Binary Format

```python
# Save/load a single array
np.save("data.npy", array)             # Saves as binary .npy
array = np.load("data.npy")            # Load back

# Save/load multiple arrays
np.savez("data.npz", arr1=a, arr2=b)            # Uncompressed
np.savez_compressed("data.npz", arr1=a, arr2=b) # Compressed (smaller file)

with np.load("data.npz") as data:
    a = data["arr1"]
    b = data["arr2"]
    print(data.files)                  # List of arrays in the file
```

### Text Files

```python
# Save as text (CSV-like)
np.savetxt("data.csv", array, delimiter=",")
np.savetxt("data.csv", array, delimiter=",",
           fmt="%.4f",           # Format string
           header="x,y,z",      # Column names (commented by default)
           comments="")          # Remove the # from header

# Load from text
array = np.loadtxt("data.csv", delimiter=",")
array = np.loadtxt("data.csv", delimiter=",",
                   skiprows=1,              # Skip header row
                   usecols=(0, 2),          # Load only columns 0 and 2
                   dtype=float)

# More flexible: genfromtxt (handles missing values)
array = np.genfromtxt("data.csv", delimiter=",",
                      names=True,           # Use first row as field names
                      dtype=None,           # Infer dtype per column
                      filling_values=0,     # Fill missing with 0
                      encoding="utf-8")

# Very large files: use pandas read_csv then .values
import pandas as pd
array = pd.read_csv("huge_file.csv").values
```

### Memory-Mapped Files

For files too large to fit in RAM:

```python
# Create a memory-mapped array (stored on disk)
fp = np.memmap("bigdata.dat", dtype=np.float64, mode="w+", shape=(1000, 1000))
fp[0, :] = np.arange(1000)        # Write data to disk
del fp                              # Flush and close

# Open existing memmap
fp = np.memmap("bigdata.dat", dtype=np.float64, mode="r", shape=(1000, 1000))
subset = fp[100:200, :]            # Only loads this slice from disk

# Modes: 'r' read-only, 'r+' read-write, 'w+' create/overwrite, 'c' copy-on-write
```

---

## 17. Structured & Record Arrays

Structured arrays let each row have named fields of different types — like a database table.

```python
# Define a dtype with named fields
dt = np.dtype([
    ("name",    "U20"),      # Unicode string, max 20 chars
    ("age",     np.int32),
    ("height",  np.float64),
    ("active",  np.bool_),
])

# Create structured array
people = np.array([
    ("Alice", 30, 1.68, True),
    ("Bob",   25, 1.82, False),
    ("Carol", 35, 1.75, True),
], dtype=dt)

# Access by field name
people["name"]            # array(['Alice', 'Bob', 'Carol'])
people["age"]             # array([30, 25, 35])
people[0]                 # ('Alice', 30, 1.68, True)
people[0]["name"]         # 'Alice'

# Boolean indexing on fields
people[people["age"] > 28]        # Rows where age > 28
people[people["active"] == True]  # Active people

# Sort by field
people.sort(order="age")           # In-place sort by age
people[np.argsort(people["height"])]  # Sort by height, return copy

# Record array — allows attribute-style access
rec = np.rec.array(people)
rec.name             # Same as rec["name"]
rec.age.mean()

# Add computed column? Convert to dict of arrays (no native support)
data = {name: people[name] for name in people.dtype.names}
data["bmi"] = data["age"] / data["height"]**2   # Not in structured array

# Nested dtypes
dt = np.dtype([
    ("name", "U20"),
    ("coords", [("x", float), ("y", float)]),
])
arr = np.array([("Alice", (1.0, 2.0))], dtype=dt)
arr["coords"]["x"]   # array([1.])
```

---

## 18. Performance & Memory

### Views vs Copies

```python
# ALWAYS REMEMBER:
# Slices     → views  (no memory copy — fast)
# Fancy idx  → copies (new memory — slow for large arrays)
# Boolean    → copies

# Benchmark
import timeit
a = np.arange(1_000_000)
# Slice (view):
timeit.timeit(lambda: a[::2], number=10000)       # ~0.001s
# Fancy index (copy):
idx = np.arange(0, 1_000_000, 2)
timeit.timeit(lambda: a[idx], number=10000)       # ~0.1s  (100× slower!)
```

### Memory Layout

```python
# C-contiguous (row-major) — default in NumPy, better for row operations
a = np.zeros((1000, 1000), order="C")
a.flags["C_CONTIGUOUS"]       # True

# Fortran-contiguous (column-major) — better for column operations
a = np.zeros((1000, 1000), order="F")
a.flags["F_CONTIGUOUS"]       # True

# Operations on C-contiguous arrays are faster when iterating over last axis
row_op = a.sum(axis=1)        # Fast for C-order (iterating last axis)
col_op = a.sum(axis=0)        # Slow for C-order (cache misses)

# Transpose just changes strides, doesn't move data
aT = a.T                      # Logical transpose (view, free!)
aT.flags["C_CONTIGUOUS"]      # False! aT is F-contiguous now
np.ascontiguousarray(aT)      # Force C-contiguous (copies data)
```

### Avoiding Copies

```python
# Write into existing array with out= parameter
result = np.empty(1_000_000)
np.add(a, b, out=result)           # No temporary array allocation
np.multiply(a, 2, out=a)           # In-place multiply

# Use in-place operators
a += b      # In-place (no copy)
a *= 2      # In-place
a **= 2     # In-place

# But be careful with type promotion!
a = np.ones(5, dtype=np.int32)
a += 2.5    # Works but 2.5 is truncated to 2 (keeps int32 dtype)
a = a + 2.5 # Returns float64 (new array, may be what you want)
```

### Vectorization Strategies

```python
# BAD: Python loop over array elements
def slow_norm(arr):
    total = 0
    for x in arr:
        total += x * x
    return total ** 0.5

# GOOD: Vectorized NumPy
def fast_norm(arr):
    return np.sqrt(np.dot(arr, arr))
    # or: np.linalg.norm(arr)

# Timing comparison on 1M elements:
# slow_norm: ~500ms
# fast_norm: ~1ms  (500x faster)

# BAD: applying Python function elementwise
def python_sigmoid(x):
    import math
    return 1 / (1 + math.exp(-x))

result = [python_sigmoid(x) for x in arr]   # Slow!

# GOOD: vectorized
def numpy_sigmoid(x):
    return 1 / (1 + np.exp(-x))

result = numpy_sigmoid(arr)   # Fast! exp() is a ufunc

# np.vectorize — use only as last resort!
# It's just a loop with syntactic sugar, NOT actually vectorized
vfunc = np.vectorize(python_func)   # Slower than a list comprehension!
# Better: rewrite the function using NumPy operations

# np.frompyfunc — similar, returns object array
ufunc = np.frompyfunc(python_func, 1, 1)  # 1 input, 1 output
```

### numexpr for Complex Expressions

```python
pip install numexpr
import numexpr as ne

# Avoids large temporary arrays
a = np.random.randn(1_000_000)
b = np.random.randn(1_000_000)

# NumPy creates temp arrays for each op
result = np.sin(a)**2 + np.cos(b)**2      # 3 temp arrays created

# numexpr: single C pass, no temps, multithreaded
result = ne.evaluate("sin(a)**2 + cos(b)**2")   # ~4x faster for large arrays
result = ne.evaluate("2*a + 3*b - c/d")
```

### Chunking Large Operations

```python
# Process large arrays in chunks to manage memory
def process_in_chunks(data, chunk_size=10_000):
    n = len(data)
    results = np.empty(n)
    for start in range(0, n, chunk_size):
        end = min(start + chunk_size, n)
        results[start:end] = expensive_op(data[start:end])
    return results

# Or with np.array_split
for chunk in np.array_split(data, 100):
    process(chunk)
```

### Profiling Memory

```python
import sys
a = np.zeros((1000, 1000), dtype=np.float64)
a.nbytes                   # 8,000,000 bytes = 8 MB
sys.getsizeof(a)           # Slightly more (Python object overhead)

# Compare dtypes
np.zeros((1000,1000), dtype=np.float64).nbytes   # 8 MB
np.zeros((1000,1000), dtype=np.float32).nbytes   # 4 MB  — use for ML!
np.zeros((1000,1000), dtype=np.float16).nbytes   # 2 MB  — only if range fits

# Memory profiling
from memory_profiler import profile

@profile
def my_function():
    a = np.zeros(1_000_000)
    b = a * 2
    return b
```

---

## 19. Practical Patterns & Recipes

### Normalizing Data

```python
data = np.random.randn(1000, 10)   # 1000 samples, 10 features

# Min-Max scaling to [0, 1]
mn = data.min(axis=0)
mx = data.max(axis=0)
normalized = (data - mn) / (mx - mn)

# Z-score standardization (zero mean, unit variance)
mean = data.mean(axis=0)
std  = data.std(axis=0)
standardized = (data - mean) / std

# L2 normalization (unit vectors)
norms = np.linalg.norm(data, axis=1, keepdims=True)
l2_normalized = data / norms

# Robust scaling (using median and IQR)
median = np.median(data, axis=0)
q75, q25 = np.percentile(data, [75, 25], axis=0)
iqr = q75 - q25
robust_scaled = (data - median) / iqr
```

### One-Hot Encoding

```python
labels = np.array([0, 2, 1, 0, 3])
n_classes = 4

# Method 1: indexing
one_hot = np.zeros((len(labels), n_classes), dtype=int)
one_hot[np.arange(len(labels)), labels] = 1

# Method 2: eye trick
one_hot = np.eye(n_classes, dtype=int)[labels]

# Decode back
np.argmax(one_hot, axis=1)     # [0 2 1 0 3]
```

### Sliding Window / Moving Average

```python
def moving_average(a, window):
    """Efficient O(n) moving average using cumsum."""
    cumsum = np.cumsum(a)
    cumsum[window:] = cumsum[window:] - cumsum[:-window]
    return cumsum[window - 1:] / window

a = np.array([1, 2, 3, 4, 5, 6, 7, 8])
moving_average(a, 3)    # [2. 3. 4. 5. 6. 7.]

# Sliding window view (strides trick)
def sliding_windows(a, window):
    shape   = (a.shape[0] - window + 1, window)
    strides = (a.strides[0], a.strides[0])
    return np.lib.stride_tricks.as_strided(a, shape=shape, strides=strides)

sliding_windows(a, 3)
# array([[1, 2, 3],
#        [2, 3, 4],
#        [3, 4, 5], ...])

# Modern: sliding_window_view (NumPy 1.20+)
from numpy.lib.stride_tricks import sliding_window_view
windows = sliding_window_view(a, window_shape=3)
```

### Working with Images

```python
# Images are just uint8 arrays of shape (H, W) or (H, W, 3) or (H, W, 4)
from PIL import Image

# Load image as NumPy array
img = np.array(Image.open("photo.jpg"))  # shape (H, W, 3), dtype uint8
img.shape          # e.g. (480, 640, 3)

# Basic operations
grayscale = img.mean(axis=2).astype(np.uint8)  # Average RGB channels
red_channel = img[:, :, 0]                      # Extract red channel
flipped = img[::-1]                              # Flip vertically
rotated = np.rot90(img)                          # Rotate 90°
cropped = img[100:300, 200:500]                 # Crop

# Normalize for neural networks
img_float = img.astype(np.float32) / 255.0     # [0, 1] range
img_norm = (img_float - np.array([0.485, 0.456, 0.406])) / \
                        np.array([0.229, 0.224, 0.225])   # ImageNet normalization

# Batch of images: shape (N, H, W, C)
batch = np.stack([img1, img2, img3], axis=0)   # (3, H, W, 3)
# PyTorch format: (N, C, H, W)
batch_torch = batch.transpose(0, 3, 1, 2)      # (3, 3, H, W)

# Save
Image.fromarray(grayscale).save("gray.jpg")
```

### Vectorized String Operations with `np.char`

```python
names = np.array(["alice", "BOB", "Carol"])
np.char.upper(names)         # ['ALICE' 'BOB' 'CAROL']
np.char.lower(names)         # ['alice' 'bob' 'carol']
np.char.capitalize(names)    # ['Alice' 'Bob' 'Carol']
np.char.strip(names)         # Remove leading/trailing whitespace
np.char.replace(names, "l", "L")     # ['aLice' 'BOB' 'CaroL']
np.char.startswith(names, "a")       # [True False False]
np.char.split(names)                 # Split each string
np.char.add(names, "!")              # Append to each: ['alice!' 'BOB!' 'Carol!']
np.char.join("-", names)             # 'a-l-i-c-e', 'B-O-B', etc.
```

### Covariance & Correlation Matrix

```python
# Dataset: n_samples × n_features
data = np.random.randn(100, 5)   # 100 samples, 5 features

# Covariance matrix (5×5)
cov = np.cov(data.T)             # np.cov expects features as rows!
# or
cov = np.cov(data, rowvar=False) # Same: rowvar=False means columns are variables

# Correlation matrix (normalized covariance)
corr = np.corrcoef(data.T)
corr = np.corrcoef(data, rowvar=False)

# Principal Component Analysis (manual)
centered = data - data.mean(axis=0)
U, s, Vt = np.linalg.svd(centered, full_matrices=False)
explained_variance_ratio = s**2 / (s**2).sum()
components = Vt                    # Principal component directions
scores = centered @ Vt.T           # Project data onto components
```

### Polynomial Operations

```python
# Fit polynomial (returns coefficients, highest degree first)
x = np.array([0, 1, 2, 3, 4])
y = np.array([1, 3, 9, 22, 40])

coeffs = np.polyfit(x, y, deg=2)   # Fit quadratic: [a, b, c]
poly   = np.poly1d(coeffs)         # Create polynomial object
poly(2.5)                           # Evaluate at x=2.5
poly.roots                          # Find roots

# Evaluate polynomial at points (without poly1d)
y_fit = np.polyval(coeffs, x)

# Polynomial operations
p1 = np.poly1d([1, 2, 3])          # x² + 2x + 3
p2 = np.poly1d([2, 1])             # 2x + 1
p1 + p2                             # x² + 4x + 4
p1 * p2                             # Multiply polynomials
np.polyder(coeffs)                  # Derivative coefficients
np.polyint(coeffs)                  # Integral coefficients
```

### Efficient Aggregation Patterns

```python
# Group by and aggregate (before pandas, use this)
categories = np.array([0, 1, 0, 2, 1, 0, 2])
values     = np.array([10, 20, 30, 40, 50, 60, 70])

# Sum per category
for cat in np.unique(categories):
    mask = categories == cat
    print(f"Category {cat}: sum={values[mask].sum()}, mean={values[mask].mean():.1f}")

# More efficient with np.add.at (ufunc accumulation)
result = np.zeros(3)
np.add.at(result, categories, values)   # result[cat] += value for each pair
# result = [100, 70, 110]

# Count per category
np.bincount(categories, minlength=3)    # [3, 2, 2]
np.bincount(categories, weights=values, minlength=3)  # Sum per category
```

---

## 20. Quick Reference Card

### Array Creation Summary

| Expression | Result |
|------------|--------|
| `np.array([1,2,3])` | From list |
| `np.zeros((3,4))` | All zeros |
| `np.ones((3,4))` | All ones |
| `np.full((3,4), 7)` | Fill with value |
| `np.empty((3,4))` | Uninitialized |
| `np.eye(n)` | Identity matrix |
| `np.arange(start, stop, step)` | Integer range |
| `np.linspace(start, stop, n)` | n evenly spaced floats |
| `np.logspace(0, 3, 4)` | Log-spaced |
| `np.random.default_rng(42).random((3,4))` | Random [0,1) |
| `np.random.default_rng(42).normal(0,1,(3,4))` | Random normal |

### Indexing Summary

| Expression | Meaning |
|------------|---------|
| `a[i]` | Element at index i (1D) |
| `a[i, j]` | Element at row i, col j (2D) |
| `a[i:j]` | Slice from i to j−1 |
| `a[::2]` | Every other element |
| `a[::-1]` | Reversed |
| `a[..., 0]` | First along last axis |
| `a[[0,2,4]]` | Fancy: elements 0, 2, 4 |
| `a[a > 0]` | Boolean: positive elements |
| `np.where(cond, x, y)` | Conditional selection |
| `a[np.newaxis, :]` | Add new axis (→ row vector) |

### Shape Operations Summary

| Expression | Effect |
|------------|--------|
| `a.reshape(m, n)` | New shape (view) |
| `a.ravel()` | Flatten (view if possible) |
| `a.flatten()` | Flatten (always copy) |
| `a.T` | Transpose (view) |
| `np.expand_dims(a, 0)` | Add axis at position 0 |
| `a.squeeze()` | Remove size-1 axes |
| `np.concatenate([a,b], axis=0)` | Stack along axis |
| `np.vstack([a,b])` | Stack vertically |
| `np.hstack([a,b])` | Stack horizontally |
| `np.stack([a,b], axis=0)` | New axis stack |

### Math & Stats Summary

| Function | Description |
|----------|-------------|
| `np.sum(a, axis=n)` | Sum along axis |
| `np.mean(a, axis=n)` | Mean |
| `np.std(a, ddof=1)` | Sample std dev |
| `np.min / np.max` | Min / max |
| `np.argmin / np.argmax` | Index of min/max |
| `np.cumsum(a)` | Cumulative sum |
| `np.sort(a)` | Sorted copy |
| `np.argsort(a)` | Sort indices |
| `np.unique(a)` | Unique values |
| `np.percentile(a, q)` | Percentile |
| `np.median(a)` | Median |
| `np.corrcoef(a, b)` | Correlation matrix |
| `np.linalg.norm(a)` | Vector/matrix norm |
| `np.linalg.solve(A, b)` | Solve Ax=b |
| `np.linalg.eig(A)` | Eigendecomposition |
| `np.linalg.svd(A)` | SVD |
| `np.dot(a, b)` or `a @ b` | Dot / matrix product |
| `np.isnan(a)` | Detect NaN |
| `np.nansum(a)` | Sum, ignoring NaN |
| `np.clip(a, lo, hi)` | Clamp values |

### Broadcasting Rules Cheatsheet

```
(3, 4) op (4,)   → (3, 4)   ✓  (4,) treated as (1,4), broadcast rows
(3, 1) op (4,)   → (3, 4)   ✓  (4,) treated as (1,4)
(3, 4) op (3,)   → ERROR!   ✗  (3,) = (1,3) ≠ (3,4)
(3, 4) op (3,1)  → (3, 4)   ✓  (3,1) broadcasts cols
(1,)   op (3,4)  → (3, 4)   ✓  scalar-like broadcast
(2,3,4) op (3,4) → (2,3,4)  ✓  prepend 1: (1,3,4) → (2,3,4)
(2,3,4) op (3,1) → (2,3,4)  ✓  prepend 1: (1,3,1) → (2,3,4)
(2,3,4) op (2,4) → ERROR!   ✗  (1,2,4) ≠ (2,3,4) in axis 1
```

### Common Gotchas

```python
# ❌ Integer division truncates — use float or cast first
np.array([1, 2, 3]) / 2    # [0.5 1.  1.5]  ✓  (Python 3, float result)
np.array([1, 2, 3]) // 2   # [0 1 1]  (integer floor division)

# ❌ Float comparison — use isclose
0.1 + 0.2 == 0.3           # False!
np.isclose(0.1 + 0.2, 0.3) # True ✓

# ❌ NaN comparisons — NaN != NaN
a = np.array([1, np.nan, 3])
a[a == np.nan]             # Empty!  NaN != NaN
a[np.isnan(a)]             # [nan]   ✓

# ❌ Slices are views — mutations propagate
b = a[1:3]
b[0] = 99                  # Also changes a! Use .copy()

# ❌ Integer overflow is silent
a = np.array([127], dtype=np.int8)
a + 1                      # array([-128], dtype=int8)  ← overflow!

# ❌ axis confusion
a = np.zeros((3, 4))
a.sum(axis=0)              # (4,) — sum OVER rows → one value per column
a.sum(axis=1)              # (3,) — sum OVER cols → one value per row

# ❌ Using 'and'/'or'/'not' — use &/|/~
a[(a > 0) and (a < 10)]    # TypeError!
a[(a > 0) & (a < 10)]      # ✓

# ❌ np.vectorize is NOT fast
# It's just a syntactic loop, not true vectorization
# Rewrite using NumPy ufuncs/operations instead

# ❌ Forgetting keepdims for broadcasting
mean = a.mean(axis=1)      # shape (3,) — can't broadcast back to (3,4)
mean = a.mean(axis=1, keepdims=True)  # shape (3,1) ✓  broadcasts correctly
centered = a - mean        # ✓
```

### Performance Rules of Thumb

| Rule | Reason |
|------|--------|
| Avoid Python loops over array elements | Each iteration has Python overhead |
| Use `out=` parameter for large operations | Avoids temporary array allocation |
| Prefer `+=`, `*=` over `a = a + b` | In-place = no copy = less memory |
| Use `float32` instead of `float64` when precision allows | 2× less memory, often 2× faster |
| Use `np.dot` / `@` for matrix multiply, not loops | BLAS-optimized (LAPACK) |
| Sort once, search with `searchsorted` | O(n log n) once, O(log n) per query |
| Use `np.save` / `.npy` for large arrays | Much faster than CSV |
| Prefer `np.linalg.solve` over `inv() @ b` | More stable, faster |
| Keep arrays C-contiguous for row-wise ops | Cache-friendly access |
| Use `rng = np.random.default_rng(seed)` | Better randomness, reproducible |
