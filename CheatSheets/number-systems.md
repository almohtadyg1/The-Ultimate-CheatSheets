# The Complete Number Systems Cheat Sheet
### From Zero to Expert

---

## 1. What Are Number Systems?

A **number system** is a way of representing quantities using a set of symbols (digits) and a **base** (radix) that defines how many unique digits exist.

The **base** tells you the place value of each position:

```
In base B, a number:  d₄ d₃ d₂ d₁ d₀

= d₄ × B⁴  +  d₃ × B³  +  d₂ × B²  +  d₁ × B¹  +  d₀ × B⁰
```

### Why Multiple Number Systems?

| System | Base | Digits | Primary Use |
|--------|------|--------|-------------|
| Decimal | 10 | 0–9 | Human everyday math |
| Binary | 2 | 0, 1 | Computer hardware (transistors = on/off) |
| Hexadecimal | 16 | 0–9, A–F | Compact binary notation, colors, memory |
| Octal | 8 | 0–7 | Unix file permissions, legacy systems |
| BCD | 10 (encoded in 4 bits) | 0–9 | Financial systems, displays |
| Base-64 | 64 | A–Z, a–z, 0–9, +, / | Data encoding, URLs |

Computers use binary because transistors have exactly **two stable states**: on (1) and off (0). Everything a computer does — storing text, playing video, running code — ultimately reduces to manipulating 1s and 0s.

---

## 2. The Four Core Systems

### Decimal — Base 10

The system humans use naturally. Ten digits: **0 1 2 3 4 5 6 7 8 9**.

```
4,  2,  7  (decimal)
│   │   │
│   │   └── 7 × 10⁰ =    7
│   └────── 2 × 10¹ =   20
└────────── 4 × 10²ˢ =  400
                       ─────
                         427
```

Notation: plain number, or subscript 10 → `427₁₀`

---

### Binary — Base 2

Two digits: **0 1**. Each digit is called a **bit** (Binary digIT).

```
1  0  1  1  0  1  (binary)
│  │  │  │  │  │
│  │  │  │  │  └── 1 × 2⁰ =  1
│  │  │  │  └───── 0 × 2¹ =  0
│  │  │  └──────── 1 × 2² =  4
│  │  └─────────── 1 × 2³ =  8
│  └────────────── 0 × 2⁴ =  0
└───────────────── 1 × 2⁵ = 32
                           ───
                            45
```

So `101101₂ = 45₁₀`

Notation: `0b101101` (programming), `101101₂`, or `101101₂`

---

### Hexadecimal — Base 16

Sixteen digits: **0 1 2 3 4 5 6 7 8 9 A B C D E F**

```
A = 10,  B = 11,  C = 12,  D = 13,  E = 14,  F = 15
```

```
2  A  F  (hex)
│  │  │
│  │  └── F × 16⁰ = 15 ×   1 =   15
│  └───── A × 16¹ = 10 ×  16 =  160
└──────── 2 × 16² =  2 × 256 =  512
                              ─────
                                687
```

So `2AF₁₆ = 687₁₀`

Notation: `0x2AF` (programming), `2AF₁₆`, `#2AF` (CSS colors)

**Why hex?** One hex digit = exactly 4 bits. Two hex digits = 1 byte. This makes hex a compact, human-readable representation of binary data.

```
Binary:  1111  0000  1010  1100
Hex:       F     0     A     C   → 0xF0AC
```

---

### Octal — Base 8

Eight digits: **0 1 2 3 4 5 6 7**. One octal digit = exactly 3 bits.

```
7  3  5  (octal)
│  │  │
│  │  └── 5 × 8⁰ =   5
│  └───── 3 × 8¹ =  24
└──────── 7 × 8² = 448
                  ─────
                    477
```

So `735₈ = 477₁₀`

Notation: `0735` (C/Python legacy), `0o735` (Python 3), `735₈`

**Real use:** Unix/Linux file permissions. `chmod 755` → `111 101 101` in binary → rwxr-xr-x

---

## 3. Counting in Every Base

### Side-by-Side Counting Table

| Decimal | Binary | Octal | Hex |
|---------|--------|-------|-----|
| 0  | 0000 | 0  | 0 |
| 1  | 0001 | 1  | 1 |
| 2  | 0010 | 2  | 2 |
| 3  | 0011 | 3  | 3 |
| 4  | 0100 | 4  | 4 |
| 5  | 0101 | 5  | 5 |
| 6  | 0110 | 6  | 6 |
| 7  | 0111 | 7  | 7 |
| 8  | 1000 | 10 | 8 |
| 9  | 1001 | 11 | 9 |
| 10 | 1010 | 12 | A |
| 11 | 1011 | 13 | B |
| 12 | 1100 | 14 | C |
| 13 | 1101 | 15 | D |
| 14 | 1110 | 16 | E |
| 15 | 1111 | 17 | F |
| 16 | 0001 0000 | 20 | 10 |
| 32 | 0010 0000 | 40 | 20 |
| 64 | 0100 0000 | 100 | 40 |
| 128 | 1000 0000 | 200 | 80 |
| 255 | 1111 1111 | 377 | FF |
| 256 | 1 0000 0000 | 400 | 100 |

### Powers of 2 — Memorize These

