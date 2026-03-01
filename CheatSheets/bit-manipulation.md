# ⚙️ The Complete Bit Manipulation Cheat Sheet
### From Zero to Expert

---

## 1. Why Bit Manipulation?

Bit manipulation is the direct use of bitwise operators to solve problems at the level of individual binary digits. It is one of the most powerful tools in a programmer's toolbox.

**Why learn it?**

```
Speed:   Bitwise ops execute in a single CPU cycle — faster than
         multiplication, division, and especially branches.

Space:   Pack 64 booleans into one 64-bit integer instead of
         64 separate bytes (64x memory savings).

Elegance: Many problems that look complex have a 1-line bit solution.

Necessity: Systems programming, cryptography, compression, graphics,
           networking, and competitive programming require it.
```

**Real-world appearances:**
- Linux kernel: bitflags for file permissions, process states, hardware registers
- Cryptography: AES, SHA, RSA all rely on XOR and shifts heavily
- Graphics: pixel blending, color channel extraction, alpha compositing
- Networking: subnet masks, IP address manipulation, checksums
- Compression: Huffman coding, LZ77 use bit-level I/O
- Games: chess engines store board state as 64-bit integers (bitboards)
- Compilers: constant folding replaces `x * 4` with `x << 2`

---

## 2. Bit Fundamentals Refresher

### Every Integer is a Bit String

```
Decimal  42  in memory (8-bit):

Bit position:  7   6   5   4   3   2   1   0
               │   │   │   │   │   │   │   │
Bit value:     0   0   1   0   1   0   1   0
               │   │   │   │   │   │   │   │
Place value:  128  64  32  16   8   4   2   1

Value = 0+0+32+0+8+0+2+0 = 42  ✅

MSB (Most Significant Bit)  = leftmost  = highest place value
LSB (Least Significant Bit) = rightmost = place value 1
```

### Bit Indexing Convention

```
Bits are numbered from 0 (rightmost/LSB) upward:

  bit 7   bit 6   bit 5   bit 4   bit 3   bit 2   bit 1   bit 0
  ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐
  │ 0 │   │ 0 │   │ 1 │   │ 0 │   │ 1 │   │ 0 │   │ 1 │   │ 0 │
  └───┘   └───┘   └───┘   └───┘   └───┘   └───┘   └───┘   └───┘
```

### Key Vocabulary

| Term | Meaning |
|------|---------|
| Bit | Single binary digit (0 or 1) |
| Nibble | 4 bits (one hex digit) |
| Byte | 8 bits |
| Word | Native integer size (typically 32 or 64 bits) |
| MSB | Most Significant Bit (leftmost, highest value) |
| LSB | Least Significant Bit (rightmost, value = 1) |
| Set bit | A bit with value 1 |
| Clear bit | A bit with value 0 |
| Popcount | Count of set bits (also called Hamming weight) |
| Bitmask | A value used to select or modify specific bits |
| Toggle | Flip a bit: 0→1, 1→0 |
| Sign bit | MSB in signed integers (0=positive, 1=negative) |

---

## 3. The Six Bitwise Operators

### AND — `&`

**Rule:** Output bit is `1` only when **both** inputs are `1`.

```
Truth table:         Mnemonic: "BOTH must be 1"
  0 & 0 = 0
  0 & 1 = 0
  1 & 0 = 0
  1 & 1 = 1    ← only case that produces 1

Example:
  0101 1100   (92)
& 0011 1111   (63)
───────────
  0001 1100   (28)

Bit-by-bit:  0&0=0  1&0=0  0&1=0  1&1=1  1&1=1  1&1=1  0&1=0  0&1=0
                                    ↑ only 1s where both had 1s
```

**Primary uses:** Masking (zeroing out bits), testing if a bit is set, extracting a field.

---

### OR — `|`

**Rule:** Output bit is `1` when **at least one** input is `1`.

```
Truth table:         Mnemonic: "EITHER can be 1"
  0 | 0 = 0
  0 | 1 = 1
  1 | 0 = 1
  1 | 1 = 1    ← both 1 also gives 1

Example:
  0101 1100   (92)
| 0011 0000   (48)
───────────
  0111 1100   (124)

Bit-by-bit: forces 1s wherever the mask has 1s
```

**Primary uses:** Setting bits (forcing to 1), combining flags.

---

### XOR — `^`

**Rule:** Output bit is `1` when inputs are **different**.

```
Truth table:         Mnemonic: "EXACTLY ONE must be 1"
  0 ^ 0 = 0
  0 ^ 1 = 1
  1 ^ 0 = 1
  1 ^ 1 = 0    ← same inputs cancel out!

Example:
  0101 1100   (92)
^ 0011 1111   (63)
───────────
  0110 0011   (99)

Key insight: a ^ a = 0   (self-cancellation)
             a ^ 0 = a   (identity)
             a ^ b ^ a = b  (undo XOR by XORing again)
```

**Primary uses:** Toggle bits, detect differences, swap without temp var, cryptography, finding the unique element.

---

### NOT — `~`

**Rule:** Flip every single bit.

```
~0101 1100  (92)
 1010 0011  (163 unsigned, or -93 in two's complement)

In two's complement:  ~n = -(n + 1)
~0  = -1
~1  = -2
~5  = -6
~(-1) = 0
```

**Primary uses:** Creating inverse masks, computing two's complement negation.

---

### Left Shift — `<<`

**Rule:** Shift all bits left by `n` positions, fill right side with `0`s. Bits shifted off the left are lost.

```
0000 0011  (3)
<< 3
0001 1000  (24)   =   3 × 2³ = 3 × 8 = 24  ✅

Bit movement:
  Original:  0 0 0 0  0 0 1 1
             ← ← ← ←   (shift 3 left, fill 3 zeros on right)
  Result:    0 0 0 1  1 0 0 0

Rule: x << n  =  x × 2ⁿ   (when no overflow)
```

**Primary uses:** Multiply by powers of 2, create bit masks (`1 << k`), pack values.

---

### Right Shift — `>>`

**Rule:** Shift all bits right by `n` positions. What fills the left side depends on the type:

```
LOGICAL right shift (unsigned integers):
  Fill with 0s regardless of sign.
  1000 0100  (132)
  >> 2
  0010 0001  (33)    =  132 / 4 = 33  ✅

ARITHMETIC right shift (signed integers):
  Fill with the sign bit (MSB) to preserve sign.
  1000 0100  (-124 in two's complement)
  >> 2
  1110 0001  (-31)   =  ⌊-124 / 4⌋ = -31  ✅

Rule: x >> n  =  ⌊x / 2ⁿ⌋   (arithmetic shift for signed)
```

**Primary uses:** Divide by powers of 2, extract bit fields, arithmetic floor division.

---

### Operator Precedence (Critical!)

```
Precedence (highest to lowest):
  ~         (NOT — unary, highest)
  << >>     (shifts)
  &         (AND)
  ^         (XOR)
  |         (OR — lowest of bitwise)

Comparison operators (==, !=, <, >) are HIGHER precedence than & | ^

⚠️ This is a famous bug source:
  if (x & 0xFF == 0)   // BUG! Evaluates as x & (0xFF == 0) → x & 0
  if ((x & 0xFF) == 0) // CORRECT — always use parentheses!
```

---

## 4. The Building Blocks — Core Operations

These are the fundamental patterns everything else is built from. Memorize them.

