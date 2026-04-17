# Bit Manipulation: A Complete Progressive Tutorial

---

## 1. What & Why

Bit manipulation is the practice of operating directly on the binary representation of integers using bitwise operators. Rather than working with numbers as abstract values, you work with their individual bits — the 1s and 0s that represent them in hardware.

Why does this matter? Three reasons. First, speed: bitwise operations execute in a single CPU cycle. A right-shift by 1 is faster than dividing by 2. A bitwise AND is faster than a modulo check. In tight inner loops, this matters. Second, space: a single 64-bit integer can store 64 boolean flags — what would otherwise require 64 separate variables or a 64-element boolean array. Third, necessity: systems programming, cryptography, compression, graphics, networking, and device drivers all require operating at the bit level. You cannot write a TCP/IP stack, implement AES, or build a Huffman encoder without bit manipulation.

When you see code like `x & (x - 1)` or `n ^ (n >> 31)` in production systems, you need to be able to read it without hesitation.

---

## 2. Mental Model

Think of an integer as a row of light switches — each switch is either on (1) or off (0). The switches are numbered from right to left starting at 0. Switch 0 (rightmost) has a place value of 1, switch 1 has a value of 2, switch 2 has a value of 4, and so on — each doubling.

```
Integer 42 in 8-bit binary:

Position:  7   6   5   4   3   2   1   0
Value:    128  64  32  16   8   4   2   1

Bits:      0   0   1   0   1   0   1   0
           OFF OFF ON  OFF ON  OFF ON  OFF

Sum of ON positions: 32 + 8 + 2 = 42 ✓
```

Bitwise operators treat two integers as two rows of switches and perform operations switch-by-switch simultaneously. AND: both switches ON → result ON. OR: either switch ON → result ON. XOR: exactly one switch ON → result ON.

---

## 3. Progressive Examples

### Level 1: The Six Operators

```python
a = 0b1010   # 10 in decimal
b = 0b1100   # 12 in decimal

# AND (&): 1 only when BOTH bits are 1
print(bin(a & b))   # 0b1000 = 8
# 1010
# 1100
# ─────
# 1000  (only the position where both had 1)

# OR (|): 1 when EITHER bit is 1
print(bin(a | b))   # 0b1110 = 14
# 1010
# 1100
# ─────
# 1110  (1 wherever either had 1)

# XOR (^): 1 when bits are DIFFERENT
print(bin(a ^ b))   # 0b0110 = 6
# 1010
# 1100
# ─────
# 0110  (1 only where they differ)

# NOT (~): flip all bits (in Python: ~n = -(n+1))
print(~a)   # -11
# For 8-bit: ~00001010 = 11110101 = -11 in two's complement

# Left shift (<<): multiply by 2 per shift
print(1 << 3)    # 8  (1 moved left 3 positions: 0b1000)
print(5 << 2)    # 20 (5 × 4)

# Right shift (>>): divide by 2 per shift (floor division)
print(16 >> 1)   # 8  (16 ÷ 2)
print(17 >> 1)   # 8  (17 ÷ 2, floor)
print(-8 >> 1)   # -4 (arithmetic right shift — preserves sign in Python)
```

### Level 2: The Core Bit Operations (The Building Blocks)