| Power | Value | Common Name |
|-------|-------|-------------|
| 2⁰ | 1 | — |
| 2¹ | 2 | — |
| 2² | 4 | — |
| 2³ | 8 | — |
| 2⁴ | 16 | — |
| 2⁵ | 32 | — |
| 2⁶ | 64 | — |
| 2⁷ | 128 | — |
| 2⁸ | 256 | — |
| 2⁹ | 512 | — |
| 2¹⁰ | 1,024 | 1 Kibibyte (KiB) |
| 2¹⁶ | 65,536 | Max unsigned 16-bit |
| 2²⁰ | 1,048,576 | 1 MiB |
| 2³² | 4,294,967,296 | Max unsigned 32-bit |
| 2⁶⁴ | 18,446,744,073,709,551,616 | Max unsigned 64-bit |

---

## 4. Conversions: The Complete Guide

### Method 1: Any Base → Decimal (Expand & Sum)

Multiply each digit by its place value and sum.

```
Binary → Decimal:
1  1  0  1  0  0  1₂

= 1×2⁶ + 1×2⁵ + 0×2⁴ + 1×2³ + 0×2² + 0×2¹ + 1×2⁰
= 64   + 32   +  0   +  8   +  0   +  0   +  1
= 105₁₀

Hex → Decimal:
3  B  7₁₆

= 3×16² + 11×16¹ + 7×16⁰
= 3×256 + 11×16  + 7×1
= 768   + 176    + 7
= 951₁₀

Octal → Decimal:
6  2  4₈

= 6×8² + 2×8¹ + 4×8⁰
= 6×64 + 2×8  + 4×1
= 384  + 16   + 4
= 404₁₀
```

---

### Method 2: Decimal → Any Base (Repeated Division)

Divide by the target base repeatedly. Read remainders **bottom to top**.

```
Decimal 105 → Binary:

105 ÷ 2 = 52  remainder 1  ← least significant bit (rightmost)
 52 ÷ 2 = 26  remainder 0
 26 ÷ 2 = 13  remainder 0
 13 ÷ 2 =  6  remainder 1
  6 ÷ 2 =  3  remainder 0
  3 ÷ 2 =  1  remainder 1
  1 ÷ 2 =  0  remainder 1  ← most significant bit (leftmost)

Read remainders bottom to top: 1101001₂  ✅

Decimal 951 → Hex:

951 ÷ 16 = 59  remainder 7   ← rightmost
 59 ÷ 16 =  3  remainder 11  → B
  3 ÷ 16 =  0  remainder 3   ← leftmost

Read bottom to top: 3B7₁₆  ✅

Decimal 404 → Octal:

404 ÷ 8 = 50  remainder 4
 50 ÷ 8 =  6  remainder 2
  6 ÷ 8 =  0  remainder 6

Read bottom to top: 624₈  ✅
```

---

### Method 3: Binary ↔ Hex (The Fast Way)

Group binary digits into sets of **4** (from right). Each group = 1 hex digit.

```
Binary → Hex:
1111 0000 1010 1100₂

 1111 = F
 0000 = 0
 1010 = A
 1100 = C

= F0AC₁₆  ✅

Hex → Binary:
2 B 9 E₁₆

2 = 0010
B = 1011
9 = 1001
E = 1110

= 0010 1011 1001 1110₂  ✅
```

**No arithmetic needed! Just lookup the 4-bit groups.**

---

### Method 4: Binary ↔ Octal (Groups of 3)

Group binary into sets of **3** (from right). Each group = 1 octal digit.

```
Binary → Octal:
110 111 010 001₂

110 = 6
111 = 7
010 = 2
001 = 1

= 6721₈  ✅

Octal → Binary:
5  4  2₈

5 = 101
4 = 100
2 = 010

= 101 100 010₂  ✅
```

---

### Method 5: Hex ↔ Octal (Via Binary)

Convert Hex → Binary (groups of 4), then regroup → Octal (groups of 3).

```
3B₁₆ → Octal:

Step 1:  3 = 0011,  B = 1011  →  0011 1011₂
Step 2:  Regroup in 3s: 000 111 011₂
Step 3:  0=0, 7=7, 3=3  →  073₈  ✅
```

---

### Conversion Cheat Map

```
         ┌─────────────────────┐
         │       DECIMAL       │
         │       (Base 10)     │
         └──────────┬──────────┘
          ÷10,×10   │ Expand/Divide
         ┌──────────▼──────────┐
    ┌────►       BINARY        ◄────┐
    │    │       (Base 2)      │    │
    │    └──────────┬──────────┘    │
    │   Group of 3  │ Group of 4    │
    │    ┌──────────▼──────────┐    │
    │    │       OCTAL         │    │
    │    │       (Base 8)      │    │
    │    └─────────────────────┘    │
    │                               │
    │    ┌─────────────────────┐    │
    └────┤    HEXADECIMAL      ├────┘
         │       (Base 16)     │
         └─────────────────────┘
```

---

### Conversion Practice Table

| Decimal | Binary | Hex | Octal |
|---------|--------|-----|-------|
| 25 | 0001 1001 | 19 | 31 |
| 42 | 0010 1010 | 2A | 52 |
| 100 | 0110 0100 | 64 | 144 |
| 200 | 1100 1000 | C8 | 310 |
| 255 | 1111 1111 | FF | 377 |
| 1000 | 0011 1110 1000 | 3E8 | 1750 |

---

## 5. Binary Arithmetic

### Binary Addition

Rules: `0+0=0`, `0+1=1`, `1+0=1`, `1+1=10` (write 0, carry 1), `1+1+1=11` (write 1, carry 1)