### The Four Atomic Bit Operations

```
Given a number n and bit position k (0-indexed from right):

1. GET bit k:      (n >> k) & 1
   Returns 1 if bit k is set, 0 if clear.

2. SET bit k:      n | (1 << k)
   Forces bit k to 1, leaves all others unchanged.

3. CLEAR bit k:    n & ~(1 << k)
   Forces bit k to 0, leaves all others unchanged.

4. TOGGLE bit k:   n ^ (1 << k)
   Flips bit k, leaves all others unchanged.
```

### Visualized on n = 0b10110100 (180), k = 2

```
n     = 1 0 1 1 0 1 0 0
                  ↑ bit 2

1 << 2           = 0 0 0 0 0 1 0 0

GET:   (n >> 2) & 1
       n >> 2    = 0 0 1 0 1 1 0 1
       & 1       = 0 0 0 0 0 0 0 1  → result = 1 (bit 2 is set)

SET:   n | (1 << 2)
       1 0 1 1 0 1 0 0
     | 0 0 0 0 0 1 0 0
       ───────────────
       1 0 1 1 0 1 0 0   (no change — already set)

CLEAR: n & ~(1 << 2)
       ~(1 << 2) = 1 1 1 1 1 0 1 1
       1 0 1 1 0 1 0 0
     & 1 1 1 1 1 0 1 1
       ───────────────
       1 0 1 1 0 0 0 0   (bit 2 cleared) = 176

TOGGLE: n ^ (1 << 2)
        1 0 1 1 0 1 0 0
      ^ 0 0 0 0 0 1 0 0
        ───────────────
        1 0 1 1 0 0 0 0   (bit 2 flipped 1→0) = 176
```

---

## 5. Reading & Writing Individual Bits

### Getting a Bit

```python
def get_bit(n, k):
    return (n >> k) & 1

# Examples:
get_bit(0b1010, 1)  # → 1  (bit 1 is set)
get_bit(0b1010, 0)  # → 0  (bit 0 is clear)
get_bit(0b1010, 3)  # → 1  (bit 3 is set)
```

### Setting a Bit

```python
def set_bit(n, k):
    return n | (1 << k)

# Examples:
set_bit(0b1010, 0)  # → 0b1011 = 11  (bit 0 set)
set_bit(0b1010, 2)  # → 0b1110 = 14  (bit 2 set)
set_bit(0b1010, 1)  # → 0b1010 = 10  (already set, no change)
```

### Clearing a Bit

```python
def clear_bit(n, k):
    return n & ~(1 << k)

# Examples:
clear_bit(0b1111, 2)  # → 0b1011 = 11  (bit 2 cleared)
clear_bit(0b1010, 0)  # → 0b1010 = 10  (already clear, no change)
```

### Toggling a Bit

```python
def toggle_bit(n, k):
    return n ^ (1 << k)

# Examples:
toggle_bit(0b1010, 0)  # → 0b1011 = 11  (0→1)
toggle_bit(0b1010, 1)  # → 0b1000 =  8  (1→0)
```

### Updating a Bit to a Specific Value

```python
def update_bit(n, k, value):
    # value must be 0 or 1
    return (n & ~(1 << k)) | (value << k)
    # First clear bit k, then OR in the desired value

# Step by step:
# 1. ~(1 << k) creates mask with all 1s except position k
# 2. n & mask clears bit k (to 0)
# 3. value << k shifts the desired bit to position k
# 4. OR combines them

update_bit(0b1010, 1, 0)  # → 0b1000 = 8   (set bit 1 to 0)
update_bit(0b1010, 0, 1)  # → 0b1011 = 11  (set bit 0 to 1)
```

### Reading a Range of Bits (Bit Field Extraction)

```python
def get_bits(n, start, length):
    """Extract 'length' bits starting at position 'start'."""
    mask = (1 << length) - 1   # e.g., length=3 → mask = 0b111
    return (n >> start) & mask

# Example: extract bits 2..4 from 0b11010110 (214)
#          214 = 1 1 0 1 0 1 1 0
#                        ↑↑↑  bits 2,3,4 = 0,1,0 = value 010 = 2 (then reversed)
# Actually: (214 >> 2) & 0b111 = 53 & 7 = 0b110101 & 0b000111 = 0b000101 = 5

get_bits(0b11010110, 2, 3)  # → 5  (bits 4,3,2 = 1,0,1)
get_bits(0xFF00, 8, 8)      # → 255  (upper byte of 16-bit value)
get_bits(0xFF00, 0, 8)      # → 0    (lower byte)
```

### Setting a Range of Bits

```python
def set_bits(n, start, length, value):
    """Set 'length' bits starting at 'start' to 'value'."""
    mask = ((1 << length) - 1) << start  # mask for the field
    return (n & ~mask) | ((value << start) & mask)

# Example: set bits 4-7 of 0b00000000 to 0b1010
set_bits(0x00, 4, 4, 0xA)  # → 0b10100000 = 0xA0 = 160
```

---

## 6. Bit Masks — The Swiss Army Knife

A **bitmask** is a value used with a bitwise operator to select, modify, or test specific bits.

### Creating Masks

```python
# Single bit at position k
mask = 1 << k

# k=0:  0000 0001
# k=3:  0000 1000
# k=7:  1000 0000

# All 1s for n bits (the "n-bit mask")
mask = (1 << n) - 1

# n=4:  (1 << 4) - 1 = 0001 0000 - 1 = 0000 1111  = 0xF
# n=8:  (1 << 8) - 1 = 1 0000 0000 - 1 = 1111 1111 = 0xFF
# n=16: 0xFFFF
# n=32: 0xFFFFFFFF

# Range of bits from start to start+length-1
mask = ((1 << length) - 1) << start
# length=4, start=4 → 0000 1111 << 4 = 1111 0000

# All 1s except position k
mask = ~(1 << k)

# Alternating bits
0xAA = 1010 1010
0x55 = 0101 0101
```

### Common Named Masks

```
0xFF         = 1111 1111              lower byte / 8-bit mask
0xFF00       = 1111 1111 0000 0000    second byte
0x0F         = 0000 1111              lower nibble
0xF0         = 1111 0000              upper nibble
0x7FFFFFFF   = 0111...1111            all bits except sign (32-bit)
0x80000000   = 1000...0000            sign bit only (32-bit)
0x55555555   = 0101...0101            every other bit (even positions)
0xAAAAAAAA   = 1010...1010            every other bit (odd positions)
0x33333333   = 0011...0011            pairs: 0011 repeating
0xCCCCCCCC   = 1100...1100            pairs: 1100 repeating
0x0F0F0F0F   = nibbles: 0000 1111 repeating
0x00FF00FF   = bytes: 0000 0000 1111 1111 repeating
```

### Applying Masks

```python
# EXTRACT lower byte of a 32-bit value
lower_byte = value & 0xFF

# EXTRACT upper byte of a 16-bit value
upper_byte = (value >> 8) & 0xFF

# CLEAR upper 16 bits, keep lower 16
value & 0x0000FFFF

# TEST if any of a set of flags is active
flags = 0b10110
mask  = 0b00110   # testing flags 1 and 2
if flags & mask:  # True — at least one of the masked bits is set
    ...

# TEST if ALL of a set of flags are active
if (flags & mask) == mask:  # True only if every bit in mask is also in flags
    ...

# SET a group of bits (e.g., set bits 4-7 to all 1s)
value | 0xF0

# CLEAR a group of bits (e.g., clear bits 4-7)
value & ~0xF0   # ~0xF0 = 0x0F (for 8-bit)
```

