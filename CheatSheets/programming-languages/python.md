# Python: A Complete Progressive Tutorial

---

## 1. What & Why

Python is a general-purpose, interpreted, dynamically typed language designed for readability. Its philosophy — "there should be one obvious way to do it" — results in code that tends to be concise, readable, and maintainable. Python is now the dominant language in data science, machine learning, scripting, and backend web development, and remains widely used in automation, scientific computing, and education.

Why Python? It has the smallest gap between "idea" and "working code" of any mainstream language. Its standard library is enormous, its package ecosystem (`pip`) is vast, and it runs on every platform. The interpreter gives you immediate feedback. You can prototype a data pipeline, an API, or an analysis in hours rather than days.

What Python is not: fast by default (CPython is slow vs compiled languages), suitable for mobile apps, or good for systems programming (no manual memory control, GIL limits true parallelism). For those use cases, use Go, Rust, or Swift.

---

## 2. Mental Model

Python's execution model is simpler than compiled languages. The interpreter reads your `.py` file line by line, converts it to bytecode, and executes that bytecode in the CPython virtual machine.

```
your_code.py
     │
     ▼
  CPython                   Everything is an object:
  interpreter               - integers are objects
     │                      - functions are objects
     ▼                      - classes are objects
  bytecode (.pyc)           - even None is an object
     │
     ▼
  Python VM
  (executes bytecode)
```

Everything in Python is a reference. Variables don't "contain" values — they are names bound to objects. This is why:

```python
a = [1, 2, 3]
b = a          # b is another name for the SAME list
b.append(4)
print(a)       # [1, 2, 3, 4] — same object!

# To copy: use list() or a[:]
b = a[:]
b.append(5)
print(a)       # [1, 2, 3, 4] — unaffected
```

---

## 3. Progressive Examples

### Level 1: Core Types and Operations

```python
# --- Numbers ---
x = 42          # int (arbitrary precision — no overflow)
y = 3.14        # float (64-bit IEEE 754 double)
z = 2 + 3j      # complex

# Arithmetic
print(10 // 3)  # 3   (floor division — always rounds toward -∞)
print(10 % 3)   # 1   (modulo)
print(2 ** 10)  # 1024 (exponentiation)
print(divmod(10, 3))  # (3, 1) — quotient AND remainder at once

# Float precision: NEVER compare floats with ==
print(0.1 + 0.2 == 0.3)   # False — floating-point representation error
print(abs(0.1 + 0.2 - 0.3) < 1e-9)  # True — use epsilon comparison
# For money: use decimal.Decimal, not float
from decimal import Decimal
print(Decimal("0.1") + Decimal("0.2"))  # Decimal('0.3') — exact

# --- Strings ---
name = "Alice"
greeting = f"Hello, {name}!"            # f-string (Python 3.6+) — preferred
greeting = "Hello, {}!".format(name)   # .format() — older style
greeting = "Hello, %s!" % name         # % formatting — oldest style

# String operations
s = "  hello, world  "
print(s.strip())           # 'hello, world'     remove whitespace
print(s.upper())           # '  HELLO, WORLD  '
print(s.replace(",", ";")) # '  hello; world  '
print(",".join(["a", "b", "c"]))  # 'a,b,c'
print("a,b,c".split(","))         # ['a', 'b', 'c']
print("hello".startswith("he"))   # True
print(len("hello"))               # 5
print("hello"[1:4])               # 'ell'  (slicing: start inclusive, end exclusive)
print("hello"[::-1])              # 'olleh' (reverse)

# f-strings support expressions and formatting
pi = 3.14159
print(f"Pi is {pi:.2f}")          # 'Pi is 3.14'
print(f"{'left':<10}|{'right':>10}")  # 'left      |     right'
print(f"{1000000:,}")              # '1,000,000'

# --- Booleans ---
# Falsy: None, 0, 0.0, "", [], {}, set(), False
# Truthy: everything else
print(bool([]))       # False
print(bool([0]))      # True  — the list exists, even though its content is "falsy"

# Short-circuit evaluation
name = None
display = name or "Unknown"   # 'Unknown' — name is falsy
count = 5
result = count > 0 and count * 10   # 50 — evaluates second operand only if needed

# Walrus operator := (Python 3.8+) — assign and use in one expression
data = [1, 2, 3, 4, 5]
if (n := len(data)) > 3:
    print(f"Long list: {n} items")  # Long list: 5 items
```