```
  1 1 0 1₂  (13)
+ 0 1 1 1₂  ( 7)
───────────
carries: 1 1 1 0

  1 1 0 1
+ 0 1 1 1
─────────
1 0 1 0 0₂  (20)  ✅

Step by step (right to left):
Col 0: 1+1 = 10 → write 0, carry 1
Col 1: 0+1+1(carry) = 10 → write 0, carry 1
Col 2: 1+1+1(carry) = 11 → write 1, carry 1
Col 3: 1+0+1(carry) = 10 → write 0, carry 1
Col 4: carry 1 → write 1
Result: 10100₂ = 20₁₀  ✅
```

### Binary Subtraction

Rules: `0-0=0`, `1-0=1`, `1-1=0`, `10-1=1` (borrow from left)

```
  1 1 0 1₂  (13)
- 0 1 1 1₂  ( 7)
───────────

Col 0: 1-1=0
Col 1: 0-1 → borrow: 10-1=1, borrow from col 2
Col 2: 1-1(due to borrow)-1 → 0-1 → borrow: 10-1=0... borrow from col 3
Col 3: 1-0-1(due to borrow)=0... then remaining 1

Result: 0110₂ = 6₁₀  ✅

Tip: For subtraction, use two's complement addition instead (see Section 7).
```

### Binary Multiplication

Multiply each bit of the bottom number by the entire top number, shift left, then add.

```
  1 0 1₂  (5)
× 0 1 1₂  (3)
─────────

  1 0 1  × 1  (2⁰) =   1 0 1
  1 0 1  × 1  (2¹) =  1 0 1 0   (shift left 1)
  1 0 1  × 0  (2²) = 0 0 0 0 0  (shift left 2, but 0 so ignored)

     1 0 1
+  1 0 1 0
──────────
  0 1 1 1 1₂ = 15₁₀  ✅  (5 × 3 = 15)
```

### Binary Division

Like long division in decimal but with only 0 and 1.

```
1100₂ ÷ 11₂  =  12 ÷ 3  =  ?

   1 0 0
   ─────
11 ) 1 1 0 0
     1 1        (11 goes into 11 once: 1×11=11)
     ────
       0 0 0
         0      (11 doesn't go into 0: write 0)
         ─
         0 0
          0     (11 doesn't go into 00: write 0)
Quotient: 100₂ = 4₁₀  ✅  (12 ÷ 3 = 4)
```

### Overflow Detection

```
Unsigned 8-bit max = 255 = 11111111₂
If result > 255, overflow occurs.

11111111 + 00000001 = (1)00000000 → overflow! Result wraps to 0.

Signed 8-bit range = -128 to 127
Overflow when two positives sum to negative, or two negatives sum to positive.
```

---

## 6. Hexadecimal Arithmetic

### Hex Addition

Add digit pairs. If sum ≥ 16, subtract 16 and carry 1.

```
  2 A F₁₆  (687)
+ 1 3 C₁₆  (316)
───────────

Col 0: F+C = 15+12 = 27 → 27-16=11=B, carry 1
Col 1: A+3+1 = 10+3+1 = 14 = E, no carry
Col 2: 2+1 = 3

Result: 3EB₁₆ = 1003₁₀  ✅  (687+316=1003)
```

### Hex Subtraction

```
  F A 0₁₆  (4000)
- 3 B 5₁₆  ( 949)
───────────

Col 0: 0-5 → borrow: 16+0-5 = 11 = B, borrow 1
Col 1: A-B-1 = 10-11-1=-2 → borrow: 16+10-11-1=14=E, borrow 1
Col 2: F-3-1 = 15-3-1 = 11 = B

Result: BEB₁₆ = 3051₁₀  ✅  (4000-949=3051)
```

### Hex Multiplication

```
2A₁₆ × 3₁₆:

A×3 = 10×3 = 30 = 1E₁₆  → write E, carry 1
2×3 = 6, + carry 1 = 7

Result: 7E₁₆ = 126₁₀  ✅  (42×3=126)
```

---

## 7. Negative Numbers in Binary

Computers represent negative numbers in one of several ways.

### Sign-Magnitude

The **leftmost bit** (MSB) is the sign: 0 = positive, 1 = negative.

```
+5  =  0101
-5  =  1101

8-bit examples:
+127 = 0111 1111
-127 = 1111 1111
 +0  = 0000 0000
 -0  = 1000 0000  ← Problem: two representations of zero!
```

Problems: Two zeros, complicated arithmetic hardware. Rarely used.

---

### One's Complement

Flip every bit to negate.

```
+5  =  0000 0101
-5  =  1111 1010  (flip all bits)

+127 = 0111 1111
-127 = 1000 0000
  +0 = 0000 0000
  -0 = 1111 1111  ← Still two zeros!
```

Slightly better, but still two zeros. Mostly obsolete.

---

### Two's Complement (Modern Standard)

**Flip all bits, then add 1.** Used in virtually all modern CPUs.

```
To negate a number: flip all bits, add 1.

+5  =  0000 0101
       1111 1010  (flip)
+      0000 0001  (add 1)
     ───────────
-5  =  1111 1011  ✅

Verify: +5 + (-5) should = 0
  0000 0101
+ 1111 1011
───────────
1 0000 0000  → drop the carry → 0000 0000  ✅ 

8-bit two's complement range: -128 to +127
  +127 = 0111 1111
  +1   = 0000 0001
   0   = 0000 0000
  -1   = 1111 1111
  -2   = 1111 1110
 -128  = 1000 0000  ← Note: no positive 128!
```

### Two's Complement Quick Conversion (Shortcut)

Copy bits from right up to and including the **first 1**, then flip all remaining bits.