---

## 7. Arithmetic with Bits

### Multiply and Divide by Powers of 2

```python
x * 2   ←→   x << 1
x * 4   ←→   x << 2
x * 8   ←→   x << 3
x * 2^k ←→   x << k

x / 2   ←→   x >> 1   (for non-negative integers)
x / 4   ←→   x >> 2
x / 8   ←→   x >> 3
x / 2^k ←→   x >> k

# Example: compute x * 10 without multiplication
x * 10 = x * (8 + 2) = (x << 3) + (x << 1)

# Example: compute x * 7
x * 7 = x * (8 - 1) = (x << 3) - x
```

### Modulo by Powers of 2

```python
x % 2   ←→   x & 1
x % 4   ←→   x & 3
x % 8   ←→   x & 7
x % 16  ←→   x & 15
x % 2^k ←→   x & (2^k - 1)   =   x & ((1 << k) - 1)

# This works because 2^k - 1 is all 1s in the lower k bits
# AND with it = keep only the lower k bits = remainder after dividing by 2^k
```

### Integer Absolute Value (Branchless)

```python
def abs_branchless(n):
    # Works for 32-bit signed integers
    mask = n >> 31       # arithmetic shift: -1 if negative, 0 if positive
    return (n + mask) ^ mask
    # If n >= 0: mask=0,  (n + 0) ^ 0 = n
    # If n <  0: mask=-1, (n + (-1)) ^ (-1) = ~(n-1) = -n  ✅

# Alternatively:
def abs_branchless_v2(n):
    mask = n >> 31
    return (n ^ mask) - mask
```

### Integer Min and Max (Branchless)

```python
def min_branchless(a, b):
    # No branch — avoids branch misprediction penalty
    return b ^ ((a ^ b) & -(a < b))
    # -(a < b) is either 0 (all zeros) or -1 (all ones)
    # If a < b: -(True) = -1 = all 1s, so: b ^ (a^b) = a
    # If a >= b: -(False) = 0 = all zeros, so: b ^ 0 = b

def max_branchless(a, b):
    return a ^ ((a ^ b) & -(a < b))
```

### Average Without Overflow

```python
# ❌ Can overflow if a+b > INT_MAX:
mid = (a + b) // 2

# ✅ Safe:
mid = a + (b - a) // 2

# ✅ Also safe using bits (works for unsigned):
mid = (a & b) + ((a ^ b) >> 1)
# a & b = bits both have in common
# (a ^ b) >> 1 = average of the differing bits
```

### Multiply Two Numbers Using Bitwise Only (Russian Peasant)

```python
def multiply(a, b):
    result = 0
    while b > 0:
        if b & 1:            # if b is odd, add a to result
            result += a
        a <<= 1              # double a
        b >>= 1              # halve b
    return result

# This is essentially binary long multiplication
# multiply(5, 6) = multiply(5, 0b110)
# b=110: bit0=0, skip; a=10,  b=011
# b=011: bit0=1, result+=10;  a=100, b=001
# b=001: bit0=1, result+=100; a=1000, b=000
# result = 10 + 100 = 110 = 30 ✅  (5×6=30)
```

---

## 8. Bit Tricks on Integers

### Check If Number is a Power of 2

```python
def is_power_of_two(n):
    return n > 0 and (n & (n - 1)) == 0

# Why? Powers of 2 have exactly ONE bit set:
# 8  = 1000
# 7  = 0111
# 8 & 7 = 0000  → IS power of 2  ✅

# 6  = 0110
# 5  = 0101
# 6 & 5 = 0100 ≠ 0 → NOT power of 2  ✅

# Handles n=0: 0 & (-1) = 0, but 0 is NOT a power of 2, hence n > 0 check
```

### Clear the Lowest Set Bit

```python
def clear_lowest_set_bit(n):
    return n & (n - 1)

# n     = 1011 0100
# n-1   = 1011 0011   (borrows from the lowest set bit, flipping it and all below)
# n&n-1 = 1011 0000   (lowest set bit and everything below it cleared)  ✅

# Application: Kernighan's bit count
def popcount(n):
    count = 0
    while n:
        n &= n - 1   # remove one set bit per iteration
        count += 1
    return count     # O(k) where k = number of set bits
```

### Isolate the Lowest Set Bit

```python
def lowest_set_bit(n):
    return n & (-n)

# n    = 0110 1100
# -n   = 1001 0100   (two's complement of n)
# n&-n = 0000 0100   → only the lowest set bit remains  ✅

# Why -n works: -n = ~n + 1
# ~n flips all bits. Adding 1 causes a carry that propagates up through
# all the 1s (which were 0s in ~n) until it hits the first 0 (the lowest set bit).
# That position has 1 in both n and ~n+1, so AND isolates it.
```

### Turn Off the Rightmost Consecutive 1s

```python
def clear_trailing_ones(n):
    return n & (n + 1)

# n     = 1011 0111
# n+1   = 1011 1000  (carry propagates through trailing 1s, turning them to 0)
# n&n+1 = 1011 0000  ✅
```

### Turn On the Rightmost Consecutive 0s

```python
def set_trailing_zeros(n):
    return n | (n - 1)

# n     = 1011 1000
# n-1   = 1011 0111  (borrow propagates, setting all trailing zeros to 1)
# n|n-1 = 1011 1111  ✅
```

### Swap Two Integers Without a Temp Variable

```python
# XOR swap — works for integers
a ^= b
b ^= a   # b = b ^ (a ^ b) = a
a ^= b   # a = (a ^ b) ^ a = b

# ⚠️ Warning: fails if a and b are the same variable/location (a ^= a → 0)
# Safe swap with aliasing check:
if a != b:
    a ^= b; b ^= a; a ^= b
```

### Find Missing Number in 0..n

```python
def find_missing(nums, n):
    # XOR all numbers 0..n, then XOR with all elements in array
    # Every number that appears cancels; missing number remains
    xor = 0
    for i in range(n + 1):
        xor ^= i
    for x in nums:
        xor ^= x
    return xor

# Example: nums = [0, 1, 3, 4], n = 4
# XOR of 0^1^2^3^4 = 4 (coincidence), then XOR with 0^1^3^4 = 7
# 4 ^ 7 = 3 → missing is 3 ✅
```

### Find the Two Non-Duplicate Numbers

```python
def find_two_unique(nums):
    # All elements appear twice except two: a and b
    # Step 1: XOR all → gets a ^ b (a and b are different, so a^b ≠ 0)
    xor = 0
    for x in nums: xor ^= x         # xor = a ^ b

    # Step 2: Find any set bit in a^b (use lowest set bit)
    diff_bit = xor & (-xor)

    # Step 3: Split nums into two groups by that bit
    # a will be in one group, b in the other
    a = b = 0
    for x in nums:
        if x & diff_bit:
            a ^= x
        else:
            b ^= x
    return a, b
```

### Check If Two Integers Have Opposite Signs

```python
def opposite_signs(a, b):
    return (a ^ b) < 0
    # Sign bits are different → XOR sign bit = 1 → result is negative
```

### Conditionally Negate Without Branch