### Level 2: Collections — Lists, Dicts, Sets, Tuples

```python
# --- Lists: ordered, mutable, heterogeneous ---
nums = [3, 1, 4, 1, 5, 9, 2, 6]
nums.append(5)            # add to end: O(1)
nums.insert(0, 0)         # insert at index: O(n) — shifts everything
nums.remove(1)            # remove first occurrence of value
popped = nums.pop()       # remove and return last element: O(1)
popped = nums.pop(0)      # remove and return first element: O(n) — avoid!
nums.sort()               # in-place sort (Timsort, stable)
sorted_copy = sorted(nums)  # returns new sorted list, original unchanged
nums.reverse()
print(3 in nums)          # membership test: O(n) for lists

# List comprehensions — preferred over map/filter for readability
squares = [x**2 for x in range(10)]
evens = [x for x in range(20) if x % 2 == 0]
flat = [item for sublist in [[1,2],[3,4],[5]] for item in sublist]

# Slicing — creates a new list
nums = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
print(nums[2:5])    # [2, 3, 4]
print(nums[:3])     # [0, 1, 2]
print(nums[7:])     # [7, 8, 9]
print(nums[::2])    # [0, 2, 4, 6, 8] (every other)
print(nums[::-1])   # reversed

# --- Dicts: key-value, ordered (3.7+), mutable ---
user = {"name": "Alice", "age": 30, "active": True}
user["email"] = "alice@example.com"   # add/update
del user["active"]                    # remove
print(user.get("missing", "default")) # safe access with default
print("name" in user)                 # True — O(1)

# Iteration
for key in user:                      # iterates over keys
    print(key, user[key])
for key, value in user.items():       # key-value pairs
    print(f"{key}: {value}")

# Dict comprehension
squares = {x: x**2 for x in range(5)}  # {0:0, 1:1, 2:4, 3:9, 4:16}

# Merging dicts (Python 3.9+)
a = {"x": 1, "y": 2}
b = {"y": 10, "z": 3}
merged = a | b        # {"x": 1, "y": 10, "z": 3} — b overwrites a

# collections.defaultdict — auto-initializes missing keys
from collections import defaultdict
freq = defaultdict(int)
for word in "the cat sat on the mat".split():
    freq[word] += 1    # no KeyError — missing keys default to 0

# --- Sets: unordered, unique values, O(1) membership ---
s = {1, 2, 3, 4, 5}
s.add(6)
s.discard(10)     # remove if present — no error if missing (unlike .remove())
print(3 in s)     # True — O(1) vs list's O(n)

a = {1, 2, 3, 4}
b = {3, 4, 5, 6}
print(a | b)      # {1, 2, 3, 4, 5, 6} — union
print(a & b)      # {3, 4}             — intersection
print(a - b)      # {1, 2}             — difference (in a not b)
print(a ^ b)      # {1, 2, 5, 6}       — symmetric difference

# --- Tuples: ordered, IMMUTABLE ---
point = (10, 20)
x, y = point        # unpacking
print(point[0])     # 10
# point[0] = 5      # TypeError — tuples are immutable

# Named tuples — tuples with field names (no overhead over tuple)
from collections import namedtuple
Point = namedtuple("Point", ["x", "y"])
p = Point(10, 20)
print(p.x, p.y)   # 10 20
print(p[0])       # 10 — still indexable
```

### Level 3: Functions — All the Patterns