```
Find two's complement of: 0110 1000

From right, copy up to first 1:     _ _ _ _ 1 0 0 0  (copy "1000")
Flip the rest:            1 0 0 1   _ _ _ _ _ _ _ _  (flip "0110" → "1001")
Result:                   1 0 0 1   1 0 0 0  = -104 in decimal ✅
```

### Detecting Overflow in Two's Complement

```
Overflow occurs when:
  Positive + Positive = Negative (result > max positive)
  Negative + Negative = Positive (result < min negative)

Example (4-bit, range -8 to +7):
  0111 (+7)
+ 0001 (+1)
─────
  1000 → interpreted as -8!  ← OVERFLOW

No overflow:
  0011 (+3)
+ 0100 (+4)
─────
  0111 (+7)  ← correct
```

### Sign Extension

Extending a binary number to more bits while preserving its value:

```
For positive numbers: pad with 0s on the left
  +5 in 4-bit: 0101
  +5 in 8-bit: 0000 0101  ✅

For negative numbers (two's complement): pad with 1s on the left
  -5 in 4-bit: 1011
  -5 in 8-bit: 1111 1011  ✅

Rule: replicate the sign bit (MSB) when extending.
```

### Comparison of Representations

| Value | Sign-Magnitude | One's Complement | Two's Complement |
|-------|----------------|------------------|------------------|
| +7 | 0111 | 0111 | 0111 |
| +1 | 0001 | 0001 | 0001 |
| +0 | 0000 | 0000 | 0000 |
| -0 | 1000 | 1111 | N/A |
| -1 | 1001 | 1110 | 1111 |
| -7 | 1111 | 1000 | 1001 |
| -8 | N/A | N/A | 1000 |
| Range (4-bit) | -7 to +7 | -7 to +7 | -8 to +7 |

---

## 8. Fractions & Floating Point

### Binary Fractions

After the binary point, positions have **negative** powers of 2:

```
  1  0  1 . 1  1  0₂

  │  │  │   │  │  │
  │  │  │   │  │  └── 0 × 2⁻³ = 0 × 0.125 = 0.000
  │  │  │   │  └───── 1 × 2⁻² = 1 × 0.25  = 0.250
  │  │  │   └──────── 1 × 2⁻¹ = 1 × 0.5   = 0.500
  │  │  └──────────── 1 × 2⁰  = 1 × 1     = 1.000
  │  └─────────────── 0 × 2¹  = 0 × 2     = 0.000
  └────────────────── 1 × 2²  = 1 × 4     = 4.000
                                           ───────
                                             5.750₁₀
```

### Decimal Fraction → Binary Fraction

Multiply by 2 repeatedly. Integer part of each result is the next binary digit.

```
Convert 0.625 to binary:

0.625 × 2 = 1.25  → integer part = 1  (first bit after point)
0.25  × 2 = 0.50  → integer part = 0
0.50  × 2 = 1.00  → integer part = 1  (stop: remainder is 0)

Result: 0.101₂  ✅  (check: 0.5 + 0 + 0.125 = 0.625)

Convert 0.1 to binary (a famous problem):

0.1 × 2 = 0.2 → 0
0.2 × 2 = 0.4 → 0
0.4 × 2 = 0.8 → 0
0.8 × 2 = 1.6 → 1
0.6 × 2 = 1.2 → 1
0.2 × 2 = 0.4 → 0  (repeats!)
...

0.1₁₀ = 0.0001100110011...₂ (repeating!)

This is why 0.1 + 0.2 ≠ 0.3 in floating point! ⚠️
```

### IEEE 754 Floating Point

The universal standard for representing real numbers in computers.

#### 32-bit Single Precision

```
 31  30─────23  22───────────────────────0
 ┌─┬──────────┬────────────────────────────┐
 │S│ Exponent │         Mantissa           │
 │1│  8 bits  │          23 bits           │
 └─┴──────────┴────────────────────────────┘

S = Sign bit (0 = positive, 1 = negative)
Exponent = biased by 127 (stored exponent = actual exponent + 127)
Mantissa = fractional part (leading 1 is implicit)

Value = (-1)^S × 1.Mantissa × 2^(Exponent - 127)
```

#### 64-bit Double Precision

```
 63  62────52  51──────────────────────────0
 ┌─┬──────────┬────────────────────────────┐
 │S│ Exponent │         Mantissa           │
 │1│ 11 bits  │          52 bits           │
 └─┴──────────┴────────────────────────────┘

Bias = 1023
Value = (-1)^S × 1.Mantissa × 2^(Exponent - 1023)
```

#### IEEE 754 Example

```
Encode -6.5 as 32-bit float:

Step 1: Sign = 1 (negative)

Step 2: Convert 6.5 to binary:
        6 = 110₂, 0.5 = 0.1₂  →  6.5 = 110.1₂

Step 3: Normalize (shift to 1.something × 2^n):
        110.1 = 1.101 × 2²

Step 4: Exponent = 2 + 127 = 129 = 10000001₂

Step 5: Mantissa = 101 followed by 20 zeros = 10100000000000000000000

Result: 1  10000001  10100000000000000000000
        S  Exponent  Mantissa
= 0xC0D00000
```

#### Special Values

| Pattern | Value |
|---------|-------|
| Exp = 0, Mantissa = 0 | ±0.0 |
| Exp = 255/2047, Mantissa = 0 | ±Infinity |
| Exp = 255/2047, Mantissa ≠ 0 | NaN (Not a Number) |
| Exp = 0, Mantissa ≠ 0 | Denormalized (subnormal) numbers |

#### Floating Point Gotchas

