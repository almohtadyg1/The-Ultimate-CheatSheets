# Python Complete Reference Guide
> From Zero to Expert — Everything You Need to Master Python

---

## 1. Installation & Setup

### Installing Python

**Linux**
```bash
sudo apt install python3 python3-pip     # Debian/Ubuntu
sudo dnf install python3 python3-pip     # Fedora/RHEL
```

**macOS**
```bash
brew install python
```

**Windows**
- Download from [python.org](https://python.org/downloads)
- Or: `winget install Python.Python.3`
- Or: `choco install python`

**Check version**
```bash
python3 --version
python3 -c "import sys; print(sys.version)"
```

### pyenv — Manage Multiple Python Versions

```bash
# Install pyenv
curl https://pyenv.run | bash

# Install a Python version
pyenv install 3.12.0
pyenv install 3.11.5

# Set global default
pyenv global 3.12.0

# Set version for current directory only
pyenv local 3.11.5

# List installed versions
pyenv versions
```

### Running Python

```bash
python3                      # Interactive REPL
python3 script.py            # Run a script
python3 -c "print('hello')"  # One-liner
python3 -m module_name       # Run a module as script
python3 -i script.py         # Run script then drop into REPL
python3 -u script.py         # Unbuffered output (useful for logs)

# Shebang line (add to top of script, then chmod +x)
#!/usr/bin/env python3
```

### IPython & Jupyter

```bash
pip install ipython jupyter

ipython                      # Enhanced REPL with tab completion
jupyter notebook             # Browser-based notebook
jupyter lab                  # JupyterLab interface
```

---

## 2. Python Basics

### Hello World

```python
print("Hello, World!")
print("Name:", "Alice", "Age:", 30)       # Multiple args, separated by space
print("A", "B", "C", sep="-")            # A-B-C
print("no newline", end="")              # No trailing newline
print(f"Pi is approximately {3.14159:.2f}")  # f-string
```

### Comments

```python
# This is a single-line comment

"""
This is a multi-line string,
often used as a docstring or block comment.
"""

def my_func():
    """This is a docstring — documents the function."""
    pass
```

### Variables & Assignment

```python
x = 10
name = "Alice"
pi = 3.14159
is_active = True

# Multiple assignment
a, b, c = 1, 2, 3
x = y = z = 0          # All set to 0

# Swap (no temp variable needed)
a, b = b, a

# Augmented assignment
x += 5    # x = x + 5
x -= 3
x *= 2
x /= 4    # Always returns float
x //= 2   # Integer (floor) division
x **= 2   # Exponentiation
x %= 3    # Modulo

# Walrus operator (Python 3.8+) — assign AND use in an expression
if n := len(my_list):
    print(f"List has {n} items")

while chunk := file.read(8192):
    process(chunk)
```

### Naming Conventions

| Style | Usage | Example |
|-------|-------|---------|
| `snake_case` | Variables, functions, modules | `user_name`, `get_value()` |
| `UPPER_SNAKE` | Constants | `MAX_SIZE`, `PI` |
| `PascalCase` | Classes | `UserAccount`, `HttpClient` |
| `_single_leading` | "Private" by convention | `_internal_var` |
| `__double_leading` | Name-mangled (truly private) | `__private` |
| `__dunder__` | Special/magic methods | `__init__`, `__str__` |

---

## 3. Data Types & Variables

### Built-in Types

```python
# Numeric
x = 42          # int (arbitrary precision)
y = 3.14        # float (64-bit double)
z = 2 + 3j      # complex
big = 10_000_000  # Underscores for readability

# Boolean (subclass of int: True==1, False==0)
flag = True
other = False

# None (the null value)
result = None

# Strings
s = "hello"
s = 'hello'      # Same thing
s = """multi
line"""

# Type checking
type(42)         # <class 'int'>
isinstance(42, int)      # True
isinstance(42, (int, float))  # True — checks multiple types
```

### Numbers

```python
# Integer operations
10 // 3       # 3  (floor division)
10 % 3        # 1  (modulo)
2 ** 10       # 1024 (exponentiation)
abs(-5)       # 5
divmod(10, 3) # (3, 1) — quotient and remainder

# Float
round(3.14159, 2)   # 3.14
import math
math.floor(3.7)     # 3
math.ceil(3.2)      # 4
math.sqrt(16)       # 4.0
math.pi             # 3.141592653589793
math.inf            # Infinity
math.nan            # Not a Number

# Float precision gotcha
0.1 + 0.2 == 0.3   # False! Use decimal for money
from decimal import Decimal
Decimal("0.1") + Decimal("0.2") == Decimal("0.3")  # True

# Integer bases
0b1010    # Binary: 10
0o17      # Octal: 15
0xFF      # Hex: 255
bin(255)  # '0b11111111'
hex(255)  # '0xff'
oct(8)    # '0o10'

# Type conversion
int("42")         # 42
int(3.9)          # 3  (truncates, not rounds)
float("3.14")     # 3.14
complex(3, 4)     # (3+4j)
int("FF", 16)     # 255 (base-16 parse)
int("1010", 2)    # 10  (base-2 parse)
```

### Truthiness

```python
# Falsy values — everything else is truthy
bool(None)      # False
bool(0)         # False
bool(0.0)       # False
bool("")        # False
bool([])        # False
bool({})        # False
bool(set())     # False

# Truthy
bool(1)         # True
bool(-1)        # True
bool("a")       # True
bool([0])       # True
```

---

## 4. Strings

### Creating Strings

```python
s1 = "double quotes"
s2 = 'single quotes'
s3 = """triple
double quotes — multi-line"""
s4 = '''triple
single quotes'''

# Raw string — backslashes not treated as escapes
path = r"C:\Users\Alice\Documents"
regex = r"\d+\.\d+"

# Byte string
b = b"bytes"
b = bytes([72, 101, 108, 108, 111])

# f-strings (Python 3.6+) — the best way to format
name = "Alice"
age = 30
s = f"Name: {name}, Age: {age}"
s = f"Pi = {3.14159:.2f}"          # Pi = 3.14
s = f"{name!r}"                    # repr() of name
s = f"{1000000:,}"                 # 1,000,000
s = f"{0.25:.0%}"                  # 25%
s = f"{'left':<10}|"               # Left-align in 10 chars
s = f"{'right':>10}|"              # Right-align
s = f"{'center':^10}|"             # Center
s = f"{42:08b}"                    # 00101010 (binary, zero-padded)
s = f"{255:#x}"                    # 0xff (hex with prefix)
# Debug shorthand (Python 3.8+)
x = 42
s = f"{x=}"                        # "x=42"
```

### Escape Sequences

```python
"\n"   # Newline
"\t"   # Tab
"\r"   # Carriage return
"\\"   # Backslash
"\'"   # Single quote
"\""   # Double quote
"\0"   # Null character
"\uXXXX"   # Unicode (4 hex digits)
"\UXXXXXXXX"  # Unicode (8 hex digits)
"\xhh"  # Hex character
```

### String Operations

```python
s = "Hello, World!"

# Length
len(s)              # 13

# Indexing & slicing
s[0]                # 'H'
s[-1]               # '!'
s[7:12]             # 'World'
s[:5]               # 'Hello'
s[7:]               # 'World!'
s[::2]              # 'Hlo ol!'  (every 2nd char)
s[::-1]             # '!dlroW ,olleH'  (reversed)

# Case
s.upper()           # 'HELLO, WORLD!'
s.lower()           # 'hello, world!'
s.title()           # 'Hello, World!'
s.capitalize()      # 'Hello, world!'
s.swapcase()        # 'hELLO, wORLD!'
s.casefold()        # Like lower() but more aggressive (ß → ss)

# Search
s.find("World")     # 7  (or -1 if not found)
s.index("World")    # 7  (raises ValueError if not found)
s.rfind("l")        # 10 (search from right)
s.count("l")        # 3
"World" in s        # True
s.startswith("He")  # True
s.endswith("!")     # True

# Modify (returns new string — strings are immutable)
s.replace("World", "Python")    # 'Hello, Python!'
s.replace("l", "L", 2)         # Replace only first 2 occurrences
s.strip()           # Remove leading/trailing whitespace
s.lstrip()          # Left strip
s.rstrip()          # Right strip
s.strip("!")        # Strip specific chars from both ends

# Split & join
"a,b,c".split(",")          # ['a', 'b', 'c']
"a  b  c".split()           # ['a', 'b', 'c']  (any whitespace)
"a,b,c".split(",", 1)       # ['a', 'b,c']  (max 1 split)
",".join(["a", "b", "c"])   # 'a,b,c'
"\n".join(lines)             # Join with newlines
"line1\nline2\n".splitlines()   # ['line1', 'line2']

# Check
s.isdigit()         # All digit characters?
s.isalpha()         # All alphabetic?
s.isalnum()         # All alphanumeric?
s.isspace()         # All whitespace?
s.isupper()
s.islower()

# Padding
"42".zfill(5)       # '00042'
"hi".center(10)     # '    hi    '
"hi".ljust(10, "-") # 'hi--------'
"hi".rjust(10, "-") # '--------hi'

# Encode/decode
"hello".encode("utf-8")              # b'hello'
b"hello".decode("utf-8")             # 'hello'
```

### String Formatting (all methods)

```python
name, score = "Alice", 98.5

# f-string (recommended)
f"Player {name} scored {score:.1f}"

# str.format()
"Player {} scored {:.1f}".format(name, score)
"Player {n} scored {s:.1f}".format(n=name, s=score)

# % formatting (old style, avoid)
"Player %s scored %.1f" % (name, score)

# Template strings (safe for user input)
from string import Template
t = Template("Player $name scored $score")
t.substitute(name=name, score=score)
```

---

## 5. Collections — List, Tuple, Set, Dict

### Lists

Ordered, mutable, allows duplicates.

```python
# Creation
lst = [1, 2, 3, 4, 5]
lst = list(range(10))
lst = list("abc")        # ['a', 'b', 'c']
lst = [0] * 5            # [0, 0, 0, 0, 0]

# Access & slicing (same as strings)
lst[0]                   # 1
lst[-1]                  # 5
lst[1:3]                 # [2, 3]
lst[::2]                 # [1, 3, 5]  (every 2nd)

# Mutation
lst.append(6)            # Add to end
lst.extend([7, 8])       # Add multiple
lst.insert(0, 0)         # Insert at index
lst[2] = 99              # Change element
lst[1:3] = [20, 30]      # Replace slice
del lst[0]               # Remove by index
lst.remove(3)            # Remove first occurrence of value
popped = lst.pop()       # Remove and return last
popped = lst.pop(1)      # Remove and return at index
lst.clear()              # Remove all elements

# Searching
3 in lst                 # True
lst.index(3)             # Index of first occurrence (raises if missing)
lst.count(3)             # Number of occurrences

# Sorting
lst.sort()               # In-place, ascending
lst.sort(reverse=True)   # In-place, descending
lst.sort(key=len)        # Sort by key function
lst.sort(key=lambda x: x.lower())
sorted_lst = sorted(lst)           # Returns new sorted list
sorted_lst = sorted(lst, reverse=True)

# Other
lst.reverse()            # In-place reverse
lst.copy()               # Shallow copy
lst2 = lst[:]            # Also shallow copy
import copy
deep = copy.deepcopy(lst)  # Deep copy (copies nested objects too)
len(lst)
min(lst)
max(lst)
sum(lst)

# Unpacking
a, b, c = [1, 2, 3]
first, *rest = [1, 2, 3, 4, 5]       # first=1, rest=[2,3,4,5]
*init, last = [1, 2, 3, 4, 5]        # init=[1,2,3,4], last=5
first, *mid, last = [1, 2, 3, 4, 5]  # first=1, mid=[2,3,4], last=5
```

### Tuples

Ordered, **immutable**, allows duplicates. Slightly faster than lists.

```python
# Creation
t = (1, 2, 3)
t = 1, 2, 3            # Parentheses optional
single = (42,)         # Single-element tuple — the comma is required!
single = 42,           # Same
empty = ()

# Access (same as lists — indexing, slicing, unpacking)
t[0]                   # 1
a, b, c = t

# Immutability
# t[0] = 99           # TypeError! Tuples can't be modified

# Useful: named tuples
from collections import namedtuple
Point = namedtuple("Point", ["x", "y"])
p = Point(3, 4)
p.x                    # 3
p.y                    # 4

# Even better: typing.NamedTuple
from typing import NamedTuple
class Point(NamedTuple):
    x: float
    y: float
    label: str = "origin"   # default value

p = Point(1.0, 2.0)
p.x, p.y               # 1.0, 2.0
```

### Sets

Unordered, **no duplicates**, mutable. O(1) membership test.

```python
# Creation
s = {1, 2, 3, 4, 5}
s = set([1, 2, 2, 3])   # {1, 2, 3} — duplicates removed
empty = set()            # NOT {} — that's an empty dict!

# Mutation
s.add(6)
s.update([7, 8, 9])     # Add multiple
s.remove(3)             # Raises KeyError if missing
s.discard(3)            # No error if missing
s.pop()                 # Remove and return arbitrary element
s.clear()

# Membership
3 in s                  # True — O(1)
3 not in s

# Set operations
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

a | b                   # Union: {1,2,3,4,5,6}
a.union(b)              # Same

a & b                   # Intersection: {3,4}
a.intersection(b)

a - b                   # Difference: {1,2}
a.difference(b)

a ^ b                   # Symmetric difference: {1,2,5,6}
a.symmetric_difference(b)

a <= b                  # Is a a subset of b?
a.issubset(b)
a >= b                  # Is a a superset of b?
a.issuperset(b)
a.isdisjoint(b)         # True if no common elements

# frozenset — immutable set (can be used as dict key)
fs = frozenset([1, 2, 3])
```

### Dictionaries

Key-value store. Keys must be hashable. Insertion-ordered since Python 3.7.

```python
# Creation
d = {"name": "Alice", "age": 30}
d = dict(name="Alice", age=30)
d = dict([("name", "Alice"), ("age", 30)])
d = {k: v for k, v in pairs}           # Dict comprehension

# Access
d["name"]                   # "Alice" — raises KeyError if missing
d.get("name")               # "Alice" — returns None if missing
d.get("email", "N/A")       # "N/A" — default value

# Mutation
d["email"] = "alice@example.com"   # Add or update
d.update({"city": "NYC", "age": 31})   # Merge/update multiple
d.update(city="NYC", age=31)           # Same with keyword args
d.setdefault("score", 0)    # Set only if key doesn't exist; returns value
del d["email"]
popped = d.pop("age")       # Remove and return; raises if missing
popped = d.pop("age", None) # Returns None if missing
d.clear()

# Views (live, reflect changes)
d.keys()                    # dict_keys([...])
d.values()                  # dict_values([...])
d.items()                   # dict_items([('k', 'v'), ...])
list(d.keys())              # Convert to list

# Iteration
for key in d:               # Iterate keys
    print(key, d[key])

for key, value in d.items():   # Iterate key-value pairs
    print(f"{key}: {value}")

# Membership
"name" in d                 # True — checks keys
"name" not in d

# Merge dicts
merged = {**d1, **d2}           # d2 values win on conflict (Python 3.5+)
merged = d1 | d2                 # Same (Python 3.9+)
d1 |= d2                         # In-place merge (Python 3.9+)

# Useful patterns
# Counting occurrences
counts = {}
for item in items:
    counts[item] = counts.get(item, 0) + 1

# Or with defaultdict
from collections import defaultdict
counts = defaultdict(int)
for item in items:
    counts[item] += 1

# Grouping
from collections import defaultdict
groups = defaultdict(list)
for item in items:
    groups[item.category].append(item)

# Ordered by value
sorted_by_val = dict(sorted(d.items(), key=lambda x: x[1]))
sorted_by_val = dict(sorted(d.items(), key=lambda x: x[1], reverse=True))
```

### Useful `collections` Types

```python
from collections import Counter, defaultdict, OrderedDict, deque, ChainMap

# Counter — count hashable objects
Counter("abracadabra")      # Counter({'a':5,'b':2,'r':2,'c':1,'d':1})
c = Counter(["a", "b", "a"])
c.most_common(2)            # [('a', 2), ('b', 1)]
c["a"]                      # 2 (0 for missing, not KeyError)
c + Counter("aab")          # Add counts
c - Counter("aab")          # Subtract counts

# deque — O(1) append/pop from both ends
dq = deque([1, 2, 3])
dq.appendleft(0)            # [0, 1, 2, 3]
dq.popleft()                # 0 — O(1), unlike list.pop(0)
dq.rotate(1)                # [3, 0, 1, 2]
dq = deque(maxlen=3)        # Fixed-size sliding window

# OrderedDict (less useful since Python 3.7 dicts are ordered)
od = OrderedDict()
od.move_to_end("key")       # Move to end or beginning

# ChainMap — layered lookup (like nested scopes)
defaults = {"color": "red", "size": "medium"}
overrides = {"color": "blue"}
cm = ChainMap(overrides, defaults)
cm["color"]                 # "blue" — overrides wins
cm["size"]                  # "medium" — from defaults
```

---

## 6. Control Flow

### if / elif / else

```python
x = 42

if x > 100:
    print("big")
elif x > 10:
    print("medium")
else:
    print("small")

# One-liner ternary
label = "big" if x > 100 else "small"

# Nested ternary (use sparingly)
label = "big" if x > 100 else "medium" if x > 10 else "small"

# Pattern matching (Python 3.10+) — match/case
match command:
    case "quit":
        quit_game()
    case "go" | "move":
        move_player()
    case ("go", direction):         # Sequence pattern
        move(direction)
    case {"action": action, "target": target}:  # Mapping pattern
        perform(action, target)
    case Point(x=0, y=0):          # Class pattern
        print("At origin")
    case _:                         # Wildcard (default)
        print("Unknown command")
```

### for Loops

```python
# Over any iterable
for i in range(5):        # 0, 1, 2, 3, 4
    print(i)

for i in range(2, 10, 2): # 2, 4, 6, 8
    print(i)

for item in my_list:
    print(item)

for char in "hello":
    print(char)

# enumerate — get index AND value
for i, item in enumerate(my_list):
    print(f"{i}: {item}")

for i, item in enumerate(my_list, start=1):   # Start index at 1
    print(f"{i}: {item}")

# zip — iterate multiple iterables in parallel
for name, score in zip(names, scores):
    print(f"{name}: {score}")

# zip_longest — zip with fill value for unequal lengths
from itertools import zip_longest
for a, b in zip_longest(lst1, lst2, fillvalue=0):
    print(a, b)

# dict items
for key, value in my_dict.items():
    print(key, value)

# else clause on for (runs if loop completed without break)
for item in items:
    if item == target:
        print("Found!")
        break
else:
    print("Not found")

# Loop control
for i in range(10):
    if i == 3:
        continue       # Skip rest of this iteration
    if i == 7:
        break          # Exit loop entirely
    print(i)
```

### while Loops

```python
count = 0
while count < 5:
    print(count)
    count += 1

# While with else (runs if condition became False naturally)
while count > 0:
    count -= 1
else:
    print("Done!")

# Infinite loop
while True:
    data = input("Enter: ")
    if data == "quit":
        break
    process(data)
```

---

## 7. Functions

### Basics

```python
def greet(name):
    """Greet someone by name."""   # docstring
    return f"Hello, {name}!"

result = greet("Alice")

# Multiple return values (actually returns a tuple)
def min_max(lst):
    return min(lst), max(lst)

lo, hi = min_max([3, 1, 4, 1, 5])

# No return value → returns None implicitly
def print_sep():
    print("---")
```

### Parameters

```python
# Positional
def add(a, b):
    return a + b
add(1, 2)       # Positional
add(b=2, a=1)   # Keyword — any order

# Default values
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

greet("Alice")             # "Hello, Alice!"
greet("Alice", "Hi")       # "Hi, Alice!"
greet("Alice", greeting="Hey")

# IMPORTANT: mutable defaults are shared across calls — a classic bug!
def bad(lst=[]):            # ❌ same list object every call
    lst.append(1)
    return lst

def good(lst=None):         # ✅ create new list each call
    if lst is None:
        lst = []
    lst.append(1)
    return lst

# *args — variadic positional arguments (tuple)
def sum_all(*args):
    return sum(args)

sum_all(1, 2, 3, 4)      # 10

# **kwargs — variadic keyword arguments (dict)
def show_info(**kwargs):
    for key, val in kwargs.items():
        print(f"{key}: {val}")

show_info(name="Alice", age=30)

# Combined — order matters: positional, *args, keyword-only, **kwargs
def func(a, b, *args, key="default", **kwargs):
    print(a, b, args, key, kwargs)

func(1, 2, 3, 4, key="custom", x=10, y=20)
# a=1, b=2, args=(3,4), key="custom", kwargs={'x':10,'y':20}

# Keyword-only parameters (after bare *)
def connect(host, port, *, timeout=30, retries=3):
    pass

connect("localhost", 8080, timeout=10)   # timeout must be keyword

# Positional-only parameters (before /, Python 3.8+)
def create(x, y, /, label="point"):
    pass

create(1, 2)              # x, y can only be positional
# create(x=1, y=2)       # TypeError!
```

### Unpacking into Function Calls

```python
def add(a, b, c):
    return a + b + c

args = [1, 2, 3]
add(*args)               # Unpack list as positional args

kwargs = {"a": 1, "b": 2, "c": 3}
add(**kwargs)            # Unpack dict as keyword args

add(*[1, 2], **{"c": 3})  # Mix
```

### Lambda Functions

```python
square = lambda x: x ** 2
add = lambda a, b: a + b

# Common uses
sorted(people, key=lambda p: p.age)
sorted(words, key=lambda w: w.lower())
filtered = filter(lambda x: x > 0, numbers)
doubled = map(lambda x: x * 2, numbers)
```

### Closures

```python
def make_multiplier(n):
    def multiplier(x):
        return x * n       # n is captured from enclosing scope
    return multiplier

double = make_multiplier(2)
triple = make_multiplier(3)
double(5)                  # 10
triple(5)                  # 15

# nonlocal — modify enclosing scope variable
def make_counter():
    count = 0
    def increment():
        nonlocal count     # Without this, count += 1 would create a new local
        count += 1
        return count
    return increment

counter = make_counter()
counter()  # 1
counter()  # 2
```

### Introspection

```python
help(len)               # Full help text
len.__doc__             # Docstring
len.__name__            # 'len'
import inspect
inspect.signature(func) # Function signature
inspect.getsource(func) # Source code
```

---

## 8. Comprehensions

Comprehensions are concise, readable, and faster than equivalent loops.

### List Comprehensions

```python
# [expression for item in iterable if condition]
squares = [x**2 for x in range(10)]
evens = [x for x in range(20) if x % 2 == 0]
words_upper = [w.upper() for w in words]

# Nested (outer loop first)
matrix = [[i*j for j in range(1,4)] for i in range(1,4)]
# [[1,2,3],[2,4,6],[3,6,9]]

flat = [x for row in matrix for x in row]   # Flatten 2D list

# With condition on output
labels = ["even" if x % 2 == 0 else "odd" for x in range(5)]

# Multiple conditions
filtered = [x for x in range(100) if x % 2 == 0 if x % 3 == 0]
```

### Dict & Set Comprehensions

```python
# Dict comprehension
squares = {x: x**2 for x in range(5)}  # {0:0, 1:1, 2:4, 3:9, 4:16}
inverted = {v: k for k, v in d.items()}  # Flip keys and values

# Filter dict
passing = {k: v for k, v in scores.items() if v >= 60}

# Set comprehension
unique_lengths = {len(w) for w in words}
```

### Generator Expressions

Like list comprehensions but **lazy** — they don't create a list in memory.

```python
# Use () instead of []
gen = (x**2 for x in range(1_000_000))   # No memory allocation yet
next(gen)              # Compute one at a time

# Efficient: pass directly to functions that accept iterables
total = sum(x**2 for x in range(1000))   # No list created
any(x > 50 for x in values)
all(x > 0 for x in values)
max(len(w) for w in words)

# Generator vs list comprehension
import sys
lst = [x**2 for x in range(10000)]
gen = (x**2 for x in range(10000))
sys.getsizeof(lst)     # ~85KB
sys.getsizeof(gen)     # ~208 bytes!
```

---

## 9. Object-Oriented Programming

### Classes & Instances

```python
class Dog:
    # Class variable — shared by all instances
    species = "Canis familiaris"
    count = 0

    # Constructor
    def __init__(self, name, age):
        # Instance variables
        self.name = name
        self.age = age
        Dog.count += 1

    # Instance method — receives self
    def bark(self):
        return f"{self.name} says: Woof!"

    def description(self):
        return f"{self.name} is {self.age} years old"

    # String representation
    def __str__(self):
        return f"Dog({self.name})"    # Human-readable (for print/str())

    def __repr__(self):
        return f"Dog(name={self.name!r}, age={self.age!r})"  # Unambiguous (for debug)

    # Destructor (rarely needed — prefer context managers)
    def __del__(self):
        Dog.count -= 1

# Usage
rex = Dog("Rex", 3)
rex.bark()            # "Rex says: Woof!"
str(rex)              # "Dog(Rex)"
repr(rex)             # "Dog(name='Rex', age=3)"
Dog.count             # 1
Dog.species           # "Canis familiaris"
rex.species           # Also "Canis familiaris" — searches instance then class
```

### Class & Static Methods

```python
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius

    @property
    def fahrenheit(self):
        return self.celsius * 9/5 + 32

    @fahrenheit.setter
    def fahrenheit(self, value):
        self.celsius = (value - 32) * 5/9

    @property
    def kelvin(self):
        return self.celsius + 273.15

    # classmethod — receives the class, not the instance
    @classmethod
    def from_fahrenheit(cls, fahrenheit):
        return cls((fahrenheit - 32) * 5/9)

    @classmethod
    def from_kelvin(cls, kelvin):
        return cls(kelvin - 273.15)

    # staticmethod — no access to class or instance
    @staticmethod
    def is_valid_celsius(c):
        return c >= -273.15

t = Temperature(100)
t.fahrenheit             # 212.0 — property access, no ()
t.fahrenheit = 32        # Triggers setter — t.celsius now 0
Temperature.from_fahrenheit(212)   # Alternate constructor
Temperature.is_valid_celsius(-300) # False
```

### Inheritance

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        raise NotImplementedError("Subclass must implement speak()")

    def __str__(self):
        return f"{type(self).__name__}({self.name!r})"

class Dog(Animal):
    def speak(self):
        return f"{self.name} says Woof!"

    def fetch(self):
        return f"{self.name} fetches the ball!"

class Cat(Animal):
    def speak(self):
        return f"{self.name} says Meow!"

class GoldenRetriever(Dog):
    def speak(self):
        # Call parent's method
        base = super().speak()
        return base + " (enthusiastically)"

# isinstance and issubclass
dog = Dog("Rex")
isinstance(dog, Dog)      # True
isinstance(dog, Animal)   # True — checks full hierarchy
issubclass(Dog, Animal)   # True

# MRO — Method Resolution Order
GoldenRetriever.__mro__
# (GoldenRetriever, Dog, Animal, object)
```

### Multiple Inheritance & Mixins

```python
class Flyable:
    def fly(self):
        return f"{self.name} is flying!"

class Swimmable:
    def swim(self):
        return f"{self.name} is swimming!"

class Duck(Animal, Flyable, Swimmable):
    def speak(self):
        return "Quack!"

d = Duck("Donald")
d.speak()   # Quack!
d.fly()     # Donald is flying!
d.swim()    # Donald is swimming!

# Mixin pattern — add behavior without inheritance hierarchy
class SerializeMixin:
    def to_dict(self):
        return self.__dict__

    def to_json(self):
        import json
        return json.dumps(self.__dict__)

class LoggingMixin:
    def log(self, message):
        print(f"[{type(self).__name__}] {message}")

class UserService(LoggingMixin, SerializeMixin):
    def __init__(self, name):
        self.name = name
```

### Dunder (Magic) Methods

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    # Representation
    def __repr__(self):   return f"Vector({self.x}, {self.y})"
    def __str__(self):    return f"({self.x}, {self.y})"

    # Arithmetic
    def __add__(self, other):   return Vector(self.x+other.x, self.y+other.y)
    def __sub__(self, other):   return Vector(self.x-other.x, self.y-other.y)
    def __mul__(self, scalar):  return Vector(self.x*scalar, self.y*scalar)
    def __rmul__(self, scalar): return self.__mul__(scalar)  # scalar * vector
    def __neg__(self):          return Vector(-self.x, -self.y)
    def __abs__(self):          return (self.x**2 + self.y**2) ** 0.5

    # Comparison
    def __eq__(self, other):    return self.x == other.x and self.y == other.y
    def __lt__(self, other):    return abs(self) < abs(other)
    def __le__(self, other):    return abs(self) <= abs(other)

    # Container protocol
    def __len__(self):          return 2
    def __getitem__(self, i):   return (self.x, self.y)[i]
    def __iter__(self):         return iter((self.x, self.y))
    def __contains__(self, v):  return v in (self.x, self.y)

    # Callable
    def __call__(self):         return abs(self)  # v() computes magnitude

    # Boolean
    def __bool__(self):         return self.x != 0 or self.y != 0

    # Context manager
    def __enter__(self):        return self
    def __exit__(self, *args):  pass

    # Hashing (needed if __eq__ is defined)
    def __hash__(self):         return hash((self.x, self.y))
```

### Dataclasses (Python 3.7+)

```python
from dataclasses import dataclass, field

@dataclass
class Point:
    x: float
    y: float
    z: float = 0.0              # Default value

@dataclass
class Player:
    name: str
    health: int = 100
    inventory: list = field(default_factory=list)  # Mutable default
    _id: int = field(init=False, repr=False)        # Not in __init__, not in repr

    def __post_init__(self):
        self._id = id(self)

@dataclass(frozen=True)         # Immutable (hashable)
class Color:
    r: int
    g: int
    b: int

@dataclass(order=True)          # Generates __lt__, __le__, etc.
class Version:
    major: int
    minor: int
    patch: int

# Auto-generates: __init__, __repr__, __eq__
p = Point(1.0, 2.0)
p.x                 # 1.0
repr(p)             # "Point(x=1.0, y=2.0, z=0.0)"

# Copy with changes
from dataclasses import replace
p2 = replace(p, x=99.0)
```

### Abstract Base Classes

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float:
        """Return the area of the shape."""
        ...

    @abstractmethod
    def perimeter(self) -> float:
        ...

    def describe(self):
        return f"Area: {self.area():.2f}, Perimeter: {self.perimeter():.2f}"

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        import math
        return math.pi * self.radius ** 2

    def perimeter(self):
        import math
        return 2 * math.pi * self.radius

# Shape()       # TypeError — can't instantiate abstract class
Circle(5)       # OK — implements all abstract methods
```

---

## 10. Modules & Packages

### Importing

```python
# Import a module
import math
math.sqrt(16)           # Use with module prefix

# Import specific names
from math import sqrt, pi
sqrt(16)                # Use directly

# Import with alias
import numpy as np
from datetime import datetime as dt

# Import everything (avoid — pollutes namespace)
from math import *

# Import a submodule
from os import path
from collections import defaultdict
```

### Creating Modules

```python
# mymodule.py
PI = 3.14159

def circle_area(r):
    return PI * r ** 2

class Circle:
    def __init__(self, r):
        self.r = r

# Guard — only runs when executed directly, not when imported
if __name__ == "__main__":
    print("Running directly")
    print(circle_area(5))
```

### Package Structure

```
mypackage/
├── __init__.py          ← Makes it a package; run on import
├── core.py
├── utils.py
└── subpackage/
    ├── __init__.py
    └── helper.py
```

```python
# mypackage/__init__.py
from .core import main_function      # Relative import
from .utils import helper_function
from . import subpackage

# Public API — what 'from mypackage import *' exports
__all__ = ["main_function", "helper_function"]
__version__ = "1.0.0"
__author__ = "Alice"
```

```python
# Usage
import mypackage
from mypackage import main_function
from mypackage.utils import helper_function
from mypackage.subpackage import helper
```

### Relative Imports

```python
# Inside mypackage/core.py
from . import utils           # Import sibling module
from .utils import something  # Import from sibling
from .. import other          # Import from parent package
from ..other import stuff     # Import from uncle module
```

### `sys.path` and Module Discovery

```python
import sys
sys.path                       # List of directories Python searches
sys.path.append("/my/dir")     # Add a search directory at runtime
sys.modules                    # Dict of all imported modules

# Find where a module is loaded from
import os
os.__file__

# Reload a module (useful in REPL during development)
import importlib
importlib.reload(mymodule)
```

---

## 11. File I/O

### Reading Files

```python
# Always use 'with' — auto-closes the file
with open("file.txt", "r") as f:
    content = f.read()         # Entire file as string

with open("file.txt") as f:    # "r" is default
    lines = f.readlines()      # List of lines (including \n)

with open("file.txt") as f:
    for line in f:             # Memory-efficient line-by-line
        print(line.strip())

with open("file.txt") as f:
    first_line = f.readline()  # One line at a time
```

### Writing Files

```python
with open("file.txt", "w") as f:   # Overwrite (creates if missing)
    f.write("Hello\n")
    f.write("World\n")

with open("file.txt", "a") as f:   # Append
    f.write("New line\n")

# Write multiple lines at once
lines = ["line1\n", "line2\n", "line3\n"]
with open("file.txt", "w") as f:
    f.writelines(lines)

# print() to file
with open("out.txt", "w") as f:
    print("Hello", file=f)
```

### Open Modes

| Mode | Meaning |
|------|---------|
| `r` | Read (default) |
| `w` | Write (overwrite) |
| `a` | Append |
| `x` | Create (fails if exists) |
| `b` | Binary mode (`rb`, `wb`) |
| `t` | Text mode (default) |
| `+` | Read+Write (`r+`, `w+`) |

### Binary Files

```python
with open("image.png", "rb") as f:
    data = f.read()                 # bytes object

with open("copy.png", "wb") as f:
    f.write(data)

# Seeking
with open("file.bin", "rb") as f:
    f.seek(10)                      # Move to byte 10
    f.read(4)                       # Read 4 bytes
    f.tell()                        # Current position
    f.seek(-4, 2)                   # 4 bytes from end
```

### Working with Paths (`pathlib`)

```python
from pathlib import Path

# Create path objects
p = Path("/home/alice/docs/file.txt")
p = Path.home() / "docs" / "file.txt"   # / operator for joining!
p = Path(".")                            # Current directory

# Path info
p.name              # "file.txt"
p.stem              # "file"
p.suffix            # ".txt"
p.suffixes          # [".txt"]
p.parent            # Path("/home/alice/docs")
p.parts             # ('/', 'home', 'alice', 'docs', 'file.txt')
p.is_absolute()     # True

# Check existence
p.exists()
p.is_file()
p.is_dir()

# Read/write directly
p.read_text()                       # Read entire file as string
p.write_text("hello\n")             # Write string
p.read_bytes()
p.write_bytes(b"\x00\x01")

# Directory operations
Path("newdir").mkdir(parents=True, exist_ok=True)
p.unlink()                          # Delete file
p.rmdir()                           # Delete empty directory

# Glob
list(Path(".").glob("*.py"))        # All .py in current dir
list(Path(".").rglob("*.py"))       # Recursive
list(Path(".").glob("**/*.py"))     # Same as rglob

# Rename and move
p.rename("new_name.txt")
p.replace("/other/dir/file.txt")    # Move (overwrite if exists)

# Other
p.stat().st_size                    # File size in bytes
p.stat().st_mtime                   # Modification time
p.resolve()                         # Absolute path, resolve symlinks
```

### CSV, JSON, and other formats

```python
# CSV
import csv

with open("data.csv", "r") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row["name"], row["age"])