```python
# Basic function
def add(a, b):
    """Add two numbers and return the result.
    Docstrings describe what a function does — first line is summary."""
    return a + b

# Default arguments — evaluated ONCE at definition time (gotcha below)
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

print(greet("Alice"))          # Hello, Alice!
print(greet("Bob", "Hi"))      # Hi, Bob!

# THE MUTABLE DEFAULT ARGUMENT TRAP
def append_to(val, lst=[]):    # WRONG — list created once, shared across calls!
    lst.append(val)
    return lst

print(append_to(1))   # [1]
print(append_to(2))   # [1, 2] — not [2]! Same list reused.

def append_to_correct(val, lst=None):  # CORRECT — use None as sentinel
    if lst is None:
        lst = []
    lst.append(val)
    return lst

# *args and **kwargs
def display(*args, **kwargs):
    """Accept any positional and keyword arguments."""
    for i, arg in enumerate(args):
        print(f"arg[{i}] = {arg}")
    for key, val in kwargs.items():
        print(f"{key} = {val}")

display(1, 2, 3, name="Alice", age=30)

# Type hints (don't enforce at runtime, but help IDEs and type checkers)
def calculate_area(width: float, height: float) -> float:
    return width * height

# Lambda — anonymous function, one expression only
square = lambda x: x ** 2
print(square(5))   # 25
# Prefer named functions for anything non-trivial — lambdas reduce readability

# Closures — functions that capture variables from enclosing scope
def make_counter(start=0):
    count = start
    def increment():
        nonlocal count    # nonlocal needed to modify enclosing scope variable
        count += 1
        return count
    return increment

counter = make_counter(10)
print(counter())   # 11
print(counter())   # 12

# Decorators — wrap functions to add behavior
import functools
import time

def timer(func):
    @functools.wraps(func)    # preserves original function's metadata
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        elapsed = time.time() - start
        print(f"{func.__name__} took {elapsed:.4f}s")
        return result
    return wrapper

@timer
def slow_function(n):
    return sum(range(n))

result = slow_function(1_000_000)   # prints timing automatically

# Generator functions — produce values lazily (crucial for large datasets)
def fibonacci():
    a, b = 0, 1
    while True:
        yield a    # pauses here, returns a, resumes on next()
        a, b = b, a + b

fib = fibonacci()
print([next(fib) for _ in range(10)])  # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

# Generator expression — memory-efficient comprehension
sum_of_squares = sum(x**2 for x in range(1_000_000))  # never materializes the list
```

### Level 4: Classes and Object-Oriented Python

```python
from typing import Optional

class BankAccount:
    """A simple bank account with deposit, withdrawal, and history."""

    # Class variable — shared across ALL instances
    interest_rate = 0.02

    def __init__(self, owner: str, balance: float = 0.0):
        # Instance variables — unique to each instance
        self.owner = owner
        self._balance = balance      # _prefix = "private by convention"
        self._history: list[str] = []

    # Property — access _balance like an attribute but with validation
    @property
    def balance(self) -> float:
        return self._balance

    @balance.setter
    def balance(self, amount: float):
        if amount < 0:
            raise ValueError("Balance cannot be negative")
        self._balance = amount

    def deposit(self, amount: float) -> None:
        if amount <= 0:
            raise ValueError("Deposit must be positive")
        self._balance += amount
        self._history.append(f"Deposit: +{amount:.2f}")

    def withdraw(self, amount: float) -> float:
        if amount > self._balance:
            raise ValueError(f"Insufficient funds: {self._balance:.2f} available")
        self._balance -= amount
        self._history.append(f"Withdrawal: -{amount:.2f}")
        return amount

    def apply_interest(self):
        interest = self._balance * self.interest_rate
        self.deposit(interest)
        self._history[-1] = f"Interest: +{interest:.2f}"

    # Special methods (dunder methods) — define Python's built-in behavior
    def __str__(self) -> str:        # str(account), print(account)
        return f"Account({self.owner}: ${self._balance:.2f})"

    def __repr__(self) -> str:       # repr(account), shown in REPL
        return f"BankAccount('{self.owner}', {self._balance})"

    def __len__(self) -> int:        # len(account) — number of transactions
        return len(self._history)

    def __lt__(self, other: "BankAccount") -> bool:  # for sorting
        return self._balance < other._balance

    def __add__(self, other: "BankAccount") -> "BankAccount":
        """Merge two accounts — supports account1 + account2."""
        merged = BankAccount(f"{self.owner}+{other.owner}")
        merged._balance = self._balance + other._balance
        return merged

# Inheritance
class SavingsAccount(BankAccount):
    def __init__(self, owner: str, balance: float = 0.0, min_balance: float = 100.0):
        super().__init__(owner, balance)    # call parent __init__
        self.min_balance = min_balance

    def withdraw(self, amount: float) -> float:
        if self._balance - amount < self.min_balance:
            raise ValueError(f"Must maintain minimum balance of ${self.min_balance}")
        return super().withdraw(amount)    # call parent method

# Dataclass — eliminates boilerplate for data-holding classes (Python 3.7+)
from dataclasses import dataclass, field

@dataclass(order=True)    # auto-generates __lt__, __le__, __gt__, __ge__
class Product:
    name: str
    price: float
    quantity: int = 0
    tags: list[str] = field(default_factory=list)   # mutable defaults need field()

    @property
    def total_value(self) -> float:
        return self.price * self.quantity

p = Product("Widget", 9.99, 100)
print(p)              # Product(name='Widget', price=9.99, quantity=100, tags=[])
print(p.total_value)  # 999.0
```