```python
def conditional_negate(n, negate: bool):
    mask = -int(negate)   # 0 if False, -1 (all 1s) if True
    return (n ^ mask) - mask
    # If negate=False: (n ^ 0) - 0 = n
    # If negate=True:  (n ^ -1) - (-1) = ~n + 1 = -n  ✅
```

---

## 9. Counting & Finding Bits

### Count Set Bits (Popcount / Hamming Weight)

```python
# Method 1: Naive — O(log n)
def popcount_naive(n):
    count = 0
    while n:
        count += n & 1
        n >>= 1
    return count

# Method 2: Kernighan's — O(k) where k = popcount
def popcount_kernighan(n):
    count = 0
    while n:
        n &= n - 1   # clear lowest set bit
        count += 1
    return count

# Method 3: Lookup table — O(1) after O(1) setup
lookup = [bin(i).count('1') for i in range(256)]  # precompute for all bytes
def popcount_lookup(n):
    return (lookup[n & 0xFF] +
            lookup[(n >> 8)  & 0xFF] +
            lookup[(n >> 16) & 0xFF] +
            lookup[(n >> 24) & 0xFF])

# Method 4: Parallel bit counting (SWAR — SIMD Within A Register)
# Works in O(1) using bit parallelism on a 32-bit integer
def popcount_parallel(n):
    n = n - ((n >> 1) & 0x55555555)            # count bits in pairs
    n = (n & 0x33333333) + ((n >> 2) & 0x33333333)  # count bits in groups of 4
    n = (n + (n >> 4)) & 0x0F0F0F0F            # count bits in groups of 8
    return (n * 0x01010101) >> 24              # sum all bytes

# Method 5: Built-in (always prefer this in production)
bin(n).count('1')            # Python
import ctypes; ctypes.c_uint32(n).value.bit_count()  # Python 3.10+
```

### Find Position of Lowest Set Bit (Trailing Zeros)

```python
def count_trailing_zeros(n):
    # Number of trailing zeros = position of lowest set bit
    if n == 0: return 32   # or 64
    count = 0
    while not (n & 1):
        n >>= 1
        count += 1
    return count

# Faster: use lowest set bit isolation + popcount
def count_trailing_zeros_fast(n):
    return popcount((n & -n) - 1)
    # n & -n isolates lowest set bit, e.g., 0001 0000
    # subtract 1 → 0000 1111 (all 1s below that bit)
    # popcount(0b1111) = 4 = position of the bit  ✅

# Built-in:
(n & -n).bit_length() - 1   # Python

# C/C++: __builtin_ctz(n)     (count trailing zeros)
# Java:  Integer.numberOfTrailingZeros(n)
```

### Find Position of Highest Set Bit (bit_length)

```python
def highest_set_bit_pos(n):
    return n.bit_length() - 1  # Python

# Equivalent to floor(log₂(n)) for n > 0
# 8 = 1000 → bit_length = 4 → position 3  ✅

# C/C++: 31 - __builtin_clz(n)  (count leading zeros)
# Java:  31 - Integer.numberOfLeadingZeros(n)
```

### Count Leading Zeros

```python
def count_leading_zeros_32(n):
    if n == 0: return 32
    count = 0
    while not (n & (1 << 31)):
        n <<= 1
        count += 1
    return count

# Built-in:
# C/C++: __builtin_clz(n)
# Java:  Integer.numberOfLeadingZeros(n)
```

### Parity (Is Popcount Even or Odd?)

```python
def parity(n):
    # Returns 1 if odd number of set bits, 0 if even
    # XOR all bits together: pairs cancel, odd one out remains
    n ^= n >> 16
    n ^= n >> 8
    n ^= n >> 4
    n ^= n >> 2
    n ^= n >> 1
    return n & 1

# Each step folds the upper half onto the lower half via XOR
# After all folds, bit 0 = XOR of all original bits = parity
```

### Hamming Distance

```python
def hamming_distance(a, b):
    # Number of bit positions where a and b differ
    return bin(a ^ b).count('1')
    # XOR gives 1s where bits differ; count those 1s

# hamming_distance(0b1001, 0b0101) = popcount(0b1100) = 2  ✅
```

---

## 10. Bit Manipulation on Arrays & Sets

### Represent a Set as a Bitmask

When you have a universe of `n` elements (n ≤ 64), represent a subset as an integer where bit `i` = 1 means element `i` is in the set.

```python
# Universe: elements {0, 1, 2, 3, 4, 5, 6, 7}
# Subset {1, 3, 5}: bit 1, bit 3, bit 5 set
subset = (1 << 1) | (1 << 3) | (1 << 5)
subset = 0b00101010 = 42

# Set operations using bitwise operators:
A | B    →   Union          (elements in A OR B)
A & B    →   Intersection   (elements in BOTH A and B)
A ^ B    →   Symmetric diff (elements in A or B, not both)
A & ~B   →   Difference     (elements in A but NOT in B)
~A       →   Complement     (flip all bits — then mask to universe size)
A == B   →   Equality
(A & B) == B   →   B is a subset of A
A == 0   →   A is the empty set
popcount(A)    →   Size of set A
```

### Enumerate All Subsets of a Set

```python
def all_subsets(mask):
    """Enumerate all subsets of 'mask' (including empty set)."""
    subset = mask
    while subset > 0:
        yield subset
        subset = (subset - 1) & mask   # ← the key trick
    yield 0   # empty set

# The trick: (subset - 1) & mask
# Subtracting 1 turns off the lowest set bit and sets all bits below it.
# ANDing with mask ensures we stay within the original set.
# This iterates through all 2^k subsets of a k-bit mask in O(2^k) time.

# Example: mask = 0b1011 (elements {0,1,3})
# Subsets: 1011, 1010, 1001, 1000, 0011, 0010, 0001, 0000
#          {0,1,3} {1,3} {0,3} {3} {0,1} {1} {0} {}
```

### Enumerate All Subsets of Size K

```python
def subsets_of_size_k(n, k):
    """Generate all n-bit masks with exactly k bits set (Gosper's hack)."""
    if k == 0:
        yield 0
        return
    mask = (1 << k) - 1   # smallest k-bit number: 000...0111...1
    while mask < (1 << n):
        yield mask
        # Gosper's Hack: compute next permutation of bits
        c = mask & (-mask)            # lowest set bit
        r = mask + c                  # clear trailing 1s, set next 0
        mask = (((r ^ mask) >> 2) // c) | r
        # This generates all masks with exactly k bits in ascending order

# Example n=4, k=2: 0011, 0101, 0110, 1001, 1010, 1100
```

### Check if Subset

```python
def is_subset(sub, full):
    """Returns True if 'sub' is a subset of 'full'."""
    return (sub & full) == sub
    # Every bit in sub must also be set in full

is_subset(0b0101, 0b1111)   # True  — {0,2} ⊆ {0,1,2,3}
is_subset(0b1001, 0b0111)   # False — bit 3 in sub but not in full
```

### Bit Array / Bitset Operations

```python
# Store n booleans in a list of integers (each int holds 64 bits)
class Bitset:
    def __init__(self, n):
        self.n = n
        self.data = [0] * ((n + 63) // 64)  # ceil(n/64) words

    def set(self, i):
        self.data[i // 64] |= (1 << (i % 64))

    def clear(self, i):
        self.data[i // 64] &= ~(1 << (i % 64))

    def get(self, i):
        return (self.data[i // 64] >> (i % 64)) & 1

    def count(self):
        return sum(bin(w).count('1') for w in self.data)

    def union(self, other):
        result = Bitset(self.n)
        result.data = [a | b for a, b in zip(self.data, other.data)]
        return result

    def intersection(self, other):
        result = Bitset(self.n)
        result.data = [a & b for a, b in zip(self.data, other.data)]
        return result
```