```python
# These patterns appear everywhere. Memorize them.

n = 0b10110100   # example value

# 1. CHECK if bit k is set
def is_bit_set(n, k):
    return bool(n & (1 << k))
    # (1 << k) creates a mask with only bit k set
    # n & mask: nonzero iff bit k in n is 1

print(is_bit_set(n, 2))   # True  (bit 2 is 1 in 10110100)
print(is_bit_set(n, 3))   # False (bit 3 is 0)

# 2. SET bit k (turn it on)
def set_bit(n, k):
    return n | (1 << k)    # OR with a mask that has only bit k set
    # Guarantees bit k becomes 1; all other bits unchanged

print(bin(set_bit(0b10100, 1)))   # 0b10110 (bit 1 turned on)

# 3. CLEAR bit k (turn it off)
def clear_bit(n, k):
    return n & ~(1 << k)   # AND with a mask that has only bit k = 0
    # ~(1 << k) flips all bits: bit k becomes 0, all others become 1
    # AND preserves all bits except bit k, which becomes 0

print(bin(clear_bit(0b10110, 1)))  # 0b10100 (bit 1 turned off)

# 4. TOGGLE bit k (flip it)
def toggle_bit(n, k):
    return n ^ (1 << k)    # XOR: 1^1=0 (flip on→off), 0^1=1 (flip off→on)

print(bin(toggle_bit(0b10110, 1)))  # 0b10100 (bit 1: 1 → 0)
print(bin(toggle_bit(0b10100, 1)))  # 0b10110 (bit 1: 0 → 1)

# 5. EXTRACT bit k value (0 or 1)
def get_bit(n, k):
    return (n >> k) & 1    # shift bit k to position 0, then mask it

# 6. UPDATE bit k to value v (0 or 1)
def update_bit(n, k, v):
    return (n & ~(1 << k)) | (v << k)
    # Clear bit k, then OR in the new value

# 7. EXTRACT multiple bits: get bits from position lo to hi (inclusive)
def extract_bits(n, lo, hi):
    mask = (1 << (hi - lo + 1)) - 1   # mask of (hi-lo+1) ones
    return (n >> lo) & mask

n = 0b11011010
print(bin(extract_bits(n, 3, 5)))   # bits 3,4,5 = 0b011 = 3
```

### Level 3: Classic Bit Tricks

```python
# These are real techniques used in production code and competitive programming.

# --- Power of 2 checks ---
def is_power_of_two(n):
    """True if n is a positive power of 2. O(1)."""
    return n > 0 and (n & (n - 1)) == 0
    # Powers of 2 in binary: 1000, 10000, etc. — exactly ONE bit set
    # n-1 flips the set bit and all bits below it: 1000 → 0111
    # n & (n-1) = 0 iff exactly one bit was set

for x in [0, 1, 2, 3, 4, 7, 8, 16, 17]:
    print(f"{x}: {is_power_of_two(x)}")

# --- Count set bits (popcount) ---
def popcount(n):
    """Count the number of 1-bits in n."""
    # Brian Kernighan's algorithm: n & (n-1) removes the lowest set bit
    count = 0
    while n:
        n &= n - 1    # clear the lowest set bit
        count += 1
    return count

print(popcount(0b10110110))   # 5

# Python has this built-in:
print(bin(255).count('1'))    # 8
print(255 .bit_count())       # 8 (Python 3.10+)

# --- Lowest set bit ---
def lowest_set_bit(n):
    """Return a mask with only the lowest set bit."""
    return n & (-n)   # two's complement trick

n = 0b10110100
print(bin(lowest_set_bit(n)))   # 0b100 (bit 2)
# -n in two's complement flips all bits and adds 1
# n & (-n) isolates the lowest set bit

# --- Clear lowest set bit ---
def clear_lowest_bit(n):
    return n & (n - 1)   # n-1 flips the lowest set bit and everything below it

# --- Swap without a temp variable ---
def swap_bits(a, b):
    a ^= b    # a now holds a XOR b
    b ^= a    # b now holds original a (b XOR (a XOR b) = original a)
    a ^= b    # a now holds original b
    return a, b

x, y = 10, 25
x, y = swap_bits(x, y)
print(x, y)   # 25, 10

# Note: Python can do x, y = y, x in one line — this trick matters in C/assembly.

# --- Single number (XOR eliminates pairs) ---
def find_single(arr):
    """Find the element that appears once; all others appear twice. O(n) time, O(1) space."""
    result = 0
    for n in arr:
        result ^= n    # XOR is commutative and associative; x^x=0, x^0=x
    return result      # pairs cancel out, leaving the single element

print(find_single([2, 3, 5, 3, 2]))   # 5
# 2^3^5^3^2 = (2^2)^(3^3)^5 = 0^0^5 = 5

# --- Missing number in [0..n] ---
def find_missing(arr, n):
    """Find the missing number in [0, 1, ..., n]. O(n) time, O(1) space."""
    expected_xor = 0
    for i in range(n + 1):
        expected_xor ^= i
    actual_xor = 0
    for num in arr:
        actual_xor ^= num
    return expected_xor ^ actual_xor
    # expected XOR actual = the missing number (everything else cancels)

print(find_missing([0, 1, 3, 4], 4))   # 2
```