```python
# These are all true in most languages:
0.1 + 0.2 == 0.30000000000000004  # NOT 0.3!
1/3        == 0.3333333333333333   # Truncated
1e308 * 10 == inf                  # Overflow to infinity
0.0 / 0.0  == NaN                  # Not a number

# Safe comparison:
abs(a - b) < 1e-9  # Use epsilon comparison, not ==
```

#### Precision

| Type | Significant Decimal Digits | Range |
|------|--------------------------|-------|
| float32 | ~7 | ±3.4 × 10³⁸ |
| float64 | ~15–16 | ±1.8 × 10³⁰⁸ |
| float128 | ~18–19 | ±1.2 × 10⁴⁹³² |

---

## 9. Bitwise Operations

Bitwise operations work on individual bits of integers.

### AND — `&`

Output bit is 1 only if **both** input bits are 1.

```
Truth table:     0 & 0 = 0
                 0 & 1 = 0
                 1 & 0 = 0
                 1 & 1 = 1

Example:
  1100 1010  (202)
& 1111 0000  (240)
───────────
  1100 0000  (192)

Use: Masking — extract specific bits, check if bit is set.
```

### OR — `|`

Output bit is 1 if **either** input bit is 1.

```
Truth table:     0 | 0 = 0
                 0 | 1 = 1
                 1 | 0 = 1
                 1 | 1 = 1

Example:
  1100 1010  (202)
| 0000 1111  ( 15)
───────────
  1100 1111  (207)

Use: Setting bits — force specific bits to 1.
```

### XOR — `^`

Output bit is 1 if input bits are **different**.

```
Truth table:     0 ^ 0 = 0
                 0 ^ 1 = 1
                 1 ^ 0 = 1
                 1 ^ 1 = 0

Example:
  1100 1010  (202)
^ 1111 0000  (240)
───────────
  0011 1010  ( 58)

Key property: a ^ a = 0,  a ^ 0 = a,  XOR is its own inverse
Use: Toggle bits, detect differences, swapping without temp variable.
```

### NOT — `~`

Flip every bit (bitwise complement).

```
~1100 1010 = 0011 0101

In two's complement: ~n = -(n+1)
~5 = -6,  ~0 = -1,  ~(-1) = 0
```

### Left Shift — `<<`

Shift bits left by n positions. Fill right with 0s. Equivalent to multiplying by 2ⁿ.

```
0000 1011  (11)
<< 2
───────────
0010 1100  (44)   11 × 2² = 11 × 4 = 44  ✅

General: x << n  =  x × 2ⁿ  (as long as no overflow)
```

### Right Shift — `>>`

Shift bits right by n positions. Equivalent to dividing by 2ⁿ (integer division).

```
Logical right shift (unsigned): fill with 0s
1100 1100  (204)
>> 2
───────────
0011 0011  (51)   204 / 4 = 51  ✅

Arithmetic right shift (signed): fill with sign bit
1100 1100  (-52 in two's complement)
>> 2
───────────
1111 0011  (-13)   ⌊-52 / 4⌋ = -13  ✅

General: x >> n  =  ⌊x / 2ⁿ⌋
```

### Bitwise Operation Summary

| Operation | Symbol | Key Use |
|-----------|--------|---------|
| AND | `&` | Clear bits / check bits |
| OR | `\|` | Set bits |
| XOR | `^` | Toggle bits / compare |
| NOT | `~` | Flip all bits |
| Left Shift | `<<` | Multiply by 2ⁿ |
| Right Shift | `>>` | Divide by 2ⁿ |

---

## 10. Bit Manipulation Tricks

### Check if Bit k is Set

```
(n >> k) & 1  == 1   → bit k is set
(n >> k) & 1  == 0   → bit k is clear

Or: n & (1 << k) != 0
```

### Set Bit k (Force to 1)

```
n | (1 << k)

Example: set bit 3 of 0000 1010
  0000 1010
| 0000 1000  (1 << 3)
───────────
  0000 1010  → bit 3 already set  ✅
```

### Clear Bit k (Force to 0)

```
n & ~(1 << k)

Example: clear bit 1 of 0000 1010
  0000 1010
& 1111 1101  (~(1 << 1))
───────────
  0000 1000  ✅
```

### Toggle Bit k (Flip)

```
n ^ (1 << k)

Example: toggle bit 3 of 0000 1010
  0000 1010
^ 0000 1000  (1 << 3)
───────────
  0000 0010  ✅
```

### Check if n is a Power of 2

```
n > 0 && (n & (n - 1)) == 0

Powers of 2 have exactly one bit set:
8  = 1000
8-1= 0111
8 & 7 = 0000  → IS a power of 2  ✅

6  = 0110
6-1= 0101
6 & 5 = 0100 ≠ 0 → NOT a power of 2  ✅
```

### Isolate the Lowest Set Bit

```
n & (-n)   or   n & (~n + 1)

Example: n = 0110 1100
-n (two's complement of 0110 1100) = 1001 0100

  0110 1100
& 1001 0100
───────────
  0000 0100  ← lowest set bit isolated  ✅
```

### Clear the Lowest Set Bit

```
n & (n - 1)

Example: n = 0110 1100
n-1       = 0110 1011

  0110 1100
& 0110 1011
───────────
  0110 1000  ← lowest set bit cleared  ✅

Application: Count set bits (Brian Kernighan's algorithm):
count = 0
while n:
    n = n & (n-1)   # remove lowest set bit
    count += 1
```

### Count Set Bits (Popcount)