with open("data.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "age"])
    writer.writeheader()
    writer.writerow({"name": "Alice", "age": 30})

# JSON
import json

data = {"name": "Alice", "scores": [95, 87, 92]}
json_str = json.dumps(data, indent=2)           # Serialize to string
json_str = json.dumps(data, ensure_ascii=False) # Allow unicode
data2 = json.loads(json_str)                    # Deserialize

with open("data.json", "w") as f:
    json.dump(data, f, indent=2)

with open("data.json") as f:
    data = json.load(f)

# pickle — Python-specific binary serialization (don't use for untrusted data)
import pickle
with open("data.pkl", "wb") as f:
    pickle.dump(obj, f)
with open("data.pkl", "rb") as f:
    obj = pickle.load(f)
```

---

## 12. Error Handling & Exceptions

### Try / Except

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero")

# Multiple except clauses
try:
    x = int(input("Number: "))
    result = 10 / x
except ValueError:
    print("Not a valid number")
except ZeroDivisionError:
    print("Cannot divide by zero")
except (TypeError, AttributeError) as e:
    print(f"Type error: {e}")
except Exception as e:
    print(f"Unexpected error: {type(e).__name__}: {e}")
    raise                          # Re-raise the same exception
finally:
    print("Always runs — cleanup here")

# else clause (runs only if no exception was raised)
try:
    x = int("42")
except ValueError:
    print("Bad input")
else:
    print(f"Got: {x}")             # Runs if no exception
finally:
    print("Done")

# Suppress specific exceptions
from contextlib import suppress
with suppress(FileNotFoundError):
    Path("missing.txt").unlink()   # Won't raise if file missing
```

### Exception Hierarchy

```
BaseException
├── SystemExit
├── KeyboardInterrupt
├── GeneratorExit
└── Exception
    ├── StopIteration
    ├── ArithmeticError
    │   ├── ZeroDivisionError
    │   └── OverflowError
    ├── AttributeError
    ├── ImportError
    │   └── ModuleNotFoundError
    ├── LookupError
    │   ├── IndexError
    │   └── KeyError
    ├── NameError
    ├── OSError
    │   ├── FileNotFoundError
    │   ├── PermissionError
    │   └── TimeoutError
    ├── RuntimeError
    │   └── RecursionError
    ├── TypeError
    ├── ValueError
    │   └── UnicodeError
    └── Warning
```

### Raising Exceptions

```python
raise ValueError("Invalid value")
raise TypeError(f"Expected int, got {type(x).__name__}")
raise RuntimeError("Something went wrong") from original_error  # Chained

# Re-raise current exception
try:
    risky()
except Exception:
    log_error()
    raise
```

### Custom Exceptions

```python
class AppError(Exception):
    """Base class for application errors."""

class DatabaseError(AppError):
    def __init__(self, message, query=None, code=None):
        super().__init__(message)
        self.query = query
        self.code = code

    def __str__(self):
        base = super().__str__()
        if self.code:
            return f"[{self.code}] {base}"
        return base

class ValidationError(AppError):
    def __init__(self, field, message):
        super().__init__(f"Validation failed for '{field}': {message}")
        self.field = field

# Usage
try:
    raise DatabaseError("Connection failed", code="DB001")
except DatabaseError as e:
    print(e.code, str(e))
except AppError:
    print("General app error")
```

### Warnings

```python
import warnings

warnings.warn("This is deprecated", DeprecationWarning, stacklevel=2)
warnings.warn("Low memory", ResourceWarning)

# Filter warnings
warnings.filterwarnings("ignore", category=DeprecationWarning)
warnings.filterwarnings("error")    # Turn all warnings into errors

with warnings.catch_warnings():
    warnings.simplefilter("ignore")
    legacy_function()
```

---

## 13. Iterators & Generators

### Iterators Protocol

```python
# Any object with __iter__ and __next__ is an iterator
class CountUp:
    def __init__(self, limit):
        self.limit = limit
        self.current = 0

    def __iter__(self):
        return self           # Return iterator object

    def __next__(self):
        if self.current >= self.limit:
            raise StopIteration   # Signal exhaustion
        value = self.current
        self.current += 1
        return value

for n in CountUp(5):
    print(n)         # 0 1 2 3 4

# iter() and next() built-ins
it = iter([1, 2, 3])
next(it)            # 1
next(it)            # 2
next(it, "done")    # 3
next(it, "done")    # "done" (default when exhausted)
```

### Generators

A function that uses `yield` is a generator. It returns a generator object that is lazy — values are computed one at a time.

```python
def count_up(limit):
    for i in range(limit):
        yield i           # Pause here, return i, resume later

gen = count_up(5)
next(gen)            # 0
next(gen)            # 1
list(gen)            # [2, 3, 4]

# Infinite generator
def naturals():
    n = 0
    while True:
        yield n
        n += 1

from itertools import islice
list(islice(naturals(), 10))   # [0,1,...,9]

# Generator with send() — bidirectional
def accumulator():
    total = 0
    while True:
        value = yield total     # Pause, send total out, receive value
        total += value

acc = accumulator()
next(acc)           # Prime — advance to first yield (returns 0)
acc.send(10)        # 10
acc.send(5)         # 15
acc.send(3)         # 18

# yield from — delegate to sub-generator
def chain(*iterables):
    for it in iterables:
        yield from it

list(chain([1,2], [3,4], [5]))   # [1,2,3,4,5]
```

### `itertools` — Iterator Toolkit

```python
import itertools as it

# Infinite iterators
it.count(10, 2)              # 10, 12, 14, ... (never ends)
it.cycle([1,2,3])            # 1,2,3,1,2,3,... (never ends)
it.repeat(42, times=3)       # 42, 42, 42

# Finite iterators
list(it.chain([1,2], [3,4]))         # [1,2,3,4]
list(it.chain.from_iterable([[1,2],[3,4]]))  # Same
list(it.islice(range(100), 5, 15, 2))  # [5,7,9,11,13]
list(it.zip_longest("AB", [1], fillvalue=0))  # [('A',1),('B',0)]
list(it.takewhile(lambda x: x<5, [1,3,5,2]))  # [1,3]
list(it.dropwhile(lambda x: x<5, [1,3,5,2]))  # [5,2]
list(it.filterfalse(lambda x: x%2, range(6))) # [0,2,4]
list(it.compress("ABCDE", [1,0,1,0,1]))       # ['A','C','E']
list(it.accumulate([1,2,3,4], operator.mul))  # [1,2,6,24]

# Combinatoric iterators
list(it.product("AB", repeat=2))             # AA AB BA BB
list(it.permutations("ABC", 2))              # AB AC BA BC CA CB
list(it.combinations("ABC", 2))             # AB AC BC
list(it.combinations_with_replacement("AB",2)) # AA AB BB

# Grouping
data = sorted([("a",1),("b",2),("a",3)], key=lambda x: x[0])
for key, group in it.groupby(data, key=lambda x: x[0]):
    print(key, list(group))
# a [('a',1),('a',3)]
# b [('b',2)]
```

---

## 14. Decorators

Decorators are functions that wrap other functions to modify or extend their behavior.

### Basic Decorator

```python
import functools

def my_decorator(func):
    @functools.wraps(func)    # Preserves __name__, __doc__, etc.
    def wrapper(*args, **kwargs):
        print("Before")
        result = func(*args, **kwargs)
        print("After")
        return result
    return wrapper

@my_decorator
def greet(name):
    print(f"Hello, {name}!")

greet("Alice")
# Before
# Hello, Alice!
# After

# Equivalent to:
greet = my_decorator(greet)
```

### Decorator with Arguments

```python
def repeat(n):
    """Decorator factory — takes arguments, returns a decorator."""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(n):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def greet(name):
    print(f"Hello, {name}!")

greet("Alice")   # Prints 3 times
```

### Practical Decorators

```python
import time
import functools
import logging

# Timer
def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.4f}s")
        return result
    return wrapper

# Memoization / caching
def memoize(func):
    cache = {}
    @functools.wraps(func)
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    return wrapper

# Or use the built-in:
from functools import lru_cache, cache

@lru_cache(maxsize=128)   # LRU cache with size limit
def fib(n):
    if n < 2: return n
    return fib(n-1) + fib(n-2)

@cache   # Unbounded cache (Python 3.9+)
def expensive(n):
    ...

# Retry on failure
def retry(times=3, exceptions=(Exception,), delay=1):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(times):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt == times - 1:
                        raise
                    time.sleep(delay)
        return wrapper
    return decorator

@retry(times=3, exceptions=(ConnectionError,), delay=2)
def fetch_data(url):
    ...

# Type validation
def validate_types(**type_map):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            import inspect
            bound = inspect.signature(func).bind(*args, **kwargs)
            for param, expected_type in type_map.items():
                if param in bound.arguments:
                    val = bound.arguments[param]
                    if not isinstance(val, expected_type):
                        raise TypeError(f"{param} must be {expected_type.__name__}")
            return func(*args, **kwargs)
        return wrapper
    return decorator

@validate_types(name=str, age=int)
def create_user(name, age):
    ...
```

### Class Decorators

```python
# Decorator that's a class
class Singleton:
    _instances = {}

    def __call__(self, cls):
        @functools.wraps(cls)
        def wrapper(*args, **kwargs):
            if cls not in self._instances:
                self._instances[cls] = cls(*args, **kwargs)
            return self._instances[cls]
        return wrapper

singleton = Singleton()

@singleton
class Database:
    def __init__(self):
        self.connection = "connected"

# Applying a decorator to a whole class
def add_repr(cls):
    def __repr__(self):
        attrs = ", ".join(f"{k}={v!r}" for k, v in self.__dict__.items())
        return f"{type(self).__name__}({attrs})"
    cls.__repr__ = __repr__
    return cls

@add_repr
class Config:
    def __init__(self, host, port):
        self.host = host
        self.port = port
```

### Stacking Decorators

```python
@timer
@memoize
@validate_types(n=int)
def fib(n):
    if n < 2: return n
    return fib(n-1) + fib(n-2)

# Applied bottom-up: validate_types first, then memoize, then timer
# Equivalent to: timer(memoize(validate_types(n=int)(fib)))
```

---

## 15. Context Managers

Context managers handle setup/teardown automatically using the `with` statement.

### Using Context Managers

```python
# File (most common)
with open("file.txt") as f:
    data = f.read()
# File is closed here, even if an exception occurred

# Multiple context managers
with open("in.txt") as src, open("out.txt", "w") as dst:
    dst.write(src.read())

# Threading
import threading
lock = threading.Lock()
with lock:
    shared_resource += 1
```

### Creating with `__enter__` / `__exit__`

```python
class Timer:
    def __enter__(self):
        import time
        self.start = time.perf_counter()
        return self                 # Bound to 'as' variable

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.elapsed = time.perf_counter() - self.start
        print(f"Elapsed: {self.elapsed:.4f}s")
        return False                # False = don't suppress exceptions
                                    # True  = suppress exceptions

with Timer() as t:
    [x**2 for x in range(1_000_000)]
print(t.elapsed)
```

### Creating with `@contextmanager`

```python
from contextlib import contextmanager

@contextmanager
def timer():
    import time
    start = time.perf_counter()
    try:
        yield                    # Code inside 'with' block runs here
    finally:
        elapsed = time.perf_counter() - start
        print(f"Elapsed: {elapsed:.4f}s")

@contextmanager
def temporary_directory():
    import tempfile, shutil
    tmpdir = tempfile.mkdtemp()
    try:
        yield tmpdir
    finally:
        shutil.rmtree(tmpdir)

@contextmanager
def managed_db_connection(dsn):
    conn = connect(dsn)
    try:
        yield conn
        conn.commit()              # Only commits if no exception
    except Exception:
        conn.rollback()
        raise
    finally:
        conn.close()

with managed_db_connection("sqlite:///db.sqlite") as db:
    db.execute("INSERT ...")
```

### `contextlib` Utilities

```python
from contextlib import (
    suppress,          # Suppress specified exceptions
    redirect_stdout,   # Redirect stdout
    ExitStack,         # Dynamically manage multiple CMs
    nullcontext,       # No-op context manager
    asynccontextmanager,
)

# suppress — silently ignore exceptions
with suppress(FileNotFoundError, PermissionError):
    Path("missing.txt").unlink()

# redirect_stdout
import io
buffer = io.StringIO()
with redirect_stdout(buffer):
    print("This goes to buffer")
output = buffer.getvalue()

# ExitStack — dynamic context managers
with ExitStack() as stack:
    files = [stack.enter_context(open(f)) for f in filenames]
    # All files closed when ExitStack exits

# nullcontext — conditional context manager
cm = lock if use_lock else nullcontext()
with cm:
    shared_resource += 1
```

---

## 16. Functional Programming

### `map`, `filter`, `reduce`

```python
nums = [1, 2, 3, 4, 5]

# map — transform each element
doubled = list(map(lambda x: x * 2, nums))  # [2,4,6,8,10]
# Prefer list comprehension: [x*2 for x in nums]

# filter — keep elements where function returns True
evens = list(filter(lambda x: x % 2 == 0, nums))
# Prefer: [x for x in nums if x % 2 == 0]

# reduce — fold a sequence into one value
from functools import reduce
total = reduce(lambda acc, x: acc + x, nums, 0)   # 15
product = reduce(lambda acc, x: acc * x, nums, 1) # 120
# Prefer: sum(nums), math.prod(nums)
```

### `functools` Module

```python
from functools import partial, reduce, wraps, total_ordering

# partial — pre-fill some arguments
def power(base, exp):
    return base ** exp

square = partial(power, exp=2)
cube   = partial(power, exp=3)
square(5)   # 25
cube(3)     # 27

# More practical example
from functools import partial
import os
join_base = partial(os.path.join, "/home/alice")
join_base("docs")     # /home/alice/docs
join_base("images")   # /home/alice/images

# total_ordering — define __eq__ and one comparison; get the rest for free
from functools import total_ordering

@total_ordering
class Student:
    def __init__(self, name, gpa):
        self.name = name
        self.gpa = gpa

    def __eq__(self, other):
        return self.gpa == other.gpa

    def __lt__(self, other):
        return self.gpa < other.gpa
    # Now __le__, __gt__, __ge__ are automatically derived
```

### Higher-Order Functions

```python
# Functions that take or return functions
def compose(*funcs):
    """Compose functions right-to-left: compose(f, g)(x) == f(g(x))"""
    def composed(x):
        for f in reversed(funcs):
            x = f(x)
        return x
    return composed

to_upper = str.upper
strip    = str.strip
process  = compose(to_upper, strip)
process("  hello  ")   # "HELLO"

# pipe — compose left-to-right
def pipe(value, *funcs):
    for f in funcs:
        value = f(value)
    return value

result = pipe("  hello  ", str.strip, str.upper, lambda s: s + "!")
# "HELLO!"

# Functions returning functions
def make_adder(n):
    return lambda x: x + n

add5  = make_adder(5)
add10 = make_adder(10)
```

---

## 17. Type Hints & Annotations

Type hints are optional but improve readability, IDE support, and catch bugs with static type checkers like `mypy`.

### Basic Annotations

```python
# Variables
name: str = "Alice"
age: int = 30
price: float = 9.99
is_active: bool = True

# Functions
def greet(name: str) -> str:
    return f"Hello, {name}!"

def add(a: int, b: int) -> int:
    return a + b

def no_return(x: int) -> None:
    print(x)
```

### `typing` Module (Python 3.5–3.8 style)

```python
from typing import List, Dict, Tuple, Set, Optional, Union, Any, Callable

def process(items: List[int]) -> Dict[str, int]:
    return {str(i): i for i in items}

def get_coords() -> Tuple[float, float]:
    return 1.0, 2.0

# Optional — can be None
def find(name: str) -> Optional[str]:
    return name if name else None

# Union — one of several types
def stringify(x: Union[int, float, str]) -> str:
    return str(x)

# Any — opt out of type checking
def dynamic(x: Any) -> Any:
    return x

# Callable
def apply(func: Callable[[int, int], int], a: int, b: int) -> int:
    return func(a, b)
```

### Modern Syntax (Python 3.9+ / 3.10+)

```python
# Use built-in generics directly (no need to import from typing)
def process(items: list[int]) -> dict[str, int]:
    return {str(i): i for i in items}

def coords() -> tuple[float, float]:
    return 1.0, 2.0

# Union with | (Python 3.10+)
def stringify(x: int | float | str) -> str:
    return str(x)

# Optional[X] is just X | None
def find(name: str) -> str | None:
    return name if name else None

# Complex nested types
from typing import TypeVar, Generic

T = TypeVar("T")

class Stack(Generic[T]):
    def __init__(self) -> None:
        self._items: list[T] = []

    def push(self, item: T) -> None:
        self._items.append(item)

    def pop(self) -> T:
        return self._items.pop()
```

### Special Types

```python
from typing import (
    TypeVar, Generic, Protocol, TypedDict,
    Final, ClassVar, Literal, Annotated,
    overload, TYPE_CHECKING
)

# Final — cannot be reassigned
MAX_SIZE: Final = 100

# ClassVar — class variable, not instance
class Config:
    debug: ClassVar[bool] = False

# Literal — exact value
Mode = Literal["r", "w", "a", "rb", "wb"]
def open_file(path: str, mode: Mode) -> None: ...

# TypedDict — dict with known keys and types
class Movie(TypedDict):
    title: str
    year: int
    rating: float

m: Movie = {"title": "Alien", "year": 1979, "rating": 8.4}

# Protocol — structural subtyping ("duck typing" for type checkers)
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None: ...
    def resize(self, factor: float) -> None: ...

def render(item: Drawable) -> None:    # Works for any class with draw() and resize()
    item.draw()

# Annotated — attach metadata
from typing import Annotated
Positive = Annotated[int, "must be > 0"]

# overload — different signatures
@overload
def process(x: int) -> int: ...
@overload
def process(x: str) -> str: ...
def process(x):
    return x

# TYPE_CHECKING — avoid circular imports
if TYPE_CHECKING:
    from mymodule import MyClass
```

### Running mypy

```bash
pip install mypy
mypy script.py
mypy --strict script.py          # Maximum strictness
mypy --ignore-missing-imports .  # Ignore missing stubs
```

---

## 18. Concurrency — Threads, Async, Multiprocessing

### Threading (I/O-bound tasks)

```python
import threading
import time

def worker(name, delay):
    print(f"{name} starting")
    time.sleep(delay)
    print(f"{name} done")

# Create and start threads
t1 = threading.Thread(target=worker, args=("Thread-1", 2))
t2 = threading.Thread(target=worker, args=("Thread-2", 1), daemon=True)
t1.start()
t2.start()
t1.join()    # Wait for t1 to finish
t2.join()

# Thread pool
from concurrent.futures import ThreadPoolExecutor

def fetch(url):
    import urllib.request
    with urllib.request.urlopen(url) as r:
        return r.read()

urls = ["https://example.com", "https://python.org"]
with ThreadPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(fetch, urls))

# Submit individual tasks
with ThreadPoolExecutor() as executor:
    futures = {executor.submit(fetch, url): url for url in urls}
    for future in futures:
        try:
            result = future.result(timeout=10)
        except Exception as e:
            print(f"Error: {e}")

# Locks and synchronization
lock = threading.Lock()
rlock = threading.RLock()       # Reentrant lock
event = threading.Event()       # Signal between threads
sem = threading.Semaphore(3)    # Limit concurrent access
cond = threading.Condition()    # Producer-consumer pattern

counter = 0
def safe_increment():
    global counter
    with lock:
        counter += 1

# Note: Python's GIL means only one thread runs Python bytecode at a time.
# Threading is great for I/O-bound work; use multiprocessing for CPU-bound.
```

### Asyncio (concurrent I/O without threads)

```python
import asyncio

# async def creates a coroutine
async def fetch_data(url: str) -> str:
    await asyncio.sleep(1)      # Non-blocking "sleep" (simulates I/O)
    return f"Data from {url}"

# Run a single coroutine
asyncio.run(fetch_data("https://example.com"))

# Run multiple concurrently
async def main():
    # gather — run all concurrently, wait for all to complete
    results = await asyncio.gather(
        fetch_data("https://example.com"),
        fetch_data("https://python.org"),
        fetch_data("https://github.com"),
    )
    for r in results:
        print(r)

asyncio.run(main())

# Tasks — schedule coroutines to run concurrently
async def main():
    task1 = asyncio.create_task(fetch_data("url1"))
    task2 = asyncio.create_task(fetch_data("url2"))
    # Both tasks are now running concurrently
    r1 = await task1
    r2 = await task2

# Timeout
async def main():
    try:
        result = await asyncio.wait_for(fetch_data("url"), timeout=5.0)
    except asyncio.TimeoutError:
        print("Timed out!")

# Async generators
async def paginate(url):
    page = 0
    while True:
        data = await fetch_page(url, page)
        if not data:
            break
        yield data
        page += 1

async def main():
    async for page in paginate("https://api.example.com/data"):
        process(page)

# Async context manager
class AsyncDB:
    async def __aenter__(self):
        self.conn = await connect()
        return self.conn

    async def __aexit__(self, *args):
        await self.conn.close()

async def main():
    async with AsyncDB() as db:
        await db.execute("SELECT 1")

# asyncio Queue — async producer/consumer
async def producer(queue):
    for i in range(5):
        await queue.put(i)
        await asyncio.sleep(0.1)
    await queue.put(None)  # Sentinel

async def consumer(queue):
    while True:
        item = await queue.get()
        if item is None:
            break
        print(f"Consumed: {item}")
        queue.task_done()

async def main():
    queue = asyncio.Queue(maxsize=10)
    await asyncio.gather(producer(queue), consumer(queue))
```

### Multiprocessing (CPU-bound tasks)

```python
from multiprocessing import Pool, Process, Queue, Value, Array
import multiprocessing as mp

def cpu_heavy(n):
    return sum(i**2 for i in range(n))

# Process pool — parallel CPU work
with Pool(processes=mp.cpu_count()) as pool:
    results = pool.map(cpu_heavy, [100_000] * 8)
    # Or starmap for multiple args:
    results = pool.starmap(pow, [(2, 10), (3, 5), (4, 4)])

# ProcessPoolExecutor (futures API)
from concurrent.futures import ProcessPoolExecutor

with ProcessPoolExecutor() as executor:
    futures = [executor.submit(cpu_heavy, 100_000) for _ in range(8)]
    results = [f.result() for f in futures]

# Shared memory
with Pool() as pool:
    # Each worker gets its own copy — no shared state issues
    # Use multiprocessing.Manager() for shared state if needed
    from multiprocessing import Manager
    with Manager() as mgr:
        shared_list = mgr.list()
        shared_dict = mgr.dict()
```

---

## 19. The Standard Library

### `os` and `sys`

```python
import os, sys

# OS interaction
os.getcwd()                     # Current working directory
os.chdir("/path/to/dir")        # Change directory
os.listdir(".")                 # List directory contents
os.makedirs("a/b/c", exist_ok=True)
os.remove("file.txt")
os.rename("old.txt", "new.txt")
os.path.join("dir", "file.txt")
os.path.exists("file.txt")
os.path.isfile("file.txt")
os.path.isdir("dir")
os.path.abspath("file.txt")
os.path.dirname("/path/to/file.txt")  # /path/to
os.path.basename("/path/to/file.txt") # file.txt
os.path.splitext("file.txt")          # ('file', '.txt')
os.getenv("HOME")                # Read environment variable
os.environ["PATH"]               # Access env vars dict
os.cpu_count()
os.getpid()                      # Current process ID

# Walk directory tree
for root, dirs, files in os.walk("."):
    for filename in files:
        print(os.path.join(root, filename))

# Run a command
os.system("ls -la")              # Simple — use subprocess for more control
import subprocess
result = subprocess.run(["ls", "-la"], capture_output=True, text=True)
print(result.stdout)
subprocess.run(["git", "commit", "-m", "msg"], check=True)

# sys module
sys.argv                         # Command-line arguments ['script.py', 'arg1', ...]
sys.exit(0)                      # Exit with code
sys.exit("Error message")        # Exit with message (code 1)
sys.path                         # Module search path
sys.version                      # Python version string
sys.platform                     # 'linux', 'darwin', 'win32'
sys.stdin, sys.stdout, sys.stderr
sys.getrecursionlimit()          # Default: 1000
sys.setrecursionlimit(10_000)
sys.getsizeof(obj)               # Memory size in bytes
```

### `datetime`

```python
from datetime import datetime, date, time, timedelta, timezone

# Current time
now = datetime.now()               # Local time
utc = datetime.now(timezone.utc)   # UTC
today = date.today()

# Create specific
dt = datetime(2024, 12, 25, 12, 0, 0)
d  = date(2024, 12, 25)
t  = time(12, 30, 45)

# Attributes
now.year, now.month, now.day
now.hour, now.minute, now.second, now.microsecond
now.weekday()     # 0=Monday ... 6=Sunday
now.isoweekday()  # 1=Monday ... 7=Sunday

# Arithmetic
delta = timedelta(days=7, hours=3, minutes=30)
next_week = now + delta
yesterday = now - timedelta(days=1)
duration = dt2 - dt1              # Returns timedelta

# Formatting & parsing
now.strftime("%Y-%m-%d %H:%M:%S")  # "2024-12-25 12:00:00"
now.strftime("%d/%m/%Y")           # "25/12/2024"
now.isoformat()                    # "2024-12-25T12:00:00"
datetime.fromisoformat("2024-12-25T12:00:00")
datetime.strptime("25/12/2024", "%d/%m/%Y")

# Timezone-aware
from zoneinfo import ZoneInfo      # Python 3.9+
tz = ZoneInfo("America/New_York")
dt = datetime(2024, 12, 25, 12, 0, tzinfo=tz)
dt_utc = dt.astimezone(timezone.utc)
```

### `re` — Regular Expressions

```python
import re

text = "My email is alice@example.com and bob@test.org"
pattern = r"[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}"

# Search — find first match
m = re.search(pattern, text)
if m:
    m.group()        # Full match
    m.start()        # Start index
    m.end()          # End index
    m.span()         # (start, end)

# Match — only matches at start of string
m = re.match(r"\d+", "42abc")
m.group()            # "42"

# Findall — list of all matches
emails = re.findall(pattern, text)   # ['alice@example.com', 'bob@test.org']

# Finditer — iterator of match objects
for m in re.finditer(pattern, text):
    print(m.group(), m.start())

# Sub — replace
cleaned = re.sub(r"\s+", " ", "too   many    spaces")    # "too many spaces"
masked  = re.sub(pattern, "***", text)                   # Mask emails

# Split
parts = re.split(r"\s*,\s*", "a, b,  c , d")   # ['a','b','c','d']

# Compile for reuse (faster if used many times)
email_re = re.compile(pattern)
email_re.findall(text)

# Groups
m = re.search(r"(\d{4})-(\d{2})-(\d{2})", "Date: 2024-12-25")
m.group(0)    # "2024-12-25" (full match)
m.group(1)    # "2024"
m.group(2)    # "12"
m.group(3)    # "25"
m.groups()    # ('2024', '12', '25')

# Named groups
m = re.search(r"(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})", "2024-12-25")
m.group("year")    # "2024"
m.groupdict()      # {'year': '2024', 'month': '12', 'day': '25'}

# Flags
re.search(pattern, text, re.IGNORECASE)
re.search(pattern, text, re.MULTILINE)   # ^ and $ match line boundaries
re.search(pattern, text, re.DOTALL)      # . matches newline too
re.search(pattern, text, re.VERBOSE)     # Allow comments in pattern
```

### `argparse` — CLI Arguments

```python
import argparse

parser = argparse.ArgumentParser(
    description="Process files",
    formatter_class=argparse.RawDescriptionHelpFormatter
)

# Positional argument
parser.add_argument("filename", help="Input file to process")

# Optional flags
parser.add_argument("-v", "--verbose", action="store_true", help="Verbose output")
parser.add_argument("-o", "--output", default="out.txt", help="Output file")
parser.add_argument("-n", "--count", type=int, default=10, help="Number of items")
parser.add_argument("--log-level", choices=["DEBUG","INFO","WARN","ERROR"],
                    default="INFO")

# Multiple values
parser.add_argument("files", nargs="+", help="Input files")
parser.add_argument("--tags", nargs="*", help="Optional tags")

# Mutually exclusive
group = parser.add_mutually_exclusive_group()
group.add_argument("--json", action="store_true")
group.add_argument("--csv",  action="store_true")

args = parser.parse_args()

if args.verbose:
    print(f"Processing {args.filename}")
```

### `logging`

```python
import logging

# Basic configuration
logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s %(name)s %(levelname)s %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
    filename="app.log",     # Log to file (omit for stdout)
    filemode="a",
)

# Logging levels: DEBUG < INFO < WARNING < ERROR < CRITICAL
logging.debug("Debug message")
logging.info("Info message")
logging.warning("Warning message")
logging.error("Error message")
logging.critical("Critical message")

# Named loggers (best practice in modules)
logger = logging.getLogger(__name__)
logger.info("Module logger")

# Exception logging
try:
    risky()
except Exception:
    logger.exception("Something went wrong")   # Includes traceback

# Structured logging setup
def setup_logging():
    logger = logging.getLogger("myapp")
    logger.setLevel(logging.DEBUG)

    handler = logging.StreamHandler()
    handler.setLevel(logging.INFO)

    formatter = logging.Formatter(
        "%(asctime)s [%(levelname)s] %(name)s: %(message)s"
    )
    handler.setFormatter(formatter)
    logger.addHandler(handler)

    return logger
```

---

## 20. Testing

### `unittest`

```python
import unittest

def add(a, b):
    return a + b

class TestAdd(unittest.TestCase):

    def setUp(self):
        """Runs before each test."""
        self.data = [1, 2, 3]

    def tearDown(self):
        """Runs after each test."""
        pass

    def test_positive(self):
        self.assertEqual(add(2, 3), 5)

    def test_negative(self):
        self.assertEqual(add(-1, -1), -2)

    def test_zero(self):
        self.assertEqual(add(0, 0), 0)

    def test_raises(self):
        with self.assertRaises(TypeError):
            add("a", 1)

    # Assertions
    # self.assertEqual(a, b)
    # self.assertNotEqual(a, b)
    # self.assertTrue(x)
    # self.assertFalse(x)
    # self.assertIs(a, b)            # Identity
    # self.assertIsNone(x)
    # self.assertIn(a, b)
    # self.assertIsInstance(obj, cls)
    # self.assertAlmostEqual(a, b, places=7)
    # self.assertRaises(Error, func, args)
    # self.assertWarns(Warning, func)

if __name__ == "__main__":
    unittest.main()
```

### `pytest` (recommended)

```bash
pip install pytest pytest-cov

pytest                         # Auto-discover and run tests
pytest tests/                  # Specific directory
pytest tests/test_math.py      # Specific file
pytest -v                      # Verbose output
pytest -k "add or subtract"    # Filter by name
pytest -m "slow"               # Run by marker
pytest --tb=short              # Short tracebacks
pytest -x                      # Stop on first failure
pytest --pdb                   # Drop into debugger on failure
pytest --cov=mypackage --cov-report=html   # Coverage report
```

```python
# test_math.py — pytest auto-discovers files matching test_*.py or *_test.py
import pytest

def add(a, b):
    return a + b

# No class needed — just functions starting with test_
def test_basic():
    assert add(2, 3) == 5

def test_negatives():
    assert add(-1, -1) == -2

# Parametrize — run same test with multiple inputs
@pytest.mark.parametrize("a,b,expected", [
    (1, 2, 3),
    (-1, 1, 0),
    (0, 0, 0),
    (100, -50, 50),
])
def test_add_parametrized(a, b, expected):
    assert add(a, b) == expected

# Fixtures — reusable setup
@pytest.fixture
def sample_data():
    return {"name": "Alice", "score": 95}

@pytest.fixture(scope="session")  # Created once per test session
def db_connection():
    conn = create_connection()
    yield conn                    # Code after yield is teardown
    conn.close()

def test_with_fixture(sample_data):
    assert sample_data["name"] == "Alice"

# Marks
@pytest.mark.slow
def test_performance():
    ...

@pytest.mark.skip(reason="Not implemented yet")
def test_future():
    ...

@pytest.mark.skipif(sys.platform == "win32", reason="Unix only")
def test_unix():
    ...

@pytest.mark.xfail(reason="Known bug")
def test_known_issue():
    ...

# Exception testing
def test_raises():
    with pytest.raises(ZeroDivisionError):
        1 / 0

def test_raises_message():
    with pytest.raises(ValueError, match="invalid"):
        raise ValueError("invalid input")

# Mocking
from unittest.mock import Mock, MagicMock, patch

def test_with_mock():
    mock_func = Mock(return_value=42)
    assert mock_func() == 42
    mock_func.assert_called_once()

def test_with_patch():
    with patch("mymodule.requests.get") as mock_get:
        mock_get.return_value.json.return_value = {"status": "ok"}
        result = my_api_call()
        assert result["status"] == "ok"

# Conftest.py — shared fixtures and plugins
# Create tests/conftest.py:
# @pytest.fixture(autouse=True)
# def reset_db():
#     setup_db()
#     yield
#     teardown_db()
```

---

## 21. Virtual Environments & Packaging

### Virtual Environments

```bash
# Create (built-in)
python3 -m venv .venv

# Activate
source .venv/bin/activate          # Linux/macOS
.venv\Scripts\activate             # Windows PowerShell
.venv\Scripts\activate.bat         # Windows CMD

# Deactivate
deactivate

# Which Python?
which python

# Install packages
pip install requests
pip install "requests>=2.28,<3"
pip install requests[security]     # With extras
pip install -e .                   # Install current project in editable mode
pip install -r requirements.txt    # From requirements file

# Manage requirements
pip freeze > requirements.txt
pip install -r requirements.txt

# Upgrade
pip install --upgrade requests
pip install --upgrade pip
```

### `pip` Essentials

```bash
pip list                           # List installed packages
pip show requests                  # Details about a package
pip check                          # Check for broken dependencies
pip install --user package         # Install for current user only
pip install --index-url https://...  # Custom index
pip uninstall requests
pip cache purge                    # Clear pip cache
```

### `pyproject.toml` — Modern Packaging

```toml
[build-system]
requires      = ["hatchling"]
build-backend = "hatchling.build"

[project]
name            = "mypackage"
version         = "1.0.0"
description     = "A great package"
readme          = "README.md"
requires-python = ">=3.10"
license         = { file = "LICENSE" }
authors         = [{ name = "Alice", email = "alice@example.com" }]
keywords        = ["example", "packaging"]

dependencies = [
    "requests>=2.28",
    "pydantic>=2.0",
]

[project.optional-dependencies]
dev  = ["pytest", "pytest-cov", "mypy", "ruff"]
docs = ["sphinx", "sphinx-rtd-theme"]

[project.scripts]
myapp = "mypackage.cli:main"        # Installs 'myapp' command

[project.urls]
Homepage = "https://github.com/alice/mypackage"

[tool.pytest.ini_options]
testpaths    = ["tests"]
addopts      = "-v --cov=mypackage"

[tool.mypy]
strict = true

[tool.ruff]
line-length = 88
select      = ["E", "F", "I"]      # pycodestyle, pyflakes, isort
```

```bash
# Build and publish
pip install build twine
python -m build           # Creates dist/*.whl and dist/*.tar.gz
twine check dist/*        # Validate
twine upload dist/*       # Upload to PyPI
twine upload --repository testpypi dist/*  # Test PyPI
```

### Poetry & uv (Modern Alternatives)

```bash
# uv — extremely fast package manager (Rust-based)
pip install uv

uv venv                     # Create venv
uv pip install requests     # Install package
uv pip sync requirements.txt
uv run pytest               # Run in venv

# Poetry — dependency management + packaging
pip install poetry

poetry new myproject        # Create project
poetry add requests         # Add dependency
poetry add --dev pytest     # Add dev dependency
poetry install              # Install all deps
poetry run pytest           # Run in venv
poetry build                # Build package
poetry publish              # Publish to PyPI
poetry shell                # Activate venv
```

---

## 22. Performance & Best Practices

### Profiling

```python
# cProfile — function-level profiling
import cProfile
cProfile.run("my_function()")

# From command line
python3 -m cProfile -s cumulative script.py

# timeit — micro-benchmarks
import timeit
timeit.timeit("sum(range(1000))", number=10_000)

# From command line
python3 -m timeit "sum(range(1000))"

# line_profiler (pip install line-profiler)
# Add @profile decorator, then:
# kernprof -l -v script.py

# memory_profiler (pip install memory-profiler)
# @memory_profiler.profile
```

### Performance Tips

```python
# 1. Use local variables in tight loops (local lookup is faster)
import math
def bad():
    for i in range(1000):
        x = math.sqrt(i)   # Attribute lookup every iteration

def good():
    sqrt = math.sqrt       # Cache the reference
    for i in range(1000):
        x = sqrt(i)

# 2. List comprehensions > loops for creating lists
bad  = []
for i in range(1000):
    bad.append(i**2)       # Slower — method lookup each time

good = [i**2 for i in range(1000)]   # Faster

# 3. Generator expressions for single-pass operations
total = sum(x**2 for x in range(1_000_000))  # No intermediate list

# 4. Join strings — never concatenate in a loop
bad  = ""
for s in strings:
    bad += s               # O(n²) — creates new string each time

good = "".join(strings)    # O(n)

# 5. Use collections for the right job
# list: O(n) search    → use set for membership test
# dict: O(1) lookup    → always prefer over nested if/elif
# deque: O(1) ends     → use instead of list for queue

# 6. __slots__ — reduce memory for many instances
class Point:
    __slots__ = ["x", "y"]    # No __dict__ per instance
    def __init__(self, x, y):
        self.x = x
        self.y = y

# 7. numpy for numerical computation
import numpy as np
arr = np.array([1, 2, 3, 4, 5])
arr * 2                            # Vectorized — much faster than list * 2

# 8. lru_cache for expensive pure functions
from functools import lru_cache
@lru_cache(maxsize=None)
def fib(n):
    if n < 2: return n
    return fib(n-1) + fib(n-2)
```

### Pythonic Code (The Zen)

```python
# Idiomatic Python: clear > clever

# 1. Use context managers for resources
with open("file") as f: ...

# 2. Enumerate instead of range(len())
for i, item in enumerate(lst): ...

# 3. zip instead of indexing
for a, b in zip(list_a, list_b): ...

# 4. Unpack tuples
x, y = point
first, *rest = items

# 5. Dictionary .get() with default
value = d.get("key", "default")

# 6. Truthiness — don't compare to True/False/None
if items: ...              # Not: if len(items) > 0
if not name: ...           # Not: if name == ""
if x is None: ...          # Test for None explicitly
if x is not None: ...

# 7. Any/all
if any(x > 0 for x in nums): ...
if all(isinstance(x, int) for x in items): ...

# 8. Walrus for assign-and-test
if (n := compute()) > 0: ...

# 9. Dataclasses over plain dicts for structured data
@dataclass
class Config: ...

# 10. Exceptions for control flow (EAFP vs LBYL)
# LBYL (Look Before You Leap)
if key in d:
    value = d[key]

# EAFP (Easier to Ask Forgiveness than Permission) — more Pythonic
try:
    value = d[key]
except KeyError:
    value = default
# Or simply: value = d.get(key, default)
```

### Code Quality Tools

```bash
# Formatter
pip install black ruff
black .                     # Format all files
ruff format .               # Ruff's formatter (faster)

# Linter
ruff check .                # Fast linter (replaces flake8 + isort + more)
flake8 .                    # Classic linter

# Type checker
mypy .

# Security scanner
pip install bandit
bandit -r .

# All-in-one pre-commit hook
pip install pre-commit
# Create .pre-commit-config.yaml:
# repos:
# - repo: https://github.com/astral-sh/ruff-pre-commit
#   hooks: [{id: ruff}, {id: ruff-format}]
pre-commit install
pre-commit run --all-files
```

---

## Quick Reference Card

### Built-in Functions

| Function | Description |
|----------|-------------|
| `print(*args, sep, end, file)` | Print to stdout |
| `len(obj)` | Length of sequence |
| `range(start, stop, step)` | Integer range |
| `type(obj)` | Get type |
| `isinstance(obj, type)` | Check type |
| `int/float/str/bool(x)` | Type conversion |
| `list/tuple/set/dict(x)` | Collection conversion |
| `sorted(iterable, key, reverse)` | Return sorted list |
| `reversed(seq)` | Reverse iterator |
| `enumerate(iterable, start)` | Index + value pairs |
| `zip(*iterables)` | Pair up iterables |
| `map(func, iterable)` | Apply func to each |
| `filter(func, iterable)` | Keep truthy results |
| `any(iterable)` | True if any truthy |
| `all(iterable)` | True if all truthy |
| `min/max(iterable, key)` | Minimum / maximum |
| `sum(iterable, start)` | Sum values |
| `abs(x)` | Absolute value |
| `round(x, n)` | Round to n decimals |
| `pow(base, exp, mod)` | Power |
| `divmod(a, b)` | (quotient, remainder) |
| `hash(obj)` | Hash value |
| `id(obj)` | Object identity |
| `dir(obj)` | List attributes |
| `vars(obj)` | Object's `__dict__` |
| `help(obj)` | Help text |
| `open(file, mode)` | Open file |
| `input(prompt)` | Read from stdin |
| `repr(obj)` | Developer string |
| `getattr/setattr/hasattr/delattr` | Attribute access |
| `callable(obj)` | Is callable? |
| `iter(obj)` / `next(it)` | Iterator protocol |
| `super()` | Parent class |
| `staticmethod/classmethod/property` | Method decorators |
| `__import__(name)` | Import module |

### Common Patterns Cheatsheet

```python
# Swap
a, b = b, a

# Ternary
x = a if condition else b

# Unpack first/last
first, *_, last = items

# Safe dict access
value = d.get("key", default)

# Count occurrences
from collections import Counter
c = Counter(items)

# Flatten list
flat = [x for sub in nested for x in sub]

# Chunk a list
chunks = [lst[i:i+n] for i in range(0, len(lst), n)]

# Remove duplicates (preserving order)
seen = set()
unique = [x for x in lst if not (x in seen or seen.add(x))]

# Sort dict by value
sorted_d = dict(sorted(d.items(), key=lambda x: x[1]))

# Transpose a matrix
transposed = list(zip(*matrix))

# Merge dicts
merged = {**dict1, **dict2}

# Read file lines stripped
lines = Path("file.txt").read_text().splitlines()

# Run as script guard
if __name__ == "__main__":
    main()
```

### Python Version Features

| Version | Key Addition |
|---------|-------------|
| 3.6 | f-strings, `__init_subclass__` |
| 3.7 | Dataclasses, `breakpoint()`, ordered dicts |
| 3.8 | Walrus operator `:=`, positional-only params `/`, `f"{x=}"` |
| 3.9 | `list[int]` generic syntax, `dict \| dict`, `str.removeprefix/suffix` |
| 3.10 | `match`/`case` structural pattern matching, `X \| Y` union types |
| 3.11 | Exception groups, `tomllib`, faster CPython (~60% speedup) |
| 3.12 | `@override` decorator, type aliases, improved error messages |
| 3.13 | JIT compiler (experimental), free-threaded mode (no GIL option) |