---

## 11. Bitmask DP (Dynamic Programming)

Bitmask DP is one of the most powerful applications of bit manipulation. It solves problems over **subsets** of elements efficiently.

### The Pattern

```
State: dp[mask] = best answer for the subset of elements represented by 'mask'
Transition: try adding element i to state dp[mask]
             → dp[mask | (1 << i)] = combine(dp[mask], cost(i, mask))

Time complexity:  O(2^n × n)   usually
Space complexity: O(2^n)
Feasible for:     n ≤ 20 (common), n ≤ 25 (tight), n ≤ 30 (very tight)
```

### Classic: Travelling Salesman Problem (TSP)

Find the shortest route visiting all `n` cities exactly once.

```python
def tsp_bitmask(dist, n):
    INF = float('inf')
    # dp[mask][i] = min cost to visit all cities in 'mask', ending at city i
    dp = [[INF] * n for _ in range(1 << n)]

    # Start at city 0 (bit 0 set)
    dp[1][0] = 0

    for mask in range(1 << n):
        for u in range(n):
            if dp[mask][u] == INF:
                continue
            if not (mask >> u & 1):    # u must be in mask
                continue
            # Try extending to city v
            for v in range(n):
                if mask >> v & 1:      # v already visited
                    continue
                new_mask = mask | (1 << v)
                dp[new_mask][v] = min(dp[new_mask][v], dp[mask][u] + dist[u][v])

    # Return to city 0 after visiting all cities
    full_mask = (1 << n) - 1
    return min(dp[full_mask][i] + dist[i][0] for i in range(1, n))

# n=4 cities: 2^4 = 16 states × 4 = 64 transitions
# vs brute force: 4! = 24 routes (small n), but scales to n=20 easily
```

### Classic: Assignment Problem / Minimum Cost Matching

Assign `n` tasks to `n` workers, each with different costs. Minimize total cost.

```python
def min_cost_assignment(cost, n):
    INF = float('inf')
    # dp[mask] = min cost to assign tasks in 'mask' to first popcount(mask) workers
    dp = [INF] * (1 << n)
    dp[0] = 0

    for mask in range(1 << n):
        if dp[mask] == INF:
            continue
        worker = bin(mask).count('1')   # next worker to assign
        if worker == n:
            continue
        for task in range(n):
            if mask >> task & 1:         # task already assigned
                continue
            dp[mask | (1 << task)] = min(
                dp[mask | (1 << task)],
                dp[mask] + cost[worker][task]
            )

    return dp[(1 << n) - 1]
```

### Subset Sum DP

```python
def subset_sum_exists(nums, target):
    # dp is a bitmask: bit k = 1 if sum k is achievable
    dp = 1   # bit 0 set = sum 0 is achievable
    for x in nums:
        dp |= dp << x   # "for each achievable sum s, s+x is also achievable"
    return bool(dp >> target & 1)

# This is O(n × target / word_size) using bitmask as a bitset
# Much faster than O(n × target) with a boolean array for large inputs
```

### Broken Profile DP / State Compression

Used for tiling problems (e.g., count ways to tile a grid with dominoes).

```python
def count_tilings(rows, cols):
    # dp[mask] = number of ways to fill current column if 'mask'
    # represents which cells in the current column are already filled
    # (from the previous column's horizontal dominoes sticking out)
    from collections import defaultdict
    dp = defaultdict(int)
    dp[0] = 1

    def fill(col_dp, cur_mask, next_mask, row, rows):
        if row == rows:
            col_dp[next_mask] += col_dp.get(cur_mask, 0)
            return
        # Try placing vertical domino at this row (occupies current column only)
        if not (cur_mask >> row & 1):   # current cell is free
            fill(col_dp, cur_mask | (1 << row), next_mask, row + 1, rows)
        # Try placing horizontal domino (occupies current and next column)
        if not (cur_mask >> row & 1):
            fill(col_dp, cur_mask | (1 << row), next_mask | (1 << row), row + 1, rows)
        # Current cell already filled from previous column
        if cur_mask >> row & 1:
            fill(col_dp, cur_mask, next_mask, row + 1, rows)

    for _ in range(cols):
        new_dp = defaultdict(int)
        for mask, ways in dp.items():
            fill({mask: ways}, mask, 0, 0, rows)
        dp = new_dp
    return dp[0]
```

---

## 12. Two's Complement Deep Dive

### How Two's Complement Works

```
For an n-bit system:
  Non-negative numbers (0 to 2^(n-1) - 1):  MSB = 0
  Negative numbers (-2^(n-1) to -1):         MSB = 1

To convert positive x to -x:
  Method 1: Flip all bits, then add 1.
  Method 2: Starting from LSB, copy bits up to and including first 1, then flip the rest.

8-bit examples:
  +5:  0000 0101
  -5:  1111 1011  (flip: 1111 1010, add 1: 1111 1011)

  +1:  0000 0001
  -1:  1111 1111

  +128: IMPOSSIBLE in 8-bit signed (max is +127 = 0111 1111)
  -128: 1000 0000  (this is the only number that has no positive counterpart!)
```

### The Magic of Two's Complement

```
Why is it elegant?

  1. Only ONE representation of zero: 0000 0000
  2. Addition works identically for positive and negative numbers:
       0000 0101  (+5)
     + 1111 1011  (-5)
     ───────────
     1 0000 0000  → drop overflow bit → 0000 0000 = 0  ✅

  3. Subtraction is just addition of the negation:
     a - b  =  a + (-b)  =  a + (~b + 1)

  4. Bit pattern of -1 is all 1s: 1111 1111
     This is why ~0 = -1 (NOT zero = all ones = negative one)
```

### Integer Overflow Behavior

```
In C/C++ (undefined for signed, defined for unsigned):
  INT_MAX + 1  → undefined (UB for signed in C!)
  UINT_MAX + 1 → wraps to 0

In Java, Python:
  Java: wraps (int is strictly 32-bit)
  Python: arbitrary precision (no overflow!)

8-bit signed overflow:
  0111 1111  (+127)
+ 0000 0001  (+1)
───────────
  1000 0000  → interpreted as -128!  ← OVERFLOW

Detecting overflow:
  For a + b: overflow iff (a > 0 && b > 0 && result < 0)
                        or (a < 0 && b < 0 && result >= 0)
  Via bits:   overflow iff carry into MSB ≠ carry out of MSB
```

### Signed vs Unsigned Comparison Trap

```python
# In C (32-bit):
int a = -1;
unsigned int b = 1;
if (a < b)   →  FALSE!  because -1 converts to 4294967295 (unsigned)

# Safe comparison:
if ((unsigned int)a < b)   →  TRUE  (both unsigned: 4294967295 < ... wait no)
# Actually comparing -1 to 1:
# As signed: -1 < 1  ✅
# As unsigned: 4294967295 > 1  ❌ wrong comparison

# Rule: never mix signed and unsigned in comparisons without explicit casts
```

---

## 13. Shifts — Every Detail