```python
# Method 1: Brian Kernighan's O(k) where k = set bits
def count_bits(n):
    count = 0
    while n:
        n &= n - 1
        count += 1
    return count

# Method 2: Built-in (fastest)
bin(n).count('1')     # Python
__builtin_popcount(n) # C/C++
Integer.bitCount(n)   # Java
```

### Swap Two Values Without Temp

```python
# Using XOR — works only for integers!
a ^= b
b ^= a
a ^= b
# Proof: After step 1: a = a^b
#        After step 2: b = (a^b)^b = a
#        After step 3: a = (a^b)^a = b  ✅
```

### Find the Only Non-Duplicate

```python
# In an array where every element appears twice except one:
result = 0
for x in arr:
    result ^= x
# XOR cancels pairs: a^a=0, 0^b=b → only the unique element remains
```

### Common Bit Masks

```
0xFF       = 1111 1111  (8-bit mask / last byte)
0xF0       = 1111 0000  (upper nibble)
0x0F       = 0000 1111  (lower nibble)
0x80000000 = sign bit of 32-bit int
0x7FFFFFFF = max positive 32-bit signed int
0xFFFFFFFF = all 1s for 32-bit
```

### Fast Arithmetic with Shifts

```
n * 2  → n << 1
n * 4  → n << 2
n * 8  → n << 3
n / 2  → n >> 1
n / 4  → n >> 2
n % 2  → n & 1       (is n odd?)
n % 4  → n & 3
n % 8  → n & 7
n % 2^k → n & (2^k - 1)
```

### Bit Reversal

```
Reverse bits of 8-bit number:
Original:  1 0 1 1 0 0 1 0
Reversed:  0 1 0 0 1 1 0 1

def reverse_bits_8(n):
    result = 0
    for _ in range(8):
        result = (result << 1) | (n & 1)
        n >>= 1
    return result
```

---

## 11. Data Sizes & Real-World Usage

### Fundamental Units

| Unit | Bits | Value | Common Use |
|------|------|-------|------------|
| Bit | 1 | 0 or 1 | Smallest unit |
| Nibble | 4 | 0–15 | One hex digit |
| Byte | 8 | 0–255 | One character (ASCII) |
| Word | 16/32/64 | Architecture-dependent | CPU register size |
| Kilobyte (KB) | — | 1,000 bytes (SI) | File sizes |
| Kibibyte (KiB) | — | 1,024 bytes | Memory |
| Megabyte (MB) | — | 1,000,000 bytes | File sizes |
| Mebibyte (MiB) | — | 1,048,576 bytes | Memory |
| Gigabyte (GB) | — | 10⁹ bytes | Drive sizes |
| Gibibyte (GiB) | — | 2³⁰ bytes | RAM |
| Terabyte (TB) | — | 10¹² bytes | Storage |

> **KB vs KiB:** Hard drive makers use 1 KB = 1000 bytes. OS/RAM uses 1 KiB = 1024 bytes. That's why a "500 GB" drive shows as ~465 GiB in Windows.

### Integer Type Ranges

| Type | Bits | Signed Range | Unsigned Range |
|------|------|-------------|----------------|
| int8 | 8 | -128 to 127 | 0 to 255 |
| int16 | 16 | -32,768 to 32,767 | 0 to 65,535 |
| int32 | 32 | -2,147,483,648 to 2,147,483,647 | 0 to 4,294,967,295 |
| int64 | 64 | -9.2×10¹⁸ to 9.2×10¹⁸ | 0 to 1.8×10¹⁹ |

### Real-World Applications

**Memory Addresses:**
```
32-bit systems can address: 2³² = 4 GiB of RAM max
64-bit systems can address: 2⁶⁴ = 16 exabytes (theoretical)
```

**Colors:**
```
RGB hex color: #RRGGBB
#FF5733 → R=255, G=87, B=51
Each channel: 8 bits → 256 shades → 256³ = 16,777,216 total colors

RGBA: #RRGGBBAA  (AA = alpha/opacity, 00=transparent, FF=opaque)
```

**IP Addresses:**
```
IPv4: 32 bits, 4 octets in decimal
192.168.1.1 → 11000000.10101000.00000001.00000001
             → 0xC0A80101

IPv6: 128 bits, 8 groups of 4 hex digits
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

**File Permissions (Unix/Linux):**
```
chmod 755:
7 = 111₂ = rwx (owner: read, write, execute)
5 = 101₂ = r-x (group: read, no write, execute)
5 = 101₂ = r-x (others: read, no write, execute)

chmod 644:
6 = 110₂ = rw- (owner: read, write, no execute)
4 = 100₂ = r-- (group: read only)
4 = 100₂ = r-- (others: read only)
```

**Networking Subnets:**
```
CIDR notation: 192.168.1.0/24
/24 → subnet mask has 24 ones: 11111111.11111111.11111111.00000000
    → 255.255.255.0
    → 256 addresses (254 usable), last octet varies
```

**Character Encoding:**
```
ASCII: 7-bit, 128 characters (0-127)
  'A' = 65 = 0x41 = 0100 0001₂
  'a' = 97 = 0x61 = 0110 0001₂
  '0' = 48 = 0x30 = 0011 0000₂

Key insight: 'a' - 'A' = 32 = 0x20 = 0010 0000₂
Toggling bit 5 switches case!

Unicode: up to 21 bits per character
UTF-8: variable-width encoding (1-4 bytes per character)
```

---

## 12. Other Number Systems

### Binary Coded Decimal (BCD)

Each **decimal digit** is encoded separately in 4 bits. Used in financial systems and displays where exact decimal representation matters.

```
Decimal 429:
4 = 0100
2 = 0010
9 = 1001

BCD: 0100 0010 1001