### Level 4: Bitmasks and Bit Arrays

```python
# Bit array: represent a set of integers using bits in a single integer
# 64-bit integer → can represent any subset of {0, 1, ..., 63}

class BitSet:
    """
    Space-efficient set for non-negative integers using bit manipulation.
    64x more space-efficient than a list of booleans for small universes.
    """

    def __init__(self):
        self._bits = 0

    def add(self, n):
        self._bits |= (1 << n)

    def remove(self, n):
        self._bits &= ~(1 << n)

    def contains(self, n):
        return bool(self._bits & (1 << n))

    def union(self, other):
        result = BitSet()
        result._bits = self._bits | other._bits
        return result

    def intersection(self, other):
        result = BitSet()
        result._bits = self._bits & other._bits
        return result

    def difference(self, other):
        result = BitSet()
        result._bits = self._bits & ~other._bits
        return result

    def size(self):
        return bin(self._bits).count('1')

    def __contains__(self, n):
        return self.contains(n)

    def __repr__(self):
        return f"BitSet({[i for i in range(64) if self.contains(i)]})"

s1 = BitSet()
s1.add(3); s1.add(5); s1.add(7)
s2 = BitSet()
s2.add(5); s2.add(7); s2.add(9)

print(s1)                     # BitSet([3, 5, 7])
print(s1.union(s2))           # BitSet([3, 5, 7, 9])
print(s1.intersection(s2))    # BitSet([5, 7])
print(s1.difference(s2))      # BitSet([3])

# --- Enumerate all subsets using bitmask ---
def all_subsets(items):
    """Generate all 2^n subsets using bitmasks."""
    n = len(items)
    subsets = []
    for mask in range(1 << n):    # 0 to 2^n - 1
        subset = [items[i] for i in range(n) if mask & (1 << i)]
        subsets.append(subset)
    return subsets

print(all_subsets(['a', 'b', 'c']))
# [[], ['a'], ['b'], ['a', 'b'], ['c'], ['a', 'c'], ['b', 'c'], ['a', 'b', 'c']]

# --- File permissions (Unix-style with bitmasks) ---
# Unix permissions: owner(rwx) | group(rwx) | others(rwx)
OWNER_READ  = 0o400   # 0b 100 000 000
OWNER_WRITE = 0o200   # 0b 010 000 000
OWNER_EXEC  = 0o100   # 0b 001 000 000
GROUP_READ  = 0o040
GROUP_WRITE = 0o020
GROUP_EXEC  = 0o010
OTHER_READ  = 0o004
OTHER_WRITE = 0o002
OTHER_EXEC  = 0o001

perm_755 = OWNER_READ | OWNER_WRITE | OWNER_EXEC | GROUP_READ | GROUP_EXEC | OTHER_READ | OTHER_EXEC
print(oct(perm_755))   # 0o755
print(bool(perm_755 & OWNER_WRITE))   # True
print(bool(perm_755 & GROUP_WRITE))   # False
```

### Level 5: Advanced Applications