### Shift Behavior by Language

```
LEFT SHIFT (<<): Always fills with 0s on the right.
  Behavior is the same for signed and unsigned.
  ⚠️ In C, left-shifting a 1 into the sign bit is UNDEFINED BEHAVIOR for signed ints.
  ✅ In Java/Python/JS, well-defined.

RIGHT SHIFT (>>):
  C/C++:  >> is IMPLEMENTATION-DEFINED for negative signed integers
          (usually arithmetic, but not guaranteed by standard!)
  Java:   >> is ARITHMETIC (sign-extending), >>> is LOGICAL (zero-filling)
  Python: >> is ARITHMETIC (sign-extending, always)
  JavaScript: >> is ARITHMETIC, >>> is LOGICAL (32-bit)

LOGICAL right shift in Python (simulate >>>):
  (n & 0xFFFFFFFF) >> k    # mask to 32 bits first, then shift
```

### Shift Edge Cases

```python
# Shifting by 0: no-op
n << 0  ==  n
n >> 0  ==  n

# Shifting by word size: UNDEFINED in C!
n << 32  # UB in C for 32-bit int!  Use n << 31 >> 31 trick instead
n >> 32  # UB in C for 32-bit int!

# Shifting by negative amount: UNDEFINED in C
n << -1  # UB!

# In Python, arbitrary precision, so no overflow or UB
1 << 1000  # works fine in Python

# Shifting past the bit width in Java:
# Java automatically mods the shift amount by the type width
# n << 33 is the same as n << 1  (for int)
# n << 65 is the same as n << 1  (for long)
```

### Signed Right Shift for Sign Propagation

```python
# Arithmetic right shift by (word_size - 1) gives the sign mask
# For 32-bit:
mask = n >> 31    # 0x00000000 if n >= 0, 0xFFFFFFFF if n < 0

# This is incredibly useful for branchless code:
abs_val = (n ^ mask) - mask     # branchless abs
sign    = (n >> 31) | 1         # +1 or -1 (never 0)
```

### Cyclic / Circular Shifts (Rotations)

A rotation is like a shift but bits that fall off one end reappear at the other.

```python
def rotate_left_32(n, k):
    k %= 32
    return ((n << k) | (n >> (32 - k))) & 0xFFFFFFFF

def rotate_right_32(n, k):
    k %= 32
    return ((n >> k) | (n << (32 - k))) & 0xFFFFFFFF

# Example: rotate_left_32(0b10110001, 3)
#   Original:  1011 0001
#   << 3:   (1)011 0001 000   (top 3 bits overflow)
#   >> 5:      0000 0101      (those bits go to the right)
#   OR:        0101 1000 101... wait:
#   rotate_left_32(0b10110001, 3) = 0b10001101  ✅ (181 → 141)

# Note: CPU instructions like ROL/ROR do this in one clock cycle
# Used heavily in: SHA-1, SHA-256, AES, MD5 hash functions
```

---

## 14. Language-Specific Reference

### Python

```python
# Literals
0b1010    # binary = 10
0o17      # octal = 15
0xFF      # hex = 255

# Arbitrary precision (no overflow!)
1 << 100  # works perfectly

# Negative numbers behave like infinite-width two's complement
bin(-1)   # '-0b1'  (Python uses sign + magnitude for display)
-1 & 0xFF # = 255   (& truncates to width)

# Useful built-ins
bin(n)           # '0b1010' — binary string
hex(n)           # '0xa'    — hex string
int('1010', 2)   # = 10     — parse binary string
int('FF', 16)    # = 255    — parse hex string
n.bit_length()   # bits needed to represent n (= ⌊log₂n⌋ + 1)
n.bit_count()    # popcount (Python 3.10+)

# Simulate 32-bit unsigned right shift (>>>) from Java:
(n & 0xFFFFFFFF) >> k

# Simulate 32-bit overflow (wrap to int32 range):
def to_int32(n):
    n &= 0xFFFFFFFF          # keep 32 bits
    if n >= 0x80000000:
        n -= 0x100000000     # interpret as signed
    return n
```

### C / C++

```c
// Integer types with known widths (use these!)
#include <stdint.h>
int8_t, int16_t, int32_t, int64_t        // signed
uint8_t, uint16_t, uint32_t, uint64_t   // unsigned

// GCC built-ins (very fast — compile to single CPU instruction)
__builtin_popcount(n)     // popcount for unsigned int
__builtin_popcountll(n)   // popcount for unsigned long long
__builtin_clz(n)          // count leading zeros (UB if n == 0!)
__builtin_ctz(n)          // count trailing zeros (UB if n == 0!)
__builtin_parity(n)       // parity (XOR of all bits)

// Safe versions
n == 0 ? 32 : __builtin_clz(n)

// Literal suffixes
0b1010U    // unsigned binary (C++14)
0xFFFFFFFFULL   // unsigned long long hex

// Right shift behavior
int a = -8;
a >> 2;    // implementation-defined (usually -2, arithmetic)
(unsigned)a >> 2;  // always logical: large positive number

// Avoid UB with shifts:
(uint32_t)1 << 31   // ✅ fine (unsigned)
1 << 31             // ⚠️ UB in C if int is 32-bit!
```

### Java

```java
// Arithmetic vs logical right shift
n >> k    // arithmetic (sign-extending)
n >>> k   // logical (zero-filling)  ← Java-unique operator

// Built-in methods
Integer.bitCount(n)              // popcount
Integer.highestOneBit(n)         // isolate highest set bit
Integer.lowestOneBit(n)          // isolate lowest set bit  (= n & -n)
Integer.numberOfLeadingZeros(n)  // CLZ
Integer.numberOfTrailingZeros(n) // CTZ
Integer.reverse(n)               // reverse all bits
Integer.reverseBytes(n)          // reverse bytes (endian swap)
Integer.toBinaryString(n)        // binary string (no '0b' prefix)
Integer.toHexString(n)           // hex string

// Long versions:
Long.bitCount(n), Long.numberOfTrailingZeros(n), etc.

// Overflow wraps in Java (no UB like C)
Integer.MAX_VALUE + 1 == Integer.MIN_VALUE  // true
```

### JavaScript

```javascript
// JS bitwise: operands converted to signed 32-bit before operation
// Result is always signed 32-bit

-1 >>> 0          // 4294967295 — convert to unsigned 32-bit
n >>> 0           // truncate to 32-bit unsigned
n | 0             // truncate to 32-bit signed (fast int conversion!)
Math.floor(x) === x | 0   // true for safe integers

// Arithmetic vs logical right shift
n >> k    // arithmetic
n >>> k   // logical (unsigned right shift)

// No built-in popcount, but:
n.toString(2).split('0').join('').length  // popcount (string trick)

// BigInt for 64-bit operations (ES2020):
1n << 32n    // BigInt shift
0xFFFFFFFFn  // BigInt hex literal

// Signed/unsigned weirdness:
0xFFFFFFFF | 0   // = -1  (treated as signed 32-bit)
0xFFFFFFFF >>> 0 // = 4294967295  (treated as unsigned via >>>)
```

---

## 15. Classic Interview Problems

### Single Number (LeetCode 136)
Every element appears twice except one. Find it.

```python
def single_number(nums):
    return reduce(lambda a, b: a ^ b, nums)
    # XOR is commutative and associative: pairs cancel, unique remains
    # Time: O(n), Space: O(1)
```