Note: BCD wastes space (only 10 of 16 possible 4-bit values are used)
But: No decimal-to-binary conversion needed → no rounding errors
```

### Gray Code

Adjacent values differ by **exactly one bit**. Used in rotary encoders, error correction.

```
Decimal | Binary | Gray
   0    |  000   |  000
   1    |  001   |  001
   2    |  010   |  011
   3    |  011   |  010
   4    |  100   |  110
   5    |  101   |  111
   6    |  110   |  101
   7    |  111   |  100

Convert Binary → Gray: gray = binary XOR (binary >> 1)
Convert Gray → Binary: binary MSB = gray MSB; each subsequent bit = previous binary XOR current gray
```

### Base-64 Encoding

Encodes binary data as printable ASCII characters. Used in email attachments, data URLs, JWTs.

```
Characters: A-Z (0-25), a-z (26-51), 0-9 (52-61), + (62), / (63)
Padding: = (no data, pad to multiple of 3 bytes)

Every 3 bytes (24 bits) → 4 base-64 characters (6 bits each)

"Hi!" → 72  105  33  →  01001000 01101001 00100001
Group into 6-bit chunks: 010010 000110 100100 100001
                             S      G      k      h
→ "SGkh"

Size overhead: ~33% larger than original binary
```

### Roman Numerals

A **subtractive** system (not positional).

```
I=1, V=5, X=10, L=50, C=100, D=500, M=1000

When smaller value precedes larger: subtract
IV = 4, IX = 9, XL = 40, XC = 90, CD = 400, CM = 900