### Level 5: Error Handling, Context Managers, and Concurrency

```python
import contextlib
import concurrent.futures

# --- Exception handling ---
def parse_config(filepath: str) -> dict:
    try:
        with open(filepath) as f:
            import json
            return json.load(f)
    except FileNotFoundError:
        raise FileNotFoundError(f"Config file not found: {filepath}")
    except json.JSONDecodeError as e:
        raise ValueError(f"Invalid JSON in {filepath}: {e}") from e
    except PermissionError as e:
        raise PermissionError(f"Cannot read {filepath}") from e
    finally:
        pass   # 'finally' always runs, even if exception is raised

# Exception hierarchy — catch specific before general
# Exception > OSError > FileNotFoundError (specific)
# Don't catch: bare except, or except Exception without re-raising

# Context managers — guarantee resource cleanup
class DatabaseConnection:
    def __enter__(self):
        self.conn = self._connect()
        return self.conn

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.conn.close()
        return False   # False: don't suppress exceptions

with DatabaseConnection() as conn:
    conn.query("SELECT 1")
# conn.close() called automatically, even if query raises

# @contextlib.contextmanager — easier way to write context managers
@contextlib.contextmanager
def timer(label: str):
    import time
    start = time.time()
    try:
        yield    # code inside 'with' runs here
    finally:
        elapsed = time.time() - start
        print(f"{label}: {elapsed:.3f}s")

with timer("data processing"):
    result = sum(range(10_000_000))

# --- Concurrency ---
# Python has the GIL — only one thread runs Python bytecode at a time.
# Use threads for I/O-bound work (network, files).
# Use processes for CPU-bound work.

# ThreadPoolExecutor — I/O-bound tasks (web requests, file operations)
import urllib.request

def fetch_url(url: str) -> str:
    with urllib.request.urlopen(url, timeout=5) as response:
        return response.read().decode()

urls = ["https://httpbin.org/delay/1"] * 4

with concurrent.futures.ThreadPoolExecutor(max_workers=4) as executor:
    futures = {executor.submit(fetch_url, url): url for url in urls}
    for future in concurrent.futures.as_completed(futures):
        url = futures[future]
        try:
            result = future.result()
            print(f"Fetched {url}: {len(result)} bytes")
        except Exception as e:
            print(f"Failed {url}: {e}")

# ProcessPoolExecutor — CPU-bound tasks (compression, encoding, heavy math)
def compute_heavy(n: int) -> int:
    return sum(i**2 for i in range(n))

with concurrent.futures.ProcessPoolExecutor() as executor:
    results = list(executor.map(compute_heavy, [100_000] * 8))

# asyncio — cooperative concurrency for high-volume I/O
import asyncio
import aiohttp   # pip install aiohttp

async def fetch_async(session, url: str) -> str:
    async with session.get(url) as response:
        return await response.text()

async def fetch_all(urls: list[str]) -> list[str]:
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_async(session, url) for url in urls]
        return await asyncio.gather(*tasks)   # run all concurrently

results = asyncio.run(fetch_all(urls))
```

### Level 6: The Standard Library You Must Know

