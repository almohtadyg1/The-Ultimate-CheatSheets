# Number Systems: A Complete Progressive Tutorial

---

## 1. What & Why

A number system is a structured way of representing quantities using a set of symbols and a base — a rule that determines how much each digit position is worth. You already know one number system fluently: decimal (base 10). But computers operate in binary (base 2), and programmers regularly work with hexadecimal (base 16) and octal (base 8).

Why do multiple number systems exist? Computers are built from transistors, which have two stable electrical states: on and off. Binary maps directly to this: 1 = on, 0 = off. Every value stored in memory, every instruction executed, is ultimately a sequence of binary digits. Hexadecimal is a shorthand for binary that is compact enough to read — each hex digit represents exactly four bits. Octal is used in Unix file permissions.

Understanding number systems is not academic. You encounter them in: memory addresses, color codes (#FF5733), IP addresses, bitwise operations, file permissions (chmod 755), protocol headers, cryptographic hashes, and assembly/machine code. When a debugger shows you `0x7ffee4b3c8f0` or a hex dump shows `DE AD BE EF`, you need to read those values.

---

## 2. Mental Model

Every number system works the same way: each digit position has a place value that is a power of the base. Moving one position to the left multiplies the value by the base.

```
Decimal (base 10):
  4 2 7
  │ │ └── 7 × 10⁰ =   7
  │ └──── 2 × 10¹ =  20
  └────── 4 × 10² = 400
                     ─────
                      427

Binary (base 2):
  1 0 1 1 0 1
  │ │ │ │ │ └── 1 × 2⁰ =  1
  │ │ │ │ └──── 0 × 2¹ =  0
  │ │ │ └────── 1 × 2² =  4
  │ │ └──────── 1 × 2³ =  8
  │ └────────── 0 × 2⁴ =  0
  └──────────── 1 × 2⁵ = 32
                           ─
                           45

Hexadecimal (base 16):
  2 D
  │ └── 13 × 16⁰ =  13  (D = 13)
  └──── 2  × 16¹ =  32
                     ─
                     45

All three represent the same value: 45₁₀ = 101101₂ = 2D₁₆
```

The pattern is identical across all bases. Only the base value and digit set change.

---

## 3. Progressive Examples

### Level 1: Converting Between Bases

```python
# Python makes base conversions trivial — learn these functions,
# then learn the manual method so you understand what's happening.

# Decimal → Binary, Octal, Hexadecimal
n = 255
print(bin(n))    # '0b11111111'   (0b prefix)
print(oct(n))    # '0o377'        (0o prefix)
print(hex(n))    # '0xff'         (0x prefix)

# Strip the prefix
print(bin(n)[2:])   # '11111111'
print(oct(n)[2:])   # '377'
print(hex(n)[2:])   # 'ff'

# Format to specific width
print(format(n, '08b'))    # '11111111'   (8 binary digits, zero-padded)
print(format(n, '04x'))    # '00ff'       (4 hex digits, zero-padded)
print(format(n, 'X'))      # 'FF'         (uppercase hex)

# Any base → Decimal
print(int('11111111', 2))    # 255 (binary string to decimal)
print(int('377', 8))         # 255 (octal string to decimal)
print(int('ff', 16))         # 255 (hex string to decimal)
print(int('FF', 16))         # 255 (case-insensitive)

# Literals in Python code:
a = 0b11111111    # binary literal = 255
b = 0o377         # octal literal  = 255
c = 0xFF          # hex literal    = 255
print(a == b == c)  # True

# ---- Manual Conversion: Decimal → Binary ----
# Repeatedly divide by 2, collect remainders bottom-to-top.

def decimal_to_binary(n):
    """Convert decimal integer to binary string (manual method)."""
    if n == 0:
        return "0"
    bits = []
    while n > 0:
        bits.append(str(n % 2))    # remainder is the next bit (LSB first)
        n //= 2
    return ''.join(reversed(bits))  # reverse to get MSB first

print(decimal_to_binary(45))   # '101101'
# Step-by-step:
# 45 ÷ 2 = 22, remainder 1 → bit 0
# 22 ÷ 2 = 11, remainder 0 → bit 1
# 11 ÷ 2 = 5,  remainder 1 → bit 2
# 5  ÷ 2 = 2,  remainder 1 → bit 3
# 2  ÷ 2 = 1,  remainder 0 → bit 4
# 1  ÷ 2 = 0,  remainder 1 → bit 5  ← MSB
# Read bottom to top: 101101

# ---- Manual Conversion: Binary → Decimal ----
def binary_to_decimal(binary_str):
    """Convert binary string to decimal integer."""
    result = 0
    for i, bit in enumerate(reversed(binary_str)):
        result += int(bit) * (2 ** i)
    return result

print(binary_to_decimal("101101"))   # 45
```

### Level 2: Hexadecimal in Practice

```python
# Hexadecimal digit values:
# 0=0, 1=1, 2=2, ..., 9=9, A=10, B=11, C=12, D=13, E=14, F=15

# Why hex is useful: each hex digit = exactly 4 binary bits
# Binary:  1111  1010  1101  0011
# Hex:       F     A     D     3   = 0xFAD3

# Common hex values to memorize:
hex_reference = {
    0x00: "0000 0000",  # all zeros
    0x0F: "0000 1111",  # lower nibble
    0xF0: "1111 0000",  # upper nibble
    0xFF: "1111 1111",  # all ones = 255
    0x80: "1000 0000",  # high bit set = 128
    0x7F: "0111 1111",  # max positive signed byte = 127
}

# RGB color codes (web colors)
def hex_to_rgb(hex_color):
    """Convert hex color code to (R, G, B) tuple."""
    hex_color = hex_color.lstrip('#')    # remove # if present
    return tuple(int(hex_color[i:i+2], 16) for i in (0, 2, 4))

def rgb_to_hex(r, g, b):
    """Convert RGB values to hex color code."""
    return f"#{r:02X}{g:02X}{b:02X}"

print(hex_to_rgb("#FF5733"))    # (255, 87, 51)  — orange-red
print(rgb_to_hex(255, 87, 51)) # '#FF5733'

# Memory addresses are always shown in hex
# 0x7ffee4b3c8f0 = an address on the stack (macOS 64-bit)
address = 0x7ffee4b3c8f0
print(f"Address: {address}")           # 140732778111216 (decimal)
print(f"Hex: {hex(address)}")          # 0x7ffee4b3c8f0

# Hex dumps — reading raw binary data
def hex_dump(data: bytes, width: int = 16) -> str:
    """Format bytes as a hex dump with ASCII representation."""
    lines = []
    for offset in range(0, len(data), width):
        chunk = data[offset:offset + width]
        hex_part = ' '.join(f'{b:02X}' for b in chunk)
        ascii_part = ''.join(chr(b) if 32 <= b < 127 else '.' for b in chunk)
        lines.append(f"{offset:08X}  {hex_part:<{width*3}}  {ascii_part}")
    return '\n'.join(lines)

sample = b"Hello, World!\x00\x01\xFF\xFE"
print(hex_dump(sample))
# 00000000  48 65 6C 6C 6F 2C 20 57 6F 72 6C 64 21 00 01 FF  Hello, World!...
# 00000010  FE                                                  .
```

### Level 3: Binary Arithmetic and Two's Complement

```python
# Binary addition: same as decimal but carries when sum >= 2 (not 10)
#
#   0 0 1 0 1 1 0 1   (45)
# + 0 0 0 1 0 1 1 1   (23)
# ─────────────────
#   0 1 0 0 0 1 0 0   (68)
#
# Carry propagates: 1+1 = 10₂ (write 0, carry 1)

def add_binary(a_str, b_str):
    """Add two binary strings."""
    return bin(int(a_str, 2) + int(b_str, 2))[2:]

print(add_binary("00101101", "00010111"))   # '1000100' = 68

# Two's complement: how computers represent negative numbers
# For an n-bit number:
#   Positive numbers: 0 to 2^(n-1) - 1 (same as unsigned)
#   Negative numbers: flip all bits, add 1
#
# Why two's complement?
# - Addition of positive and negative works with the same circuit
# - Only one representation of zero (unlike sign-magnitude)

def to_twos_complement(n, bits=8):
    """Convert signed integer to its two's complement binary representation."""
    if n >= 0:
        return format(n, f'0{bits}b')
    else:
        # Method: 2^bits + n
        return format((1 << bits) + n, f'0{bits}b')

def from_twos_complement(binary_str):
    """Convert two's complement binary string to signed integer."""
    n = len(binary_str)
    value = int(binary_str, 2)
    # If the MSB is 1, the number is negative
    if binary_str[0] == '1':
        value -= (1 << n)
    return value

# 8-bit examples:
for n in [0, 1, 127, -1, -128]:
    bits = to_twos_complement(n, 8)
    back = from_twos_complement(bits)
    print(f"{n:5d} → {bits} → {back}")
#     0 → 00000000 → 0
#     1 → 00000001 → 1
#   127 → 01111111 → 127
#    -1 → 11111111 → -1   (all ones!)
#  -128 → 10000000 → -128

# Two's complement arithmetic: -1 + 1 = 0
# 11111111 (-1)
# 00000001 (+1)
# ─────────────
# 00000000 (0, with carry out discarded)

# Integer overflow example (8-bit)
# 127 + 1 = 128... but 128 can't be represented in 8 bits signed!
# 01111111 (127)
# 00000001 (+1)
# ─────────────
# 10000000 (-128 in two's complement) ← overflow!
```

### Level 4: Bit-Level Data Representation

```python
# Understanding how data types map to bits

# IEEE 754 single-precision float (32 bits)
# Layout: [1 sign bit][8 exponent bits][23 mantissa bits]
import struct

def float_to_bits(f):
    """Get the raw bits of a float."""
    packed = struct.pack('>f', f)   # big-endian 4-byte float
    bits = ''.join(format(b, '08b') for b in packed)
    return bits

def explain_float_bits(f):
    bits = float_to_bits(f)
    sign = bits[0]
    exponent = bits[1:9]
    mantissa = bits[9:]
    exp_value = int(exponent, 2) - 127   # biased by 127
    print(f"Float: {f}")
    print(f"Bits:     {bits}")
    print(f"Sign:     {sign} ({'negative' if sign == '1' else 'positive'})")
    print(f"Exponent: {exponent} = {int(exponent, 2)} - 127 = {exp_value}")
    print(f"Mantissa: {mantissa}")

explain_float_bits(1.0)
# Bits:     00111111100000000000000000000000
# Sign:     0 (positive)
# Exponent: 01111111 = 127 - 127 = 0  (so 2^0 = 1)
# Mantissa: 00000000000000000000000           (= 1.0 exactly)

# This is why floating-point arithmetic can have precision errors:
# 0.1 cannot be represented exactly in binary
print(format(0.1, '.20f'))   # 0.10000000000000000555...
print(0.1 + 0.2 == 0.3)     # False
print(abs(0.1 + 0.2 - 0.3) < 1e-9)  # True — use epsilon comparison

# Character encoding: ASCII and UTF-8
# ASCII: 7 bits, 128 characters
# 'A' = 65 = 0x41 = 0100 0001

for char in "Hello":
    code = ord(char)
    print(f"'{char}': decimal={code}, hex={hex(code)}, binary={bin(code)}")
# 'H': decimal=72, hex=0x48, binary=0b1001000

# UTF-8: variable-length encoding
text = "Hello, 世界"
utf8_bytes = text.encode('utf-8')
print(f"String: {text}")
print(f"UTF-8 bytes ({len(utf8_bytes)}): {utf8_bytes.hex()}")
# 'H'-'o' each = 1 byte, '世' and '界' each = 3 bytes
```

### Level 5: Practical Applications

```python
# Unix file permissions use octal
# chmod 755 → rwxr-xr-x
# 7 = 111₂ = rwx, 5 = 101₂ = r-x

def explain_permission(mode):
    """Explain a Unix permission mode in octal."""
    owner = (mode >> 6) & 0o7    # upper 3 bits
    group = (mode >> 3) & 0o7    # middle 3 bits
    other = mode & 0o7           # lower 3 bits

    def bits_to_str(bits):
        return ('r' if bits & 4 else '-') + \
               ('w' if bits & 2 else '-') + \
               ('x' if bits & 1 else '-')

    return f"{oct(mode)} = {bits_to_str(owner)}{bits_to_str(group)}{bits_to_str(other)}"

print(explain_permission(0o755))   # 0o755 = rwxr-xr-x
print(explain_permission(0o644))   # 0o644 = rw-r--r--
print(explain_permission(0o600))   # 0o600 = rw-------

# Network: IPv4 addresses and subnet masks
def ip_to_binary(ip):
    parts = ip.split('.')
    return '.'.join(format(int(p), '08b') for p in parts)

def subnet_info(ip, cidr):
    """Calculate network address, broadcast, and host range."""
    parts = list(map(int, ip.split('.')))
    ip_int = (parts[0] << 24) | (parts[1] << 16) | (parts[2] << 8) | parts[3]
    mask = (0xFFFFFFFF << (32 - cidr)) & 0xFFFFFFFF
    network = ip_int & mask
    broadcast = network | (~mask & 0xFFFFFFFF)

    def int_to_ip(n):
        return '.'.join(str((n >> i) & 0xFF) for i in [24, 16, 8, 0])

    print(f"IP:        {ip}")
    print(f"Mask:      {int_to_ip(mask)} (/{cidr})")
    print(f"Network:   {int_to_ip(network)}")
    print(f"Broadcast: {int_to_ip(broadcast)}")
    print(f"Hosts:     {int_to_ip(network+1)} to {int_to_ip(broadcast-1)}")
    print(f"Count:     {broadcast - network - 1}")

subnet_info("192.168.1.100", 24)
# IP:        192.168.1.100
# Mask:      255.255.255.0 (/24)
# Network:   192.168.1.0
# Broadcast: 192.168.1.255
# Hosts:     192.168.1.1 to 192.168.1.254
# Count:     254
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Confusing binary with "just 0s and 1s"**

```python
# Binary is not a display format — it's a positional number system.
# 11111111 in binary is 255 in decimal.
# The VALUE is 255. The REPRESENTATION depends on the base.

# WRONG thinking: "binary is harder than decimal"
# RIGHT thinking: it follows the exact same positional rules, just base 2

# Verify this by computing manually:
# 1×2^7 + 1×2^6 + ... + 1×2^0 = 128 + 64 + 32 + 16 + 8 + 4 + 2 + 1 = 255
print(sum(2**i for i in range(8)))   # 255
```

**Mistake 2: Forgetting hexadecimal digits go to F, not 9**

```python
# Common error: treating hex like decimal when doing mental math
# 0x19 is NOT 19. It's 1×16 + 9 = 25.
# 0x1A is 1×16 + 10 = 26.
# 0xFF is 15×16 + 15 = 255 (not 99).

print(0x19)   # 25
print(0x1A)   # 26
print(0xFF)   # 255
print(0x100)  # 256 (just past one byte)
```

**Mistake 3: Integer overflow from ignoring bit width**

```python
# In Python, integers are arbitrary precision — no overflow.
# In C, Java, Go: they have fixed width, and overflow wraps around.

# C example (mental model):
# int8_t x = 127;
# x++;           // x is now -128 (overflow!)

# Python doesn't have this problem, but when working with binary protocols,
# struct packing, or interfacing with C, you must think in bit widths.

import struct
# Pack a value as a signed 8-bit integer
try:
    struct.pack('b', 128)   # error: 128 doesn't fit in int8 (-128 to 127)
except struct.error as e:
    print(e)   # byte format requires -128 <= number <= 127

# Use unsigned: 'B' for uint8 (0 to 255)
print(struct.pack('B', 255))   # b'\xff'
```

**Mistake 4: Conflating bit order (endianness)**

```python
# Big-endian: most significant byte first (network byte order)
# Little-endian: least significant byte first (most Intel/AMD CPUs)

value = 0x01020304

big_endian    = struct.pack('>I', value)   # > = big-endian
little_endian = struct.pack('<I', value)   # < = little-endian

print(big_endian.hex())     # '01020304'  — MSB first
print(little_endian.hex())  # '04030201'  — LSB first

# This matters when reading binary files or network packets.
# TCP/IP uses big-endian ("network byte order").
# Most desktop CPUs use little-endian.
```

---

## 5. The "Why Does This Work" Layer

### Why Computers Use Binary, Not Decimal

Transistors have two stable states determined by voltage thresholds: below a threshold → 0, above → 1. Building a reliable circuit that distinguishes ten stable voltage levels (for decimal) would require far more precision and consume far more power. Two stable states are robust against noise and manufacturing variation.

This is why quantum computing is such a different paradigm — quantum bits (qubits) can represent superpositions, breaking the binary constraint entirely. But all classical computers, from microcontrollers to supercomputers, reduce everything to binary.

### Why Hexadecimal Is the Programmer's Shorthand

One hex digit represents exactly four bits (a "nibble"), so two hex digits represent exactly one byte (eight bits). This correspondence makes reading binary data human-tractable:

```
Binary:  1010 1111 0011 0110
                         ↕ (group into nibbles, read as hex)
Hex:       A    F    3    6   → 0xAF36

Without hex, you'd have to read "10101111 00110110" = 44854 (decimal)
That's much harder to decompose mentally.
```

Every byte in RAM, every network packet, every file on disk has a hex representation that programmers can scan and reason about directly.

### How Two's Complement Unifies Addition and Subtraction

Before two's complement, hardware needed separate circuits for addition and subtraction. Two's complement made subtraction identical to addition of the negative:

```
5 - 3 = 5 + (-3)

In 4-bit two's complement:
 5 = 0101
-3 = 1101  (flip 0011 → 1100, add 1 → 1101)

 0101
+1101
──────
10010  ← 5 bits, but we only keep 4: 0010 = 2 ✓

The carry out of the MSB is simply discarded.
```

This is why modern CPUs have an "add" instruction and no dedicated "subtract" instruction — you subtract by adding the two's complement. The hardware is simpler and the operation is universal.

---

## 6. Quick Reference

### Base Conversion Table (0–15)

| Decimal | Binary | Hex |
|---------|--------|-----|
| 0 | 0000 | 0 |
| 1 | 0001 | 1 |
| 2 | 0010 | 2 |
| 3 | 0011 | 3 |
| 4 | 0100 | 4 |
| 5 | 0101 | 5 |
| 6 | 0110 | 6 |
| 7 | 0111 | 7 |
| 8 | 1000 | 8 |
| 9 | 1001 | 9 |
| 10 | 1010 | A |
| 11 | 1011 | B |
| 12 | 1100 | C |
| 13 | 1101 | D |
| 14 | 1110 | E |
| 15 | 1111 | F |

### Powers of 2

| Power | Value | Bytes |
|-------|-------|-------|
| 2⁸ | 256 | 1 byte range |
| 2¹⁰ | 1,024 | 1 KB |
| 2¹⁶ | 65,536 | 2 bytes, max port |
| 2²⁰ | 1,048,576 | 1 MB |
| 2³² | 4,294,967,296 | 4 bytes, IPv4 addresses |
| 2⁶⁴ | 18,446,744,073,709,551,616 | 8 bytes, 64-bit integers |

### Python Conversion Functions

```python
# To binary/octal/hex strings
bin(n)            # '0b...'
oct(n)            # '0o...'
hex(n)            # '0x...'
format(n, '08b')  # zero-padded binary
format(n, '04X')  # zero-padded uppercase hex

# From string in any base
int('ff', 16)     # 255 — hex to decimal
int('101101', 2)  # 45  — binary to decimal
int('377', 8)     # 255 — octal to decimal

# Literals
0b1010    # binary = 10
0o12      # octal  = 10
0xA       # hex    = 10
```

### Common Hex Values

| Hex | Decimal | Meaning |
|-----|---------|---------|
| 0x00 | 0 | Null byte |
| 0x0A | 10 | Newline (\n) |
| 0x0D | 13 | Carriage return (\r) |
| 0x20 | 32 | Space |
| 0x41 | 65 | 'A' |
| 0x61 | 97 | 'a' |
| 0x7F | 127 | Max signed byte |
| 0x80 | 128 | Min negative signed byte (-128) |
| 0xFF | 255 | Max unsigned byte |
| 0xDEADBEEF | 3,735,928,559 | Classic debug marker |
| 0xCAFEBABE | 3,405,691,582 | Java class file magic |