```python
# --- Arithmetic with bit shifts ---
def multiply_by_power_of_two(n, k):
    return n << k    # n * 2^k

def divide_by_power_of_two(n, k):
    return n >> k    # n // 2^k

def is_odd(n):
    return bool(n & 1)   # check lowest bit

def abs_without_branch(n):
    """Absolute value using arithmetic right shift — branchless."""
    mask = n >> 31    # all 1s if negative, all 0s if positive
    return (n + mask) ^ mask
    # For negative n: mask = -1 = 0xFFFFFFFF
    #   (n + (-1)) ^ (-1) = (n-1) ^ 0xFFFFFFFF = flip all bits of (n-1) = ~(n-1) = -n

print(abs_without_branch(-7))   # 7
print(abs_without_branch(5))    # 5

# --- Reverse bits in a 32-bit integer ---
def reverse_bits(n):
    result = 0
    for _ in range(32):
        result = (result << 1) | (n & 1)   # shift result left, add LSB of n
        n >>= 1                             # shift n right to get next bit
    return result

print(bin(reverse_bits(0b10110001)))   # 0b10001101 (reversed)

# --- Gray code: consecutive values differ by exactly one bit ---
# Used in hardware to prevent spurious outputs during state transitions
def to_gray(n):
    return n ^ (n >> 1)

def from_gray(g):
    n = 0
    while g:
        n ^= g
        g >>= 1
    return n

for i in range(8):
    g = to_gray(i)
    print(f"{i:3d} binary={bin(i):>10s}  gray={bin(g):>10s}  back={from_gray(g)}")

# --- Bit manipulation in competitive programming: subset enumeration ---
def sum_over_subsets(n, values):
    """
    For each subset, compute the OR of all values in the subset.
    Common in DP problems.
    """
    for mask in range(1 << n):
        subset_or = 0
        for i in range(n):
            if mask & (1 << i):
                subset_or |= values[i]
        # Process subset_or for this mask...

# Enumerate all subsets of a given mask (faster than nested loops)
def enumerate_submasks(mask):
    """All subsets of a bitmask, from mask down to 0. O(3^n) total across all masks."""
    sub = mask
    while sub > 0:
        yield sub
        sub = (sub - 1) & mask    # the magic: go to next smaller submask
    yield 0   # the empty subset
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Operator precedence — bitwise operators have lower precedence than comparisons**

```python
# WRONG: this evaluates as (n & 1) == 0, not n & (1 == 0)
if n & 1 == 0:      # actually works here by accident
    print("even")

# DANGEROUS CASE:
x = 5 & 3 == 1     # evaluates as 5 & (3 == 1) = 5 & False = 5 & 0 = 0
                    # NOT as (5 & 3) == 1

# CORRECT: always parenthesize bitwise expressions
if (n & 1) == 0:
    print("even")

x = (5 & 3) == 1   # True (5 & 3 = 1, 1 == 1)
```

**Mistake 2: Python's `~` is not a bitmask flip for unsigned values**

```python
# In C with uint8_t: ~0 = 0xFF = 255
# In Python: ~0 = -1  (integers are arbitrary precision in Python)

# WRONG: expecting ~n to give unsigned complement
n = 0b10110100   # 180
print(~n)        # -181, not 0b01001011 = 75

# CORRECT: mask to the desired bit width
def bitwise_not_8bit(n):
    return ~n & 0xFF

print(bitwise_not_8bit(0b10110100))   # 0b01001011 = 75

# For 32-bit: mask with 0xFFFFFFFF
# For 64-bit: mask with 0xFFFFFFFFFFFFFFFF
```

**Mistake 3: Using `>>` for signed right-shift when you want unsigned**

```python
# Python's >> is arithmetic right shift (fills with sign bit)
print(-8 >> 1)    # -4 (preserves sign — this is usually what you want)

# In C, right-shifting negative numbers is implementation-defined.
# In Java, >>> is logical right shift (fills with 0).
# In Python, use masking if you need unsigned behavior:

def unsigned_right_shift(n, k, bits=32):
    mask = (1 << bits) - 1
    return (n & mask) >> k

print(unsigned_right_shift(-8, 1, 32))   # 2147483644 (fills with 0s, not 1s)
```

**Mistake 4: `x & x - 1` — forgetting operator precedence**

```python
# This is (x & x) - 1 = x - 1, NOT x & (x-1)
result = x & x - 1      # WRONG