```python
# pathlib — modern file paths (replaces os.path)
from pathlib import Path

data_dir = Path("/var/data")
config = Path.home() / ".config" / "myapp" / "config.json"
config.parent.mkdir(parents=True, exist_ok=True)

# Reading and writing
text = Path("data.txt").read_text(encoding="utf-8")
Path("output.txt").write_text("content", encoding="utf-8")

# Globbing
py_files = list(Path(".").glob("**/*.py"))   # all .py files recursively
for f in Path("/var/log").glob("*.log"):
    print(f.name, f.stat().st_size)

# itertools — combinatoric and infinite iterators
import itertools

# Chaining sequences
combined = list(itertools.chain([1,2], [3,4], [5,6]))   # [1,2,3,4,5,6]

# Grouping consecutive elements
data = [("Alice", "Engineering"), ("Bob", "Engineering"), ("Carol", "Marketing")]
data.sort(key=lambda x: x[1])   # must sort before groupby
for dept, people in itertools.groupby(data, key=lambda x: x[1]):
    print(dept, list(people))

# Sliding window
def sliding_window(iterable, n):
    iters = itertools.tee(iterable, n)
    for i, it in enumerate(iters):
        next(itertools.islice(it, i, i), None)   # advance each iterator
    return zip(*iters)

# functools — tools for higher-order functions
from functools import reduce, partial, lru_cache

@lru_cache(maxsize=None)    # memoize function results
def fib(n: int) -> int:
    if n < 2: return n
    return fib(n-1) + fib(n-2)

# partial — create new functions with pre-filled arguments
def power(base, exp):
    return base ** exp

square = partial(power, exp=2)
cube = partial(power, exp=3)
print(square(5), cube(3))   # 25 27

# collections — specialized containers
from collections import Counter, deque, OrderedDict

# Counter: frequency counting
words = "the cat sat on the mat the cat".split()
freq = Counter(words)
print(freq.most_common(3))   # [('the', 3), ('cat', 2), ('sat', 1)]
print(freq["cat"])           # 2
freq.update(["cat", "bat"])  # add more counts

# deque: O(1) appends/pops from BOTH ends
dq = deque([1, 2, 3], maxlen=5)   # maxlen: auto-discards oldest when full
dq.appendleft(0)    # O(1) — unlike list.insert(0, ...)
dq.rotate(1)        # [0, 1, 2, 3] → [3, 0, 1, 2]
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Mutable default argument**

```python
# WRONG: the list is created once at function definition
def append_to(val, lst=[]):
    lst.append(val)
    return lst

append_to(1)   # [1]
append_to(2)   # [1, 2] — NOT [2]! Same list object reused across calls.

# CORRECT: use None as default, create new object inside
def append_to(val, lst=None):
    if lst is None:
        lst = []
    lst.append(val)
    return lst
```

**Mistake 2: `is` vs `==`**

```python
# == compares VALUE (calls __eq__)
# is  compares IDENTITY (same object in memory)

a = [1, 2, 3]
b = [1, 2, 3]
print(a == b)   # True  — same value
print(a is b)   # False — different objects

# CORRECT for checking None:
if result is None: ...    # correct — None is a singleton
if result == None: ...    # works but wrong style; linters will flag it

# TRAP with small integers (CPython caches -5 to 256):
x = 256
y = 256
print(x is y)   # True — CPython reuses the same object
x = 257
y = 257
print(x is y)   # False — new objects created
# Never use 'is' to compare integers or strings.
```

**Mistake 3: Modifying a list while iterating over it**

```python
# WRONG: skips elements silently
nums = [1, 2, 3, 4, 5]
for i, n in enumerate(nums):
    if n % 2 == 0:
        nums.remove(n)   # shifts elements, causing skips
print(nums)   # [1, 3, 5] — looks right by accident here, but breaks on other inputs

# CORRECT: iterate over a copy, or build a new list
nums = [n for n in nums if n % 2 != 0]
# OR
for n in nums[:]:    # nums[:] is a shallow copy
    if n % 2 == 0:
        nums.remove(n)
```

**Mistake 4: Using `+` to concatenate strings in a loop**

```python
# WRONG: O(n²) — each += creates a new string object
result = ""
for word in word_list:
    result += word + " "   # copies entire string each time

# CORRECT: join is O(n) — builds result in one pass
result = " ".join(word_list)
```

**Mistake 5: Not using `enumerate` when you need both index and value**

```python
items = ["apple", "banana", "cherry"]

# WRONG: manual indexing
for i in range(len(items)):
    print(i, items[i])

# CORRECT: enumerate gives (index, value) pairs
for i, item in enumerate(items):
    print(i, item)

# enumerate with start index
for i, item in enumerate(items, start=1):
    print(f"{i}. {item}")
```

**Mistake 6: Catching too broad an exception**

```python
# WRONG: swallows real errors including bugs
try:
    result = process_data(data)
except Exception:
    pass   # silent failure — bugs become invisible