### Number of 1 Bits (LeetCode 191)

```python
def hammingWeight(n):
    count = 0
    while n:
        n &= n - 1   # Kernighan's trick
        count += 1
    return count
```

### Reverse Bits (LeetCode 190)

```python
def reverseBits(n):
    result = 0
    for _ in range(32):
        result = (result << 1) | (n & 1)
        n >>= 1
    return result

# Divide and conquer approach — O(1) no loop:
def reverseBits_fast(n):
    n = ((n & 0xFFFF0000) >> 16) | ((n & 0x0000FFFF) << 16)  # swap 16-bit halves
    n = ((n & 0xFF00FF00) >>  8) | ((n & 0x00FF00FF) <<  8)  # swap bytes
    n = ((n & 0xF0F0F0F0) >>  4) | ((n & 0x0F0F0F0F) <<  4)  # swap nibbles
    n = ((n & 0xCCCCCCCC) >>  2) | ((n & 0x33333333) <<  2)  # swap 2-bit groups
    n = ((n & 0xAAAAAAAA) >>  1) | ((n & 0x55555555) <<  1)  # swap adjacent bits
    return n
```

### Power of Two (LeetCode 231)

```python
def isPowerOfTwo(n):
    return n > 0 and (n & (n - 1)) == 0
```

### Sum of Two Integers Without + or - (LeetCode 371)

```python
def getSum(a, b):
    # Simulate binary addition using XOR (sum without carry) and AND (carry)
    mask = 0xFFFFFFFF   # 32-bit mask
    while b & mask:
        carry = (a & b) << 1   # carry bits
        a = a ^ b              # sum without carry
        b = carry
    # Handle negative result (Python has arbitrary precision, need to sign-extend)
    return a if (a >> 31 & 1) == 0 else ~(a ^ mask)
```

### Missing Number (LeetCode 268)

```python
def missingNumber(nums):
    # XOR 0..n with all elements; missing number has no pair
    xor = len(nums)
    for i, v in enumerate(nums):
        xor ^= i ^ v
    return xor
```

### Counting Bits (LeetCode 338)

```python
def countBits(n):
    # dp[i] = popcount(i)
    # Key insight: dp[i] = dp[i >> 1] + (i & 1)
    # (remove LSB, look up its count, add 1 if LSB was 1)
    dp = [0] * (n + 1)
    for i in range(1, n + 1):
        dp[i] = dp[i >> 1] + (i & 1)
    return dp
    # Time: O(n), Space: O(n), no math functions needed
```

### Bitwise AND of Numbers Range (LeetCode 201)

```python
def rangeBitwiseAnd(left, right):
    # Find common prefix of left and right in binary
    # Any bit that flips between left and right will be 0 in the AND result
    shift = 0
    while left != right:
        left >>= 1
        right >>= 1
        shift += 1
    return left << shift
```

### Maximum XOR of Two Numbers (LeetCode 421)

```python
def findMaximumXOR(nums):
    max_xor = 0
    mask = 0
    for i in range(31, -1, -1):   # try to set each bit from high to low
        mask |= (1 << i)
        prefixes = {n & mask for n in nums}   # all prefixes of length (32-i)
        # Can we achieve max_xor | (1 << i)?
        candidate = max_xor | (1 << i)
        # Check if any two prefixes XOR to the candidate
        if any(candidate ^ p in prefixes for p in prefixes):
            max_xor = candidate
    return max_xor
```

### Total Hamming Distance (LeetCode 477)

```python
def totalHammingDistance(nums):
    # For each bit position, count pairs that differ
    # = (count of 1s) × (count of 0s)
    total = 0
    for bit in range(32):
        ones = sum((n >> bit) & 1 for n in nums)
        zeros = len(nums) - ones
        total += ones * zeros
    return total
    # O(32n) instead of O(n²) brute force
```

### Decode XORed Array (LeetCode 1720)

```python
def decode(encoded, first):
    result = [first]
    for e in encoded:
        result.append(result[-1] ^ e)
        # a ^ b = encoded[i], a = result[-1], so b = a ^ encoded[i]
    return result
```

---

## 16. Expert Patterns & Advanced Tricks

### The n & (n-1) Family

```python
# These all stem from: n-1 flips the lowest set bit and everything below it

n & (n-1)    # clear lowest set bit
n | (n+1)    # set lowest clear bit
n ^ (n-1)    # create mask of lowest set bit + everything below
             # e.g., 1011 0100 → 0000 0111

n & (n+1)    # clear trailing 1s   e.g., 1011 0111 → 1011 0000
n | (n-1)    # set trailing 0s     e.g., 1011 0100 → 1011 0111

~n & (n+1)   # isolate lowest clear bit  e.g., 1011 0111 → 0000 1000
~n | (n-1)   # complement with trailing  
```

### SWAR (SIMD Within A Register) Techniques

Apply an operation to multiple packed values in a single integer.

```python
# Add 1 to each nibble (4-bit field) in a 32-bit integer simultaneously
# Saturating, no overflow between nibbles
def add_nibbles(packed, val):
    # Each nibble is 4 bits wide; we add val to each independently
    # Mask alternating nibbles to prevent carry bleed
    lo_mask = 0x0F0F0F0F
    hi_mask = 0xF0F0F0F0
    lo = (packed & lo_mask) + (val & lo_mask)
    hi = (packed & hi_mask) + (val << 4 & hi_mask)
    return (lo & lo_mask) | (hi & hi_mask)

# Count bytes equal to zero in a 32-bit word (useful in strlen optimization)
def has_zero_byte(v):
    # Returns non-zero if any byte of v is zero
    return (v - 0x01010101) & ~v & 0x80808080
```

### Parallel Prefix (Prefix XOR in O(log n) steps)

```python
# Compute XOR prefix sum array in O(log n) parallel steps
def parallel_prefix_xor(arr_as_int, n):
    # Treats each element as a bit in the integer
    for i in range(int.bit_length(n)):
        arr_as_int ^= arr_as_int >> (1 << i)
    return arr_as_int
```

### Gray Code Encoding/Decoding

```python
def binary_to_gray(n):
    return n ^ (n >> 1)

def gray_to_binary(gray):
    mask = gray >> 1
    while mask:
        gray ^= mask
        mask >>= 1
    return gray

# Gray code: adjacent values differ by exactly 1 bit
# 0=000, 1=001, 2=011, 3=010, 4=110, 5=111, 6=101, 7=100
```

### Bit Interleaving (Morton Code / Z-order Curve)

Pack two n-bit integers into a 2n-bit integer by interleaving bits.

```python
def interleave_bits(x, y):
    """Interleave bits of 16-bit x and y into 32-bit Morton code."""
    def spread(n):
        n &= 0x0000FFFF
        n = (n | (n << 8))  & 0x00FF00FF
        n = (n | (n << 4))  & 0x0F0F0F0F
        n = (n | (n << 2))  & 0x33333333
        n = (n | (n << 1))  & 0x55555555
        return n
    return spread(x) | (spread(y) << 1)

# Usage: 2D points → 1D space-filling curve index
# morton = interleave_bits(x_coord, y_coord)
# Nearby points in 2D → nearby Morton codes → cache-friendly access
```

### Lookup Table Generation