# CORRECT:
result = x & (x - 1)    # clears the lowest set bit
```

**Mistake 5: XOR swap can fail when both variables alias the same memory**

```python
# WRONG: XOR swap with the same variable destroys the value
def xor_swap_broken(arr, i, j):
    if i == j:  # MUST check this!
        return
    arr[i] ^= arr[j]
    arr[j] ^= arr[i]
    arr[i] ^= arr[j]

arr = [1, 2, 3]
xor_swap_broken(arr, 1, 1)   # Without the guard: arr[1] = 0!
# arr[1] ^= arr[1] → 0 (x XOR x = 0)
# arr[1] ^= 0 → 0
# arr[1] ^= 0 → 0
```

---

## 5. The "Why Does This Work" Layer

### Why n & (n-1) Clears the Lowest Set Bit

Consider n with its lowest set bit at position k:
```
n     = ...1 0...0  (1 at position k, zeros below)
n - 1 = ...0 1...1  (borrow propagates: position k becomes 0, all below become 1)
n & (n-1) = ...0 0...0  (AND: position k differs, everything below differs)
```
The bits above position k are unchanged (subtraction didn't affect them). The bit at position k and all bits below it become 0. This is why looping `n &= n-1` and counting iterations gives popcount in O(set_bits) time — one iteration per set bit.

### Why n ^ (n-1) Gives a Mask of the Lowest Set Bit and All Bits Below It

```
n       = 10110100  (lowest set bit at position 2)
n - 1   = 10110011  (bits at and below position 2 flipped)
n ^ (n-1) = 00000111  (XOR: bits that differ = position 2 and below)
```

This gives a mask covering the lowest set bit and everything below. Useful for isolating ranges of bits.

### Why XOR Can Find the Single Non-Duplicate

XOR has three properties that make this work: commutativity (a^b = b^a), associativity ((a^b)^c = a^(b^c)), and self-inverse (a^a = 0 for any a, and a^0 = a).

When you XOR all elements: pairs cancel (x^x = 0), and the single element survives (x^0 = x). The order doesn't matter because XOR is commutative and associative.

```
2 ^ 3 ^ 5 ^ 3 ^ 2
= (2 ^ 2) ^ (3 ^ 3) ^ 5    (reorder — commutative)
= 0 ^ 0 ^ 5                 (self-inverse)
= 5                          (identity)
```

---

## 6. Quick Reference

### Operators

| Operator | Symbol | Rule |
|----------|--------|------|
| AND | `&` | 1 only when both are 1 |
| OR | `\|` | 1 when either is 1 |
| XOR | `^` | 1 only when they differ |
| NOT | `~` | Flip all bits (Python: ~n = -(n+1)) |
| Left shift | `<<` | Multiply by 2 per position |
| Right shift | `>>` | Integer divide by 2 per position |

### Core Patterns

```python
# Test bit k
n & (1 << k)          # nonzero if set

# Set bit k
n | (1 << k)

# Clear bit k
n & ~(1 << k)

# Toggle bit k
n ^ (1 << k)

# Clear lowest set bit
n & (n - 1)

# Isolate lowest set bit
n & (-n)

# Check power of 2
n > 0 and (n & (n-1)) == 0

# Check even/odd
n & 1  # 0 = even, 1 = odd

# Multiply/divide by 2^k
n << k  # multiply
n >> k  # divide (floor)

# XOR trick: a^a=0, a^0=a
# Find single element in pairs: XOR all elements
# Swap: a^=b; b^=a; a^=b

# Popcount (count set bits)
bin(n).count('1')      # Python string method
n.bit_count()          # Python 3.10+
```

### Useful Masks

```python
0x1         # 1 — select bit 0
0xFF        # 255 — select lowest byte
0xFFFF      # 65535 — select lowest 16 bits
0xFFFFFFFF  # 4294967295 — select lowest 32 bits
(1 << k) - 1  # k ones (mask for k-bit field)
~((1 << k) - 1)  # mask for upper bits above k
```