# WRONG: catches everything including KeyboardInterrupt, SystemExit
try:
    result = process_data(data)
except:
    pass

# CORRECT: catch specific exceptions, log them, re-raise or handle explicitly
try:
    result = process_data(data)
except ValueError as e:
    logger.error(f"Invalid data: {e}")
    raise   # re-raise to let caller decide
except ConnectionError as e:
    logger.warning(f"Connection failed, retrying: {e}")
    result = retry_process(data)
```

---

## 5. The "Why Does This Work" Layer

### Python's Object Model: Everything Is an Object

In CPython, every value is a Python object — a block of memory with a header containing the type pointer and reference count. Even `int` and `float` are Python objects (with an extra field for the value). This is why Python integers can be arbitrarily large (they allocate more memory as needed), but also why Python arithmetic is slower than C: every operation goes through the object system.

The reference count is the primary garbage collection mechanism. Every time you bind a name to an object (`x = obj`) or add it to a container, the count increments. When it drops to zero, the memory is freed immediately. Circular references (where objects reference each other) are handled by a separate cyclic GC that runs periodically.

### Why the GIL Limits Parallelism

The Global Interpreter Lock (GIL) is a mutex that protects CPython's internal state. Only one thread can execute Python bytecode at a time. This means multi-threaded Python code doesn't parallelize CPU work — threads take turns.

The GIL exists because CPython's memory management (reference counting) is not thread-safe. Without the GIL, you'd need fine-grained locks on every object, which would make single-threaded code much slower.

The GIL is released during I/O operations (network, disk, sleep) — so threads DO parallelize I/O-bound work effectively. For CPU-bound parallelism, use `multiprocessing` (separate processes with separate GILs) or write C extensions that release the GIL.

### Why Generators Are Memory-Efficient

A regular list comprehension `[f(x) for x in range(1_000_000)]` materializes all 1 million values in memory at once. A generator expression `(f(x) for x in range(1_000_000))` creates a generator object that produces values one at a time on demand.

The generator function's local state is preserved between `yield` calls: local variables, the current position in the code, and the execution stack frame are all stored in the generator object. This is why generators can represent infinite sequences like Fibonacci without running forever — they only compute what's requested.

---

## 6. Quick Reference

### Type Cheat Sheet

| Type | Mutable | Ordered | Duplicate Values | O(1) Lookup |
|------|---------|---------|-----------------|-------------|
| `list` | Yes | Yes | Yes | No (index only) |
| `tuple` | No | Yes | Yes | No (index only) |
| `dict` | Yes | Yes (3.7+) | Keys: No | Yes (by key) |
| `set` | Yes | No | No | Yes |
| `frozenset` | No | No | No | Yes |
| `str` | No | Yes | Yes | No (index only) |

### String Formatting

```python
name, n, pi = "Alice", 42, 3.14159

f"{name}"             # 'Alice'
f"{n:05d}"            # '00042'   (zero-padded integer)
f"{pi:.2f}"           # '3.14'    (2 decimal places)
f"{n:,}"              # '42'      (thousands separator)
f"{n:b}"              # '101010'  (binary)
f"{n:x}"              # '2a'      (hex)
f"{'left':<10}"       # 'left      ' (left-align, width 10)
f"{'right':>10}"      # '     right' (right-align)
f"{'center':^10}"     # '  center  ' (centered)
```

### Essential Built-ins

```python
len(x)            # length
type(x)           # type of x
isinstance(x, T)  # is x an instance of T?
range(n)          # 0..n-1
enumerate(it)     # (index, value) pairs
zip(a, b)         # pair elements from multiple iterables
map(f, it)        # apply f to each element
filter(f, it)     # elements where f returns True
sorted(it, key=f, reverse=True)
min(it, key=f)    # minimum by key function
max(it, key=f)
sum(it)
any(it)           # True if any element is truthy
all(it)           # True if all elements are truthy
```

### Collections Cheat Sheet

```python
from collections import Counter, defaultdict, deque, namedtuple, OrderedDict

Counter("hello")             # {'l': 2, 'h': 1, 'e': 1, 'o': 1}
defaultdict(list)            # auto-creates [] for missing keys
deque(maxlen=10)             # O(1) both ends; auto-drops oldest when full
namedtuple("Point", "x y")  # immutable, named fields
```