MCMXCIX = 1000 + (1000-100) + (100-10) + (10-1) = 1999
```

### Scientific Notation in Different Bases

```
Decimal: 6.022 × 10²³  (Avogadro's number)
Binary:  1.0101 × 2⁵   = IEEE 754 normalized form
Hex:     1.5 × 16²     = 1.5 × 256 = 384
```

### Balanced Ternary (Base 3 with digits -1, 0, 1)

Used in some historical computers (Setun, Soviet Union). Digits: T(−1), 0, 1.

```
+5  =  1TT  (1×9 + (−1)×3 + (−1)×1 = 9−3−1 = 5)
−5  = T11   (−1×9 + 1×3 + 1×1 = −9+3+1 = −5)
Negation: just flip all digits (T↔1, 0↔0)
```

---

## 13. Number Theory Essentials

### Divisibility & Modular Arithmetic

```
a mod m = remainder when a is divided by m

Properties:
(a + b) mod m = ((a mod m) + (b mod m)) mod m
(a × b) mod m = ((a mod m) × (b mod m)) mod m
(a - b) mod m = ((a mod m) - (b mod m) + m) mod m  ← add m to avoid negative

Example: (17 × 23) mod 7
= (17 mod 7) × (23 mod 7)  mod 7
= 3 × 2  mod 7
= 6 mod 7
= 6  ✅  (verify: 391 mod 7 = 55×7 + 6 = 6 ✅)
```

### Greatest Common Divisor (GCD)

```
gcd(a, b) = largest number that divides both a and b

Euclidean Algorithm:
gcd(a, b) = gcd(b, a mod b)   until b = 0

gcd(48, 18):
gcd(48, 18) = gcd(18, 48 mod 18) = gcd(18, 12)
gcd(18, 12) = gcd(12, 18 mod 12) = gcd(12, 6)
gcd(12, 6)  = gcd(6, 12 mod 6)  = gcd(6, 0)
= 6  ✅

def gcd(a, b):
    while b:
        a, b = b, a % b
    return a
```

### Least Common Multiple (LCM)

```
lcm(a, b) = a × b / gcd(a, b)

lcm(12, 18) = 12 × 18 / gcd(12, 18) = 216 / 6 = 36  ✅
```

### Prime Numbers & Factorization

```
A prime has exactly 2 factors: 1 and itself.
2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47...

Fundamental Theorem of Arithmetic:
Every integer > 1 = unique product of primes
360 = 2³ × 3² × 5¹

Primality test: check divisors up to √n (see Big O sheet)

Sieve of Eratosthenes — find all primes up to n:
Start with all numbers 2..n marked prime.
For each prime p, mark all multiples of p as not prime.
Time: O(n log log n)
```

### Fermat's Little Theorem

```
If p is prime and gcd(a, p) = 1:
a^(p-1) ≡ 1 (mod p)

Therefore: a^(p-2) ≡ a⁻¹ (mod p)   ← modular inverse!

Used in: RSA encryption, modular inverse computation
```

### The Relationship Between Binary and Computers

```
Why base 2?
  Transistor: 2 states (on/off) → 1 bit
  More states = harder to distinguish reliably in physical hardware

Why is hex used for debugging?
  1 hex digit = 4 bits exactly
  2 hex digits = 1 byte exactly
  A 32-bit value = 8 hex digits = readable
  vs. 32 binary digits = hard to read
  vs. 10 decimal digits = requires conversion

Memory dump example:
  48 65 6C 6C 6F  →  "Hello" in ASCII
  DE AD BE EF     →  Famous test pattern (0xDEADBEEF)
  CA FE BA BE     →  Java class file magic bytes (0xCAFEBABE)
  FF FE           →  UTF-16 BOM (Byte Order Mark)
```

---

## 14. Expert Reference & Quick Lookup

### All Conversion Methods at a Glance

| From → To | Method |
|-----------|--------|
| Any base → Decimal | Multiply digits by place values, sum |
| Decimal → Any base | Repeated division, read remainders bottom-up |
| Binary → Hex | Group bits into 4s from right, lookup |
| Hex → Binary | Expand each hex digit to 4 bits |
| Binary → Octal | Group bits into 3s from right, lookup |
| Octal → Binary | Expand each octal digit to 3 bits |
| Hex ↔ Octal | Via binary (Hex→Binary→Octal or reverse) |

---

### Powers of 2 Quick Reference

```
2¹=2    2²=4    2³=8     2⁴=16    2⁵=32
2⁶=64   2⁷=128  2⁸=256   2⁹=512   2¹⁰=1024
2¹¹=2048   2¹²=4096   2¹³=8192   2¹⁴=16384   2¹⁵=32768
2¹⁶=65536  2²⁰=1M     2²⁴=16M    2³²=4G       2⁶⁴=18.4E
```

---

### Hex-Decimal-Binary Lookup

| Hex | Dec | Binary |
|-----|-----|--------|
| 0 | 0 | 0000 |
| 1 | 1 | 0001 |
| 2 | 2 | 0010 |
| 3 | 3 | 0011 |
| 4 | 4 | 0100 |
| 5 | 5 | 0101 |
| 6 | 6 | 0110 |
| 7 | 7 | 0111 |
| 8 | 8 | 1000 |
| 9 | 9 | 1001 |
| A | 10 | 1010 |
| B | 11 | 1011 |
| C | 12 | 1100 |
| D | 13 | 1101 |
| E | 14 | 1110 |
| F | 15 | 1111 |

---

### Bit Manipulation One-Liners

```python
n & 1              # Is n odd? (1=odd, 0=even)
n & (n-1)          # Clear lowest set bit
n & (-n)           # Isolate lowest set bit
n | (1 << k)       # Set bit k
n & ~(1 << k)      # Clear bit k
n ^ (1 << k)       # Toggle bit k
(n >> k) & 1       # Get bit k
n & (n-1) == 0     # Is n a power of 2?
~n + 1             # Two's complement negation
n >> 31            # Sign of 32-bit int (-1 or 0)
(a ^ b) < 0        # Do a and b have different signs?
```

---

### Negative Number Representations Summary

| Representation | Negate by | Zero | Range (8-bit) |
|---------------|-----------|------|---------------|
| Sign-Magnitude | Flip MSB | ±0 | -127 to +127 |
| One's Complement | Flip all bits | ±0 | -127 to +127 |
| Two's Complement | Flip all bits, add 1 | One 0 | -128 to +127 |

---

### IEEE 754 Special Values (32-bit)

| Sign | Exponent | Mantissa | Value |
|------|----------|----------|-------|
| 0 | 0000 0000 | 0...0 | +0 |
| 1 | 0000 0000 | 0...0 | -0 |
| 0 | 1111 1111 | 0...0 | +∞ |
| 1 | 1111 1111 | 0...0 | -∞ |
| * | 1111 1111 | ≠0 | NaN |
| 0 | 0000 0000 | ≠0 | +denormal |
| 1 | 0000 0000 | ≠0 | -denormal |

---

### Common Real-World Hex Values

```
0x00000000  = 0 (null pointer)
0x7FFFFFFF  = 2,147,483,647 (max int32)
0x80000000  = -2,147,483,648 (min int32)
0xFFFFFFFF  = -1 (signed) or 4,294,967,295 (unsigned)
0xFF        = 255 (max byte)
0x0A        = 10 = '\n' (newline ASCII)
0x0D        = 13 = '\r' (carriage return)
0x20        = 32 = ' ' (space)
0x41        = 65 = 'A'
0x61        = 97 = 'a'
0x30        = 48 = '0' (digit zero)
0xDEADBEEF  = test/placeholder value (famous in debugging)
0xCAFEBABE  = Java class file magic number
0xFEEDFACE  = Mach-O binary magic number (macOS)
0xBAADF00D  = Microsoft debug fill pattern
```

---

### Summary: When to Use Which System

| Situation | System |
|-----------|--------|
| Human calculations | Decimal |
| Hardware / logic gates | Binary |
| Memory addresses, colors, bytes | Hexadecimal |
| Unix file permissions | Octal |
| Financial / no rounding errors | BCD |
| Binary data in text | Base-64 |
| Rotary encoders, error correction | Gray Code |
| Compact representation of data | Base-64 / Hex |

---

### Common Mistakes & Gotchas

```
❌ 0.1 + 0.2 == 0.3  →  False! Use epsilon comparison
❌ int overflow: 2^31 + 1 wraps to negative in signed 32-bit
❌ Arithmetic right shift vs logical right shift differ for negatives
❌ Leading zeros in octal: 077 in C is octal (63), NOT decimal 77!
❌ ~0 is not 0 — it's -1 in two's complement
❌ Left shifting a negative number is undefined behavior in C
❌ -1 >> 1 is implementation-defined in C (but -1 in most systems)
❌ Mixing signed and unsigned in comparisons causes bugs:
     (unsigned)-1 > 2 is TRUE because -1 wraps to 4294967295

✅ Use >>> for logical right shift in Java (>> is arithmetic)
✅ Python integers have unlimited precision (no overflow!)
✅ Use 0b, 0o, 0x prefixes in Python for binary, octal, hex
✅ hex(n), bin(n), oct(n) in Python convert to string representation
✅ int("FF", 16), int("1010", 2) in Python parse from strings
```

---

*For deeper study: "Computer Organization and Design" by Patterson & Hennessy covers hardware-level number representation. "Hacker's Delight" by Henry S. Warren is the definitive bible of bit manipulation tricks.*