```python
# Precompute popcount for all 8-bit values at import time
POPCOUNT_8 = [bin(i).count('1') for i in range(256)]

# Precompute reverse bits for all 8-bit values
REVERSE_8 = [int(f'{i:08b}'[::-1], 2) for i in range(256)]

# Precompute parity for all 8-bit values
PARITY_8 = [bin(i).count('1') % 2 for i in range(256)]

# Usage: O(1) lookup vs O(8) computation
def popcount32_table(n):
    return (POPCOUNT_8[n & 0xFF] +
            POPCOUNT_8[(n >> 8)  & 0xFF] +
            POPCOUNT_8[(n >> 16) & 0xFF] +
            POPCOUNT_8[(n >> 24) & 0xFF])
```

### Endianness Swap

```python
def swap_bytes_32(n):
    """Convert between big-endian and little-endian for 32-bit integer."""
    return (((n & 0xFF000000) >> 24) |
            ((n & 0x00FF0000) >>  8) |
            ((n & 0x0000FF00) <<  8) |
            ((n & 0x000000FF) << 24))

def swap_bytes_16(n):
    return ((n & 0xFF00) >> 8) | ((n & 0x00FF) << 8)

# In C: __builtin_bswap32(n), __builtin_bswap64(n)
# In Python: int.from_bytes(n.to_bytes(4, 'little'), 'big')
```

### Bit-Parallel String Matching (Shift-Or / Bitap Algorithm)

```python
def bitap_search(text, pattern):
    """Find pattern in text using bitwise parallelism. O(|text| × |pattern|/w)"""
    m = len(pattern)
    if m > 64: raise ValueError("Pattern too long for 64-bit machine word")

    # Precompute: for each character c, which positions in pattern match c?
    char_masks = {}
    for i, c in enumerate(pattern):
        char_masks[c] = char_masks.get(c, 0) | (1 << i)

    # Search: state tracks which positions in pattern could end at current text pos
    state = 0
    for i, c in enumerate(text):
        state = ((state << 1) | 1) & char_masks.get(c, 0)
        if state & (1 << (m - 1)):
            return i - m + 1   # match ends at position i
    return -1   # not found
```

---

## 17. Quick Reference Card

### Core Operations at a Glance

```
GET bit k:         (n >> k) & 1
SET bit k:          n | (1 << k)
CLEAR bit k:        n & ~(1 << k)
TOGGLE bit k:       n ^ (1 << k)
UPDATE bit k to v:  (n & ~(1 << k)) | (v << k)

EXTRACT field [start, start+len):  (n >> start) & ((1 << len) - 1)
SET field:                         (n & ~mask) | (value << start)
  where mask = ((1 << len) - 1) << start
```

### Number Properties

```
Is power of 2?      n > 0 && (n & (n-1)) == 0
Is odd?             n & 1
Next power of 2:    1 << n.bit_length()   (Python)

Lowest set bit:     n & (-n)
Clear lowest set:   n & (n-1)
Toggle trailing 0s: n | (n-1)

Count set bits:     bin(n).count('1')          (Python)
Bit length:         n.bit_length()              (Python)
```

### Arithmetic Shortcuts

```
x * 2^k     →   x << k
x / 2^k     →   x >> k   (signed: arithmetic; unsigned: logical)
x % 2^k     →   x & (2^k - 1)
x * 10      →   (x << 3) + (x << 1)    [= x*8 + x*2]
x * 7       →   (x << 3) - x
|x|         →   (x ^ (x >> 31)) - (x >> 31)   [32-bit, branchless]
-x          →   ~x + 1   (two's complement)
```

### Set Operations on Bitmasks

```
Union:           A | B
Intersection:    A & B
Difference:      A & ~B
Symmetric diff:  A ^ B
Complement:      ~A  (then mask to universe size)
Subset check:    (A & B) == A   (A ⊆ B)
Empty set:       A == 0
Add element i:   A | (1 << i)
Remove element:  A & ~(1 << i)
Has element i:   (A >> i) & 1
Enumerate subs:  sub = mask; while sub: yield sub; sub = (sub-1) & mask
```

### Operator Precedence (Low → High)

```
|    (OR)       ← lowest
^    (XOR)
&    (AND)
<< >> (shifts)
~    (NOT)      ← highest unary
⚠️  Always use parentheses with comparisons: (x & mask) == 0
```

### Language Cheat Sheet

| Operation | Python | Java | C/C++ |
|-----------|--------|------|-------|
| Popcount | `bin(n).count('1')` | `Integer.bitCount(n)` | `__builtin_popcount(n)` |
| CTZ | `(n&-n).bit_length()-1` | `Integer.numberOfTrailingZeros(n)` | `__builtin_ctz(n)` |
| CLZ | `n.bit_length()` via calc | `Integer.numberOfLeadingZeros(n)` | `__builtin_clz(n)` |
| Logical >> | `(n & 0xFFFFFFFF) >> k` | `n >>> k` | `(unsigned)n >> k` |
| Rotate left | `((n<<k)\|(n>>(32-k))) & 0xFFFFFFFF` | `Integer.rotateLeft(n,k)` | `_rotl(n,k)` |
| Byte swap | `int.from_bytes(...)` | `Integer.reverseBytes(n)` | `__builtin_bswap32(n)` |
| Bit length | `n.bit_length()` | `32 - Integer.numberOfLeadingZeros(n)` | `32 - __builtin_clz(n)` |

### Common Masks

```
0x1         = ...0001   single bit 0
0xF         = ...1111   lower nibble (4 bits)
0xFF        = lower byte
0xFFFF      = lower 2 bytes
0x7FFFFFFF  = max int32 (all bits except sign)
0x80000000  = sign bit of int32
0x55555555  = 0101...  alternating bits (even positions)
0xAAAAAAAA  = 1010...  alternating bits (odd positions)
0x33333333  = 00110011... pairs
0x0F0F0F0F  = nibble pairs
0x00FF00FF  = byte pairs
```

### Common Pitfalls

```
❌ Forgetting operator precedence:
   (n & 0xFF == 0)  evaluates as  n & (0xFF == 0) = n & 0 = 0 (always!)
✅  (n & 0xFF) == 0

❌ Shifting by word size:
   n >> 32  is UB in C for 32-bit int
✅  Use n == 0 check first, or shift by at most (width - 1)

❌ Left-shifting into sign bit in C:
   1 << 31  is UB for signed int in C
✅  (uint32_t)1 << 31  or use unsigned types for bit manipulation

❌ Mixing signed/unsigned in C comparisons:
   (int)-1 < (unsigned)1  is FALSE  (-1 becomes huge unsigned number)
✅  Cast explicitly, or use same signedness

❌ XOR swap with same variable:
   a ^= a; a ^= a; a ^= a;  → a becomes 0!
✅  Check a != b before XOR swap, or just use a temp

❌ Python: assuming >> fills with 0s for negative numbers
   -8 >> 2  = -2  (arithmetic, fills with 1s)
✅  Use  (n & 0xFFFFFFFF) >> k  for logical shift in Python

❌ Not masking after NOT in Python:
   ~0xFF = -256  (Python uses arbitrary precision!)
✅  ~0xFF & 0xFF = 0  (mask to desired width)
```

---

*For deeper study: "Hacker's Delight" by Henry S. Warren Jr. is the definitive reference. "Programming Pearls" by Jon Bentley covers clever bit-level problem solving. For competitive programming, Codeforces and LeetCode have extensive bitmask DP problem sets.*