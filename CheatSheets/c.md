# C Complete Reference Guide
> From Zero to Expert — Everything You Need to Master C Programming

---

## Table of Contents

1. [Environment Setup & Compilation](#1-environment-setup--compilation)
2. [Program Structure & Basics](#2-program-structure--basics)
3. [Data Types & Variables](#3-data-types--variables)
4. [Operators](#4-operators)
5. [Control Flow](#5-control-flow)
6. [Functions](#6-functions)
7. [Arrays](#7-arrays)
8. [Pointers](#8-pointers)
9. [Strings](#9-strings)
10. [Structures, Unions & Enums](#10-structures-unions--enums)
11. [Dynamic Memory Management](#11-dynamic-memory-management)
12. [File I/O](#12-file-io)
13. [Preprocessor & Macros](#13-preprocessor--macros)
14. [Bitwise Operations & Bit Fields](#14-bitwise-operations--bit-fields)
15. [Advanced Pointers & Function Pointers](#15-advanced-pointers--function-pointers)
16. [Linked Lists & Data Structures](#16-linked-lists--data-structures)
17. [Error Handling & Debugging](#17-error-handling--debugging)
18. [C Standard Library Deep Dive](#18-c-standard-library-deep-dive)
19. [Advanced Topics](#19-advanced-topics)
20. [Best Practices, Security & Performance](#20-best-practices-security--performance)

---

## 1. Environment Setup & Compilation

### Installing GCC

**Linux (Debian/Ubuntu)**
```bash
sudo apt update
sudo apt install gcc gdb make
```

**Linux (Fedora/RHEL)**
```bash
sudo dnf install gcc gdb make
```

**macOS**
```bash
xcode-select --install       # Installs Clang (compatible with GCC commands)
brew install gcc              # Optional: true GCC via Homebrew
```

**Windows**
- Install [MinGW-w64](https://www.mingw-w64.org/) — provides `gcc` on Windows
- Or use WSL (Windows Subsystem for Linux) for a full Linux environment
- Or install [TDM-GCC](https://jmeubank.github.io/tdm-gcc/)

### Compiling Your First Program

```bash
gcc hello.c -o hello          # Compile to executable named "hello"
./hello                       # Run it

gcc hello.c                   # Compiles to default "a.out"
./a.out
```

### Essential Compiler Flags

```bash
gcc -Wall -Wextra -o app main.c        # Enable most warnings (use always)
gcc -std=c99 main.c -o app            # Use C99 standard
gcc -std=c11 main.c -o app            # Use C11 standard
gcc -std=c17 main.c -o app            # Use C17 (latest stable)
gcc -g main.c -o app                  # Add debug symbols (for GDB)
gcc -O0 main.c -o app                 # No optimization (default)
gcc -O2 main.c -o app                 # Standard optimization
gcc -O3 main.c -o app                 # Aggressive optimization
gcc main.c -o app -lm                 # Link math library
gcc main.c -o app -lpthread           # Link POSIX threads
gcc -fsanitize=address,undefined main.c -o app   # Runtime error detection
gcc -E main.c -o main.i               # Only run preprocessor
gcc -S main.c -o main.s               # Generate assembly
gcc -c main.c -o main.o               # Compile to object file (no linking)
```

### Compiling Multiple Files

```bash
# Compile each to object file, then link
gcc -c main.c -o main.o
gcc -c utils.c -o utils.o
gcc main.o utils.o -o app

# Or in one step
gcc main.c utils.c -o app
```

### Using Make (Build Automation)

```makefile
# Makefile
CC = gcc
CFLAGS = -Wall -Wextra -std=c11 -g

app: main.o utils.o
	$(CC) main.o utils.o -o app

main.o: main.c utils.h
	$(CC) $(CFLAGS) -c main.c

utils.o: utils.c utils.h
	$(CC) $(CFLAGS) -c utils.c

clean:
	rm -f *.o app
```

```bash
make          # Build
make clean    # Remove build artifacts
```

### GDB Debugger Basics

```bash
gcc -g main.c -o app      # Compile with debug info
gdb ./app                 # Start GDB

# Inside GDB:
(gdb) break main          # Set breakpoint at main
(gdb) break 25            # Set breakpoint at line 25
(gdb) run                 # Run program
(gdb) next                # Next line (step over)
(gdb) step                # Step into function
(gdb) print x             # Print variable x
(gdb) print *ptr          # Print value at pointer
(gdb) backtrace           # Show call stack
(gdb) continue            # Continue until next breakpoint
(gdb) quit                # Exit GDB
```

### C Standards Overview

| Standard | Year | Key Additions |
|----------|------|---------------|
| C89/ANSI C | 1989 | Original standard |
| C90 | 1990 | ISO version of C89 (same language) |
| C99 | 1999 | `//` comments, `bool`, `stdint.h`, VLAs, `inline` |
| C11 | 2011 | `_Generic`, threads, `_Static_assert`, anonymous structs |
| C17 | 2018 | Bug fixes only |
| C23 | 2024 | `bool`/`true`/`false` as keywords, `#embed`, more |

---

## 2. Program Structure & Basics

### Minimal Program

```c
#include <stdio.h>

int main(void) {
    printf("Hello, World!\n");
    return 0;
}
```

### What Each Part Means

```c
#include <stdio.h>    // Preprocessor: paste contents of stdio.h here

int main(void) {      // Entry point. int = return type, void = no parameters
    // All C programs start here
    return 0;         // Return 0 to OS = success. Non-zero = error.
}
```

### main() Variants

```c
// No arguments
int main(void) { return 0; }

// With command-line arguments
int main(int argc, char *argv[]) {
    // argc = argument count (includes program name)
    // argv = array of argument strings
    printf("Program: %s\n", argv[0]);
    for (int i = 1; i < argc; i++) {
        printf("Arg %d: %s\n", i, argv[i]);
    }
    return 0;
}

// Also valid (some systems)
int main(int argc, char *argv[], char *envp[]) { }
```

```bash
./app hello world     # argc=3, argv={"./app","hello","world"}
```

### Comments

```c
// Single-line comment (C99 and later)

/* Multi-line
   comment */

/* Nested comments are NOT allowed:
/* This would break! */   <-- This ends the comment early
*/

// Commonly used for documentation:
/**
 * @brief Adds two integers.
 * @param a First integer
 * @param b Second integer
 * @return Sum of a and b
 */
int add(int a, int b) {
    return a + b;
}
```

### Output with printf

```c
#include <stdio.h>

printf("Hello\n");                  // Newline with \n
printf("Tab\there\n");             // Tab with \t
printf("Quote: \"hi\"\n");         // Escaped quote
printf("Backslash: \\\n");         // Escaped backslash
printf("Value: %d\n", 42);         // Integer
printf("Float: %f\n", 3.14);       // Float (6 decimal places)
printf("Float: %.2f\n", 3.14159);  // Float with 2 decimal places
printf("Sci: %e\n", 12345.678);    // Scientific notation: 1.234568e+04
printf("String: %s\n", "hello");   // String
printf("Char: %c\n", 'A');         // Character
printf("Hex: %x\n", 255);          // Hexadecimal: ff
printf("Hex: %X\n", 255);          // Uppercase hex: FF
printf("Hex: %#x\n", 255);         // Hex with prefix: 0xff
printf("Oct: %o\n", 8);            // Octal: 10
printf("Ptr: %p\n", ptr);          // Pointer address
printf("Unsigned: %u\n", 42u);     // Unsigned int
printf("Long: %ld\n", 1000000L);   // Long
printf("Long long: %lld\n", 9LL);  // Long long
printf("%d + %d = %d\n", 3, 4, 7); // Multiple values

// Width and alignment
printf("%10d\n", 42);    // Right-aligned in 10 chars: "        42"
printf("%-10d|\n", 42);  // Left-aligned:               "42        |"
printf("%010d\n", 42);   // Zero-padded:                 "0000000042"
printf("%+d\n", 42);     // Always show sign:            "+42"
```

### Input with scanf

```c
int n;
float f;
char c;
char str[50];

scanf("%d", &n);          // Read integer (& is address-of operator!)
scanf("%f", &f);          // Read float
scanf("%c", &c);          // Read character
scanf("%s", str);         // Read word (no & needed for arrays)
scanf("%49s", str);       // Read word, max 49 chars (safe version)
scanf("%d %f", &n, &f);   // Read multiple values

// Return value: number of items successfully read
int result = scanf("%d", &n);
if (result != 1) {
    printf("Invalid input\n");
    // Clear the input buffer
    int ch;
    while ((ch = getchar()) != '\n' && ch != EOF);
}

// Read a full line
char line[100];
fgets(line, sizeof(line), stdin);
// Note: fgets keeps the '\n' — strip it:
line[strcspn(line, "\n")] = '\0';
```

### Escape Sequences

| Sequence | Meaning |
|----------|---------|
| `\n` | Newline |
| `\t` | Horizontal tab |
| `\r` | Carriage return |
| `\\` | Backslash |
| `\"` | Double quote |
| `\'` | Single quote |
| `\0` | Null character (string terminator) |
| `\a` | Alert/bell |
| `\b` | Backspace |
| `\f` | Form feed |
| `\v` | Vertical tab |
| `\xhh` | Hex value (e.g., `\x41` = 'A') |
| `\ooo` | Octal value (e.g., `\101` = 'A') |

---

## 3. Data Types & Variables

### Fundamental Types

| Type | Size | Format | Range |
|------|------|--------|-------|
| `char` | 1 byte | `%c` / `%d` | -128 to 127 (signed) |
| `unsigned char` | 1 byte | `%u` | 0 to 255 |
| `short` | 2 bytes | `%hd` | -32,768 to 32,767 |
| `unsigned short` | 2 bytes | `%hu` | 0 to 65,535 |
| `int` | 4 bytes | `%d` | ~±2.1 billion |
| `unsigned int` | 4 bytes | `%u` | 0 to ~4.3 billion |
| `long` | 4 or 8 bytes | `%ld` | Platform-dependent |
| `unsigned long` | 4 or 8 bytes | `%lu` | Platform-dependent |
| `long long` | 8 bytes | `%lld` | ~±9.2 × 10¹⁸ |
| `unsigned long long` | 8 bytes | `%llu` | 0 to ~1.8 × 10¹⁹ |
| `float` | 4 bytes | `%f` | ~7 significant digits |
| `double` | 8 bytes | `%lf` / `%f` | ~15 significant digits |
| `long double` | 10–16 bytes | `%Lf` | ~18–19 significant digits |

> **Note:** Sizes above are typical for 64-bit systems. Use `sizeof()` to confirm on your platform.

### Declaring and Initializing Variables

```c
int x;            // Declared — value is UNDEFINED (garbage!)
int y = 10;       // Declared and initialized
int a, b, c;      // Multiple declarations
int p = 1, q = 2; // Multiple with initialization

// Always initialize!
int count = 0;
double total = 0.0;
char letter = '\0';

// Checking sizes
printf("int: %zu bytes\n", sizeof(int));
printf("double: %zu bytes\n", sizeof(double));
printf("long long: %zu bytes\n", sizeof(long long));
// Note: use %zu for sizeof results (type is size_t)
```

### Constants

```c
// Macro constant (preprocessor — no type, no memory)
#define PI 3.14159265358979
#define MAX_SIZE 100
#define GREETING "Hello"

// const variable (has type, occupies memory, type-safe)
const int MAX = 100;
const double RATE = 0.05;
const char NEWLINE = '\n';

// const with pointers
const int *p = &x;       // Pointer to const int — can't modify *p
int * const p2 = &x;     // Const pointer — can't change where p2 points
const int * const p3 = &x;  // Both const

// Literal suffixes
42        // int
42L       // long
42LL      // long long
42U       // unsigned int
42UL      // unsigned long
3.14      // double
3.14f     // float
3.14L     // long double
0xFF      // Hexadecimal int (= 255)
0377      // Octal int (= 255)
0b1010    // Binary int (GCC extension, C23 standard)
```

### Fixed-Width Integer Types (C99, `<stdint.h>`)

```c
#include <stdint.h>

int8_t   a = 127;           // Exactly 8 bits signed
int16_t  b = 32767;         // Exactly 16 bits signed
int32_t  c = 2147483647;    // Exactly 32 bits signed
int64_t  d = 9000000000LL;  // Exactly 64 bits signed

uint8_t  e = 255;
uint16_t f = 65535;
uint32_t g = 4294967295U;
uint64_t h = 18446744073709551615ULL;

// Fast types (at least N bits, fastest available)
int_fast8_t   i;
int_fast32_t  j;
uint_fast64_t k;

// Minimum types (smallest that fits N bits)
int_least8_t  l;
int_least16_t m;

// Pointer-sized integer
intptr_t  ptr_int;   // Can hold a pointer value
uintptr_t uptr_int;

// Max-width integer
intmax_t  biggest;   // Largest signed integer type
uintmax_t ubiggest;

// Limits
#include <stdint.h>
INT8_MAX;     // 127
INT8_MIN;     // -128
UINT8_MAX;    // 255
INT32_MAX;    // 2147483647
INT64_MIN;    // -9223372036854775808
```

### Boolean Type (C99)

```c
#include <stdbool.h>

bool is_valid = true;
bool is_done = false;

if (is_valid) {
    printf("Valid!\n");
}

// In C, any non-zero integer is "true", zero is "false"
int flag = 1;    // true
int off  = 0;    // false

// Before C99 (no stdbool.h):
typedef int bool;
#define true  1
#define false 0
```

### Type Conversion (Casting)

```c
int a = 5, b = 2;

// Integer division — result is 2, not 2.5!
double result = a / b;       // Still 2.0 (division happens as int first)

// Correct — cast before division
double result2 = (double)a / b;    // 2.5
double result3 = a / (double)b;    // 2.5

// Casting
int x = (int)3.99;     // Truncates to 3 (not rounds!)
float f = (float)7;    // 7.0
char c = (char)65;     // 'A'

// Implicit conversions (happen automatically, be aware!)
int i = 3.9;           // Implicit: i = 3 (truncated, may warn)
float g = 1000000 * 1000000;  // Overflow! Use long or cast

// Integer promotion
char ch = 'A';         // ch promoted to int in expressions
short s = 10;          // s promoted to int in expressions
unsigned + signed      // signed converts to unsigned (can cause bugs!)
```

### Variable Scope and Lifetime

```c
int global = 100;       // Global scope — accessible everywhere, lifetime = program

void func(void) {
    int local = 5;      // Local scope — only inside func, lifetime = function call
    
    static int count = 0;  // Static local — persists between calls, initialized once
    count++;
    printf("Called %d times\n", count);
    
    {
        int block_var = 10;  // Block scope — only inside this {}
    }
    // block_var is not accessible here
}

// extern — declare variable defined elsewhere
extern int global;      // Tells compiler: this is defined in another file

// Storage classes
auto int x = 5;         // auto is default for locals (rarely written explicitly)
register int y = 5;     // Hint to store in CPU register (mostly ignored today)
```

---

## 4. Operators

### Arithmetic

```c
int a = 17, b = 5;

a + b    // 22  — addition
a - b    // 12  — subtraction
a * b    // 85  — multiplication
a / b    // 3   — integer division (truncates toward zero)
a % b    // 2   — modulo (remainder)

// Important: integer division truncates!
7 / 2    // 3, not 3.5
-7 / 2   // -3 (C99: truncates toward zero)
-7 % 2   // -1 (sign of result matches dividend in C99)

// Increment and decrement
int x = 5;
x++;        // Post-increment: uses x (5), then x becomes 6
++x;        // Pre-increment: x becomes 6, then uses x (6)
x--;        // Post-decrement
--x;        // Pre-decrement

int y = x++;  // y = 5, x = 6 (post: y gets old value)
int z = ++x;  // z = 7, x = 7 (pre: z gets new value)

// Floating-point arithmetic
5.0 / 2.0   // 2.5
5.0 / 2     // 2.5 (int promotes to double)
5 / 2.0     // 2.5
```

### Assignment

```c
x = 5;
x += 5;    // x = x + 5
x -= 3;    // x = x - 3
x *= 2;    // x = x * 2
x /= 4;    // x = x / 4
x %= 3;    // x = x % 3
x &= 0xFF; // x = x & 0xFF
x |= 0x01; // x = x | 0x01
x ^= 0x10; // x = x ^ 0x10
x <<= 2;   // x = x << 2
x >>= 1;   // x = x >> 1
```

### Comparison & Logical

```c
// Comparison (all return 0 or 1)
a == b   // Equal
a != b   // Not equal
a >  b   // Greater than
a <  b   // Less than
a >= b   // Greater or equal
a <= b   // Less or equal

// Logical (short-circuit evaluation!)
a && b   // AND — if a is false, b is NOT evaluated
a || b   // OR  — if a is true, b is NOT evaluated
!a       // NOT

// Short-circuit example:
if (ptr != NULL && *ptr > 0) { }  // Safe: *ptr only evaluated if ptr != NULL
if (x == 0 || expensive_func()) { }  // expensive_func only called if x != 0

// Common mistake: using = instead of ==
if (x = 5) { }   // ASSIGNS 5 to x, always true! (should be ==)
if (5 == x) { }  // Yoda condition: prevents accidental assignment
```

### Bitwise Operators

```c
unsigned char a = 0b11001010;   // 202
unsigned char b = 0b10110101;   // 181

a & b    // AND:  0b10000000 = 128  (both bits must be 1)
a | b    // OR:   0b11111111 = 255  (either bit is 1)
a ^ b    // XOR:  0b01111111 = 127  (bits differ)
~a       // NOT:  0b00110101 = 53   (flip all bits)
a << 1   // Left shift:  shift bits left, fill 0s on right (multiply by 2)
a >> 1   // Right shift: shift bits right (divide by 2)

// Practical uses:
// Set bit n:
x |= (1 << n);

// Clear bit n:
x &= ~(1 << n);

// Toggle bit n:
x ^= (1 << n);

// Check bit n:
if (x & (1 << n)) { /* bit n is set */ }

// Get lower nibble (lower 4 bits):
x & 0x0F;

// Extract bytes:
uint32_t val = 0xDEADBEEF;
uint8_t byte0 = val & 0xFF;           // 0xEF
uint8_t byte1 = (val >> 8) & 0xFF;   // 0xBE
uint8_t byte2 = (val >> 16) & 0xFF;  // 0xAD
uint8_t byte3 = (val >> 24) & 0xFF;  // 0xDE
```

### Ternary Operator

```c
// condition ? value_if_true : value_if_false
int max = (a > b) ? a : b;
int abs_val = (x >= 0) ? x : -x;
printf("%s\n", (score >= 60) ? "Pass" : "Fail");

// Nested (avoid — hard to read)
int grade = (score >= 90) ? 'A' :
            (score >= 80) ? 'B' :
            (score >= 70) ? 'C' : 'F';
```

### sizeof Operator

```c
sizeof(int)            // 4
sizeof(double)         // 8
sizeof(char)           // 1
sizeof(long long)      // 8

int arr[10];
sizeof(arr)            // 40 (total bytes: 4 * 10)
sizeof(arr[0])         // 4
sizeof(arr)/sizeof(arr[0])  // 10 — number of elements

// sizeof returns type size_t (unsigned)
printf("%zu\n", sizeof(int));   // Use %zu for size_t
```

### Operator Precedence (Highest to Lowest)

| Level | Operators | Associativity |
|-------|-----------|---------------|
| 1 | `()` `[]` `->` `.` | Left to right |
| 2 | `!` `~` `++` `--` `+` `-` `*` `&` `sizeof` (unary) | Right to left |
| 3 | `*` `/` `%` | Left to right |
| 4 | `+` `-` | Left to right |
| 5 | `<<` `>>` | Left to right |
| 6 | `<` `<=` `>` `>=` | Left to right |
| 7 | `==` `!=` | Left to right |
| 8 | `&` | Left to right |
| 9 | `^` | Left to right |
| 10 | `\|` | Left to right |
| 11 | `&&` | Left to right |
| 12 | `\|\|` | Left to right |
| 13 | `?:` | Right to left |
| 14 | `=` `+=` `-=` etc. | Right to left |
| 15 | `,` | Left to right |

> **Tip:** When in doubt, use parentheses. Don't rely on precedence for tricky cases.

---

## 5. Control Flow

### if / else if / else

```c
int x = 42;

if (x > 100) {
    printf("Large\n");
} else if (x > 50) {
    printf("Medium\n");
} else if (x > 0) {
    printf("Small\n");
} else {
    printf("Negative or zero\n");
}

// Single-statement body (braces optional — but always use them!)
if (x > 0) printf("positive\n");  // Works but risky

// The "dangling else" problem — always use braces to avoid:
if (a)
    if (b)
        printf("a and b\n");
else     // This else belongs to the INNER if, not outer!
    printf("not b\n");  // Misleading indentation

// Correct with braces:
if (a) {
    if (b) {
        printf("a and b\n");
    }
} else {
    printf("not a\n");
}
```

### switch

```c
char grade = 'B';

switch (grade) {
    case 'A':
        printf("Excellent\n");
        break;              // MUST break, or falls through to next case
    case 'B':
    case 'C':               // Fall-through: both B and C handled here
        printf("Good\n");
        break;
    case 'D':
        printf("Passing\n");
        break;
    default:
        printf("Invalid grade\n");
        break;              // Not required for last case, but good practice
}

// Intentional fall-through (document it!)
switch (x) {
    case 1:
        do_something();
        /* FALLTHROUGH */   // Comment to signal intentional fall-through
    case 2:
        do_more();
        break;
}

// switch only works with integer types (int, char, enum)
// Cannot use float, string, or struct with switch
```

### Loops

```c
// for loop
for (int i = 0; i < 10; i++) {
    printf("%d ", i);
}

// All parts of for are optional
for (;;) {           // Infinite loop
    if (done) break;
}

// Multiple variables in for (C99)
for (int i = 0, j = 10; i < j; i++, j--) {
    printf("%d %d\n", i, j);
}

// while loop
int i = 0;
while (i < 10) {
    printf("%d ", i);
    i++;
}

// do-while (executes at least once)
int choice;
do {
    printf("Enter 1-5: ");
    scanf("%d", &choice);
} while (choice < 1 || choice > 5);

// Nested loops
for (int row = 0; row < 3; row++) {
    for (int col = 0; col < 3; col++) {
        printf("%d ", row * 3 + col);
    }
    printf("\n");
}
```

### break, continue, goto

```c
// break — exit the innermost loop or switch
for (int i = 0; i < 100; i++) {
    if (i == 10) break;  // Stop when i reaches 10
}

// continue — skip to next iteration
for (int i = 0; i < 10; i++) {
    if (i % 2 == 0) continue;   // Skip even numbers
    printf("%d ", i);            // Prints: 1 3 5 7 9
}

// goto — jump to a label (use sparingly — mainly for error cleanup)
int result = allocate_resources();
if (result < 0) goto cleanup;

result = do_work();
if (result < 0) goto cleanup;

result = 0;  // Success

cleanup:
    free_resources();
    return result;
```

---

## 6. Functions

### Basics

```c
// Function prototype (declaration) — needed before use
int add(int a, int b);
void greet(void);

// Function definition
int add(int a, int b) {
    return a + b;
}

void greet(void) {
    printf("Hello!\n");
    // No return needed for void, but allowed: return;
}

// Call
int sum = add(3, 4);     // sum = 7
greet();
```

### Pass by Value vs Pass by Pointer

```c
// Pass by VALUE — function gets a copy, original unchanged
void double_val(int x) {
    x *= 2;   // Only modifies local copy
}

int n = 5;
double_val(n);
printf("%d\n", n);   // Still 5!

// Pass by POINTER — function receives address, can modify original
void double_ptr(int *x) {
    *x *= 2;   // Dereferences pointer to modify original
}

double_ptr(&n);
printf("%d\n", n);   // Now 10!

// Swap example (only works with pointers)
void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

int x = 3, y = 7;
swap(&x, &y);
printf("%d %d\n", x, y);   // 7 3
```

### Returning Multiple Values (via Pointers)

```c
// C functions return only one value — use pointer params for multiple outputs
void min_max(int arr[], int size, int *min, int *max) {
    *min = *max = arr[0];
    for (int i = 1; i < size; i++) {
        if (arr[i] < *min) *min = arr[i];
        if (arr[i] > *max) *max = arr[i];
    }
}

int arr[] = {5, 2, 8, 1, 9};
int mn, mx;
min_max(arr, 5, &mn, &mx);
printf("Min: %d, Max: %d\n", mn, mx);  // Min: 1, Max: 9

// Or return a struct
typedef struct { int min; int max; } MinMax;

MinMax find_min_max(int arr[], int size) {
    MinMax result = { arr[0], arr[0] };
    for (int i = 1; i < size; i++) {
        if (arr[i] < result.min) result.min = arr[i];
        if (arr[i] > result.max) result.max = arr[i];
    }
    return result;
}
```

### Variable Arguments (Variadic Functions)

```c
#include <stdarg.h>

// At least one fixed parameter required
double average(int count, ...) {
    va_list args;
    va_start(args, count);   // Initialize args after last fixed param

    double sum = 0;
    for (int i = 0; i < count; i++) {
        sum += va_arg(args, int);  // Retrieve next argument (specify type)
    }

    va_end(args);    // Clean up
    return sum / count;
}

printf("%.2f\n", average(3, 10, 20, 30));  // 20.00
printf("%.2f\n", average(5, 1, 2, 3, 4, 5));  // 3.00
```

### Inline Functions (C99)

```c
// Suggests compiler to expand inline (avoids function call overhead)
static inline int square(int x) {
    return x * x;
}

// Use static inline for functions in header files
// to avoid multiple-definition errors
```

### Recursion

```c
// Factorial (recursive)
long long factorial(int n) {
    if (n < 0) return -1;   // Error
    if (n == 0 || n == 1) return 1;
    return n * factorial(n - 1);
}

// Fibonacci (naive — exponential time)
int fib_slow(int n) {
    if (n <= 1) return n;
    return fib_slow(n - 1) + fib_slow(n - 2);
}

// Fibonacci (with memoization — linear time)
#define MEMO_SIZE 100
long long memo[MEMO_SIZE];

void init_memo(void) {
    for (int i = 0; i < MEMO_SIZE; i++) memo[i] = -1;
}

long long fib(int n) {
    if (n <= 1) return n;
    if (memo[n] != -1) return memo[n];
    return memo[n] = fib(n - 1) + fib(n - 2);
}

// Binary search (recursive)
int bsearch_r(int arr[], int low, int high, int target) {
    if (low > high) return -1;
    int mid = low + (high - low) / 2;   // Avoids overflow vs (low+high)/2
    if (arr[mid] == target) return mid;
    if (arr[mid] < target) return bsearch_r(arr, mid + 1, high, target);
    return bsearch_r(arr, low, mid - 1, target);
}

// Power (fast exponentiation — O(log n))
long long power(long long base, int exp) {
    if (exp == 0) return 1;
    if (exp % 2 == 0) {
        long long half = power(base, exp / 2);
        return half * half;
    }
    return base * power(base, exp - 1);
}
```

### Math Functions (`<math.h>`)

```c
#include <math.h>
// Link with -lm: gcc prog.c -o prog -lm

sqrt(16.0)        // 4.0  — square root
cbrt(27.0)        // 3.0  — cube root
pow(2.0, 10.0)    // 1024.0 — power
fabs(-3.7)        // 3.7  — absolute value (float)
abs(-5)           // 5    — absolute value (int, from <stdlib.h>)
ceil(3.1)         // 4.0  — round up
floor(3.9)        // 3.0  — round down
round(3.5)        // 4.0  — round to nearest
trunc(3.9)        // 3.0  — truncate (toward zero)
fmod(10.5, 3.0)   // 1.5  — float modulo
log(M_E)          // 1.0  — natural log
log2(8.0)         // 3.0  — log base 2
log10(1000.0)     // 3.0  — log base 10
exp(1.0)          // e (≈ 2.718)
sin(M_PI / 2)     // 1.0
cos(0.0)          // 1.0
tan(M_PI / 4)     // ≈ 1.0
asin(1.0)         // M_PI / 2
atan2(y, x)       // Angle from origin to (x,y) — better than atan

// Constants (may need _GNU_SOURCE or _USE_MATH_DEFINES)
M_PI     // π ≈ 3.14159265358979
M_E      // e ≈ 2.71828182845905
M_SQRT2  // √2 ≈ 1.41421356237310
```

---

## 7. Arrays

### One-Dimensional Arrays

```c
// Declaration
int arr[5];                          // 5 ints, values UNDEFINED
int nums[5] = {1, 2, 3, 4, 5};     // Initialized
int vals[] = {10, 20, 30};          // Size inferred: 3
int zeros[10] = {0};                // First = 0, rest auto-initialized to 0
int zeros2[10] = {};                // All zeros (C99)

// Access (0-indexed)
printf("%d\n", nums[0]);   // 1
printf("%d\n", nums[4]);   // 5
nums[2] = 99;

// Common error — no bounds checking in C!
nums[5] = 1;    // UNDEFINED BEHAVIOR — writes past end of array!

// Size of array
int size = sizeof(nums) / sizeof(nums[0]);   // 5
// This ONLY works for true arrays, not pointer-to-array!

// Iterating
for (int i = 0; i < 5; i++) {
    printf("%d ", nums[i]);
}
```

### Multi-Dimensional Arrays

```c
// 2D array — stored in row-major order (rows are contiguous)
int matrix[3][4];                      // 3 rows, 4 columns
int grid[2][3] = {{1, 2, 3},
                  {4, 5, 6}};

printf("%d\n", grid[0][1]);   // 2 (row 0, col 1)
printf("%d\n", grid[1][2]);   // 6 (row 1, col 2)

// Traversal
for (int i = 0; i < 2; i++) {
    for (int j = 0; j < 3; j++) {
        printf("%d ", grid[i][j]);
    }
    printf("\n");
}

// 3D array
int cube[2][3][4];
cube[0][1][2] = 42;

// Passing 2D array to function — must specify all dimensions except first
void print_matrix(int rows, int cols, int mat[][4]) { }
// Or use pointer to array:
void print_matrix2(int rows, int cols, int (*mat)[4]) { }
```

### Arrays and Pointers (Key Relationship)

```c
int arr[] = {10, 20, 30, 40, 50};

// Array name decays to pointer to first element
int *p = arr;           // Same as: int *p = &arr[0]
printf("%d\n", *p);     // 10
printf("%d\n", p[2]);   // 30 — pointer subscript works same as array

// Pointer arithmetic
printf("%d\n", *(p + 1));   // 20
printf("%d\n", *(p + 4));   // 50

// arr[i] is EXACTLY equivalent to *(arr + i)
// i[arr] is also valid! (rare/confusing — don't use)

// BUT: arr is not a pointer variable — you cannot do arr++
// p++ is fine; arr++ is a compile error

// Important difference:
int arr2[5];
int *ptr = arr2;

sizeof(arr2)   // 20 (size of entire array)
sizeof(ptr)    // 8  (size of pointer — just an address!)
```

### Passing Arrays to Functions

```c
// Arrays always passed as pointer — function cannot know size!
void print_array(int *arr, int size) {
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
}

// These three declarations are equivalent:
void func(int arr[], int n) { }
void func(int *arr, int n) { }
void func(int arr[10], int n) { }  // The 10 is ignored!

// Always pass size separately
int data[] = {1, 2, 3, 4, 5};
print_array(data, 5);

// To prevent modification, use const:
void print_readonly(const int *arr, int size) { }
```

### Variable Length Arrays — VLAs (C99)

```c
// Size determined at runtime (allocated on stack)
int n;
scanf("%d", &n);
int vla[n];              // VLA — size known only at runtime

for (int i = 0; i < n; i++) vla[i] = i * i;

// VLA limitations:
// - Cannot be initialized at declaration
// - Cannot be global or static
// - Large VLAs can cause stack overflow — be careful!
// - Optional in C11 (use dynamic allocation for reliability)
```

### Common Array Algorithms

```c
// Linear search
int linear_search(int arr[], int n, int target) {
    for (int i = 0; i < n; i++) {
        if (arr[i] == target) return i;
    }
    return -1;
}

// Binary search (array must be sorted!)
int binary_search(int arr[], int n, int target) {
    int low = 0, high = n - 1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) low = mid + 1;
        else high = mid - 1;
    }
    return -1;
}

// Bubble sort
void bubble_sort(int arr[], int n) {
    for (int i = 0; i < n - 1; i++) {
        int swapped = 0;
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j+1]) {
                int tmp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = tmp;
                swapped = 1;
            }
        }
        if (!swapped) break;  // Already sorted
    }
}

// Using qsort from <stdlib.h>
int compare_ints(const void *a, const void *b) {
    int x = *(const int *)a;
    int y = *(const int *)b;
    return (x > y) - (x < y);  // Returns -1, 0, or 1 safely
    // Don't use: return x - y; (can overflow!)
}

int arr[] = {5, 2, 8, 1, 9, 3};
qsort(arr, 6, sizeof(int), compare_ints);

// Reverse an array
void reverse(int arr[], int n) {
    int left = 0, right = n - 1;
    while (left < right) {
        int tmp = arr[left];
        arr[left++] = arr[right];
        arr[right--] = tmp;
    }
}
```

---

## 8. Pointers

### Pointer Fundamentals

```c
int x = 42;

int *p = &x;       // p holds the address of x (& = address-of operator)
printf("%d\n", *p);   // Dereference: prints 42
printf("%p\n", (void *)p);  // Print address (cast to void* for %p)
*p = 100;           // Modify x through pointer: x is now 100

// Pointer types must match
double y = 3.14;
double *dp = &y;

// void pointer — generic pointer, no type
void *vp = &x;       // Any pointer type can be assigned to void*
int *ip = (int *)vp; // Must cast to use

// Pointer size (always same regardless of pointed-to type)
printf("%zu\n", sizeof(int *));     // 8 on 64-bit
printf("%zu\n", sizeof(double *));  // 8 on 64-bit
printf("%zu\n", sizeof(void *));    // 8 on 64-bit
```

### NULL Pointer

```c
int *p = NULL;     // Points to nothing (address 0)

// ALWAYS check before dereferencing!
if (p != NULL) {
    printf("%d\n", *p);  // Safe
}

// Dereferencing NULL = segmentation fault (crash)
*p = 5;   // CRASH! Undefined behavior

// NULL is defined in <stddef.h>, <stdio.h>, <stdlib.h>, etc.
// In C, NULL is typically ((void *)0)
```

### Pointer Arithmetic

```c
int arr[] = {10, 20, 30, 40, 50};
int *p = arr;

p + 1    // Points to arr[1] (advances by sizeof(int) = 4 bytes)
p + 4    // Points to arr[4]
*(p + 2) // 30 — same as arr[2]
p++      // Move to next element

// Pointer difference
int *start = &arr[0];
int *end   = &arr[4];
ptrdiff_t diff = end - start;  // 4 (number of elements, not bytes)
// Use ptrdiff_t (from <stddef.h>) for pointer differences

// Valid pointer arithmetic:
// pointer ± integer  → pointer
// pointer - pointer  → ptrdiff_t (only within same array)

// Invalid:
// pointer + pointer  (not defined)
// pointer * integer  (not defined)
```

### Pointers to Pointers

```c
int x = 42;
int *p = &x;
int **pp = &p;    // Pointer to pointer to int

printf("%d\n", x);    // 42
printf("%d\n", *p);   // 42
printf("%d\n", **pp); // 42

**pp = 100;  // Changes x to 100

// Common use: modify a pointer in a function
void allocate(int **ptr, int size) {
    *ptr = malloc(size * sizeof(int));
}

int *arr = NULL;
allocate(&arr, 10);
free(arr);

// 2D dynamic arrays use int **
int **matrix = malloc(rows * sizeof(int *));
for (int i = 0; i < rows; i++)
    matrix[i] = malloc(cols * sizeof(int));
```

### const and Pointers

```c
int x = 10;
int y = 20;

// Read the type right-to-left:
const int *p1 = &x;     // p1 is a pointer to const int
                         // — can change where p1 points, cannot modify *p1
*p1 = 5;    // ERROR — cannot modify *p1
p1 = &y;    // OK — can repoint

int * const p2 = &x;    // p2 is a const pointer to int
                         // — cannot change where p2 points, can modify *p2
*p2 = 5;    // OK — can modify *p2
p2 = &y;    // ERROR — cannot repoint

const int * const p3 = &x;  // const pointer to const int
                              // — cannot do either
*p3 = 5;    // ERROR
p3 = &y;    // ERROR

// Pass const pointer to prevent function from modifying data:
void print_array(const int *arr, int n) {
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);
    // arr[i] = 0;  // Compile error — protected!
}
```

### Pointer Pitfalls

```c
// 1. Uninitialized pointer (wild pointer)
int *p;           // p has garbage value — pointing to random memory
*p = 5;           // UNDEFINED BEHAVIOR — may crash

// 2. Dangling pointer (use-after-free)
int *p = malloc(sizeof(int));
*p = 42;
free(p);
printf("%d\n", *p);  // UNDEFINED BEHAVIOR — p still holds old address

// Fix: set to NULL after free
free(p);
p = NULL;

// 3. Double free
free(p);
free(p);   // UNDEFINED BEHAVIOR / crash

// Fix: set NULL, check before freeing
if (p != NULL) { free(p); p = NULL; }

// 4. Returning pointer to local variable
int *bad_func(void) {
    int local = 42;
    return &local;   // UNDEFINED BEHAVIOR — local destroyed after return!
}

// Fix: use malloc, static, or pass pointer in
int *good_func(void) {
    int *p = malloc(sizeof(int));
    *p = 42;
    return p;   // Caller must free!
}

// 5. Buffer overrun
int arr[5];
arr[5] = 1;   // UNDEFINED BEHAVIOR — past end of array
```

---

## 9. Strings

### String Fundamentals

```c
// In C, a string is an array of chars terminated by '\0' (null byte)
char str[] = "Hello";   // {'H','e','l','l','o','\0'} — size 6
char str2[10] = "Hi";   // {'H','i','\0', ...rest undefined}
char *str3 = "World";   // String literal — stored in read-only memory

// NEVER modify a string literal!
str3[0] = 'w';   // UNDEFINED BEHAVIOR — may crash

// Use array for modifiable strings:
char modifiable[] = "World";
modifiable[0] = 'w';   // OK — "world"

// Length
#include <string.h>
strlen("Hello")   // 5 — does NOT count the '\0'
sizeof("Hello")   // 6 — counts the '\0'
```

### String I/O

```c
char name[50];

// scanf — reads until whitespace (no spaces in input)
scanf("%49s", name);    // Limit to 49 chars (safe)

// fgets — reads entire line including spaces (safest)
fgets(name, sizeof(name), stdin);
// WARNING: fgets keeps the '\n' character!
name[strcspn(name, "\n")] = '\0';   // Strip trailing newline

// gets() — NEVER USE! No bounds checking, removed in C11
// gets(name);  // Dangerous!

// Output
printf("%s\n", name);
puts(name);       // Adds newline automatically

// sprintf — write to string (like printf but to buffer)
char buf[100];
sprintf(buf, "Name: %s, Age: %d", "Alice", 30);
snprintf(buf, sizeof(buf), "Safe: %s", name);  // Safer: limits output length
```

### String Library (`<string.h>`)

```c
#include <string.h>

char s1[50] = "Hello";
char s2[] = "World";

// Length
strlen(s1)                  // 5

// Copy
strcpy(s1, "New");          // Unsafe — no bounds checking
strncpy(s1, "New", 49);     // Safer — limits copy length
s1[49] = '\0';               // strncpy may not null-terminate!

// Concatenate
strcat(s1, " World");        // Unsafe
strncat(s1, " World", 44);  // Safer — limits appended length

// Compare (returns 0 if equal, <0 or >0 if not)
strcmp("abc", "abc")    // 0
strcmp("abc", "abd")    // negative (c < d)
strcmp("abd", "abc")    // positive
strncmp("abc", "abZ", 2)  // 0 — only compare first 2 chars
strcasecmp("Hello", "hello")  // 0 — case-insensitive (POSIX)

// Search
strchr(s1, 'l')           // Pointer to first 'l', or NULL
strrchr(s1, 'l')          // Pointer to LAST 'l', or NULL
strstr(s1, "ell")         // Pointer to first occurrence of "ell", or NULL
strpbrk("hello", "aeiou") // Pointer to first vowel in "hello"
strspn("hello123", "abcdefghijklmnopqrstuvwxyz")  // 5 (length of leading alpha)
strcspn("hello!world", "!?.")  // 5 (length before any punctuation)

// Tokenize (splits string by delimiters — MODIFIES the string!)
char sentence[] = "one two three";
char *token = strtok(sentence, " ");
while (token != NULL) {
    printf("%s\n", token);
    token = strtok(NULL, " ");  // Continue from where we left off
}

// Memory operations
memset(s1, 0, sizeof(s1));    // Fill with zeros (clear string)
memset(s1, 'x', 5);           // Fill first 5 bytes with 'x'
memcpy(s1, s2, strlen(s2)+1); // Copy bytes (doesn't handle overlap)
memmove(s1, s1+2, 5);         // Like memcpy but handles overlapping regions
memcmp(s1, s2, 3)             // Compare first 3 bytes

// Conversion (from <stdlib.h>)
atoi("42")           // String to int (no error handling)
atof("3.14")         // String to double
atol("100000")       // String to long
strtol("42", NULL, 10)   // Safer — detects errors, supports bases
strtod("3.14", NULL)     // String to double with error handling
```

### Character Functions (`<ctype.h>`)

```c
#include <ctype.h>

// Testing (return non-zero if true)
isalpha('a')    // Alphabetic: a-z, A-Z
isdigit('5')    // Digit: 0-9
isalnum('a')    // Alphanumeric
isspace(' ')    // Whitespace: space, tab, newline, etc.
islower('a')    // Lowercase letter
isupper('A')    // Uppercase letter
ispunct('.')    // Punctuation
isprint('a')    // Printable character
iscntrl('\n')   // Control character
isxdigit('F')   // Hex digit: 0-9, a-f, A-F

// Conversion
tolower('A')    // 'a'
toupper('a')    // 'A'

// Example: check if string is all digits
int is_number(const char *s) {
    if (!s || !*s) return 0;       // Empty string
    while (*s) {
        if (!isdigit(*s++)) return 0;
    }
    return 1;
}

// Count vowels
int count_vowels(const char *s) {
    int count = 0;
    while (*s) {
        char c = tolower(*s++);
        if (c=='a'||c=='e'||c=='i'||c=='o'||c=='u') count++;
    }
    return count;
}
```

### String Manipulation Examples

```c
// Reverse a string in-place
void reverse_string(char *s) {
    int left = 0, right = strlen(s) - 1;
    while (left < right) {
        char tmp = s[left];
        s[left++] = s[right];
        s[right--] = tmp;
    }
}

// Check if palindrome
int is_palindrome(const char *s) {
    int left = 0, right = strlen(s) - 1;
    while (left < right) {
        if (tolower(s[left]) != tolower(s[right])) return 0;
        left++; right--;
    }
    return 1;
}

// Convert int to string (without sprintf)
void int_to_str(int n, char *buf) {
    if (n == 0) { buf[0]='0'; buf[1]='\0'; return; }
    int i = 0;
    int neg = n < 0;
    if (neg) n = -n;
    while (n) { buf[i++] = '0' + (n % 10); n /= 10; }
    if (neg) buf[i++] = '-';
    buf[i] = '\0';
    reverse_string(buf);
}

// Count word occurrences
int count_word(const char *text, const char *word) {
    int count = 0;
    const char *pos = text;
    while ((pos = strstr(pos, word)) != NULL) {
        count++;
        pos += strlen(word);
    }
    return count;
}
```

---

## 10. Structures, Unions & Enums

### Structures

```c
// Define a struct type
struct Person {
    char name[50];
    int age;
    double salary;
};

// Declare variables
struct Person p1;                              // Uninitialized
struct Person p2 = {"Alice", 30, 75000.0};    // Initialized
struct Person p3 = {.name = "Bob", .age = 25}; // Designated initializers (C99)

// Access members
strcpy(p1.name, "Charlie");
p1.age = 35;
p1.salary = 90000.0;

printf("Name: %s, Age: %d\n", p2.name, p2.age);

// typedef to avoid writing "struct"
typedef struct {
    float x;
    float y;
} Point;

Point origin = {0.0, 0.0};
Point p = {3.0, 4.0};

// Nested structs
typedef struct {
    int day;
    int month;
    int year;
} Date;

typedef struct {
    char name[50];
    Date birth;
    Date hire;
} Employee;

Employee emp = {"Dave", {15, 6, 1990}, {1, 9, 2020}};
printf("Birth year: %d\n", emp.birth.year);
```

### Structures and Pointers

```c
typedef struct {
    char name[50];
    int age;
} Person;

Person p = {"Eve", 28};
Person *ptr = &p;

// Two ways to access through pointer:
printf("%s\n", (*ptr).name);   // Dereference then member access
printf("%s\n", ptr->name);     // Arrow operator (preferred, identical result)

ptr->age = 29;

// Pass by pointer for efficiency (avoid copying large structs)
void birthday(Person *p) {
    p->age++;
}

birthday(&p);

// Return struct from function
Person create_person(const char *name, int age) {
    Person p;
    strncpy(p.name, name, 49);
    p.name[49] = '\0';
    p.age = age;
    return p;   // Small structs OK to return by value
}

// Array of structs
Person employees[100];
employees[0] = create_person("Frank", 40);
employees[0].age = 41;
```

### Struct Size and Padding

```c
struct A {
    char c;    // 1 byte
    int i;     // 4 bytes
    char d;    // 1 byte
};
// sizeof(struct A) is likely 12, not 6 — due to padding!
// Compiler adds padding for alignment: c(1)+pad(3)+i(4)+d(1)+pad(3) = 12

struct B {
    int i;     // 4 bytes
    char c;    // 1 byte
    char d;    // 1 byte
};
// sizeof(struct B) is likely 8: i(4)+c(1)+d(1)+pad(2) = 8

// Pack tightly (GCC extension — may cause performance issues)
struct __attribute__((packed)) Packed {
    char c;
    int i;
};
// sizeof = 5 (no padding)

// Use offsetof to find member position
#include <stddef.h>
offsetof(struct A, i)   // Byte offset of i within struct A
```

### Self-Referential Structs (Linked List Node)

```c
// A struct can contain a pointer to itself (not the struct itself)
typedef struct Node {
    int data;
    struct Node *next;   // Must use "struct Node" here, not "Node" (not defined yet)
} Node;

Node *n = malloc(sizeof(Node));
n->data = 42;
n->next = NULL;
```

### Unions

```c
// Union: all members share the same memory
// Size = size of largest member
// Only one member is "active" at a time

union Data {
    int    i;
    float  f;
    double d;
    char   str[20];
};

union Data u;
u.i = 42;
printf("%d\n", u.i);    // 42

u.f = 3.14f;
// Now u.i is meaningless — float overwrote int memory
printf("%f\n", u.f);    // 3.14

printf("%zu\n", sizeof(union Data));  // 20 (size of largest: str)

// Practical: tagged union (discriminated union)
typedef enum { TYPE_INT, TYPE_FLOAT, TYPE_STRING } Type;

typedef struct {
    Type type;
    union {
        int   i;
        float f;
        char  str[50];
    } value;
} Variant;

Variant v;
v.type = TYPE_INT;
v.value.i = 100;

if (v.type == TYPE_INT) printf("%d\n", v.value.i);
```

### Enumerations

```c
// enum assigns integer values to named constants
enum Day { MON, TUE, WED, THU, FRI, SAT, SUN };
// MON=0, TUE=1, WED=2, ... SUN=6

enum Day today = WED;
printf("%d\n", today);   // 2

// Custom values
enum Status {
    OK      = 200,
    CREATED = 201,
    NOT_FOUND = 404,
    SERVER_ERROR = 500
};

enum Status code = NOT_FOUND;
if (code == NOT_FOUND) printf("404 Not Found\n");

// typedef for cleaner syntax
typedef enum { FALSE = 0, TRUE = 1 } Bool;
Bool flag = TRUE;

// Bit-flag enum (powers of 2)
typedef enum {
    PERM_NONE    = 0,
    PERM_READ    = 1 << 0,   // 1
    PERM_WRITE   = 1 << 1,   // 2
    PERM_EXECUTE = 1 << 2    // 4
} Permission;

Permission p = PERM_READ | PERM_WRITE;  // 3
if (p & PERM_READ) printf("Can read\n");
```

---

## 11. Dynamic Memory Management

### malloc, calloc, realloc, free

```c
#include <stdlib.h>

// malloc — allocate uninitialized memory
int *arr = malloc(10 * sizeof(int));
if (arr == NULL) {
    fprintf(stderr, "Allocation failed\n");
    exit(EXIT_FAILURE);
}
// Contents are garbage — you must initialize!
for (int i = 0; i < 10; i++) arr[i] = 0;

// calloc — allocate zero-initialized memory
int *arr2 = calloc(10, sizeof(int));
// Contents are all 0
if (arr2 == NULL) { /* handle error */ }

// realloc — resize previously allocated block
int *arr3 = malloc(5 * sizeof(int));
int *tmp = realloc(arr3, 10 * sizeof(int));  // Grow to 10 elements
if (tmp == NULL) {
    // realloc failed — arr3 is still valid!
    free(arr3);
    exit(EXIT_FAILURE);
}
arr3 = tmp;   // Only update pointer if realloc succeeded

// free — release memory
free(arr);
free(arr2);
free(arr3);
arr = NULL;   // Good practice: null the pointer after freeing

// Freeing NULL is safe (does nothing)
free(NULL);   // OK
```

### Dynamic 2D Arrays

```c
int rows = 3, cols = 4;

// Method 1: Array of pointers (non-contiguous)
int **matrix = malloc(rows * sizeof(int *));
for (int i = 0; i < rows; i++) {
    matrix[i] = malloc(cols * sizeof(int));
}

matrix[1][2] = 42;   // Access like normal 2D array

// Free (reverse order of allocation)
for (int i = 0; i < rows; i++) free(matrix[i]);
free(matrix);

// Method 2: Single block (contiguous — better for cache performance)
int *flat = malloc(rows * cols * sizeof(int));
// Access: row r, column c → flat[r * cols + c]
flat[1 * cols + 2] = 42;
free(flat);

// Method 3: Pointer to VLA-style (C99) — clean syntax
int (*mat)[cols] = malloc(rows * sizeof(*mat));
mat[1][2] = 42;
free(mat);
```

### Memory Management Best Practices

```c
// 1. Always check return value of malloc/calloc/realloc
void *p = malloc(1000);
if (!p) {
    perror("malloc");    // Print system error message
    exit(EXIT_FAILURE);
}

// 2. Match every malloc with exactly one free
// 3. Never double-free
// 4. Never use memory after freeing (use-after-free)
// 5. Never read/write past allocation bounds

// Wrapper with error checking (common pattern)
void *safe_malloc(size_t size) {
    void *p = malloc(size);
    if (p == NULL) {
        fprintf(stderr, "Out of memory — requested %zu bytes\n", size);
        exit(EXIT_FAILURE);
    }
    return p;
}

// 6. Use valgrind to detect leaks:
// valgrind --leak-check=full --track-origins=yes ./app
```

### Memory Layout of a C Program

```
High address
┌──────────────────────┐
│     Stack            │ ← Local variables, function call frames
│     (grows down)     │   (automatic allocation/deallocation)
├──────────────────────┤
│          ↓           │
│                      │
│          ↑           │
├──────────────────────┤
│     Heap             │ ← malloc/calloc/realloc/free
│     (grows up)       │   (manual allocation/deallocation)
├──────────────────────┤
│     BSS Segment      │ ← Uninitialized global/static variables (zeroed)
├──────────────────────┤
│     Data Segment     │ ← Initialized global/static variables
├──────────────────────┤
│     Text Segment     │ ← Program code (read-only)
└──────────────────────┘
Low address
```

---

## 12. File I/O

### Opening, Reading, Writing, Closing

```c
#include <stdio.h>

// Open file
FILE *fp = fopen("data.txt", "r");   // "r" = read
if (fp == NULL) {
    perror("fopen");   // "fopen: No such file or directory"
    return 1;
}

// Write text
FILE *out = fopen("output.txt", "w");  // "w" = write (creates or truncates)
fprintf(out, "Hello, %s!\n", "world");
fputs("Another line\n", out);
fputc('A', out);

// Read text
char buffer[256];
while (fgets(buffer, sizeof(buffer), fp) != NULL) {
    printf("%s", buffer);    // fgets keeps '\n'
}

// Read one character at a time
int ch;
while ((ch = fgetc(fp)) != EOF) {
    putchar(ch);
}

// Read formatted
int n;
float f;
fscanf(fp, "%d %f", &n, &f);

// Close file
fclose(fp);
fclose(out);
```

### File Open Modes

| Mode | Description |
|------|-------------|
| `"r"` | Read — file must exist |
| `"w"` | Write — creates new or truncates existing |
| `"a"` | Append — creates new or adds to end |
| `"r+"` | Read and write — file must exist |
| `"w+"` | Read and write — creates new or truncates |
| `"a+"` | Read and append — creates new or adds to end |
| `"rb"` | Binary read |
| `"wb"` | Binary write |
| `"ab"` | Binary append |

### Binary File I/O

```c
// Write binary data
typedef struct {
    char name[50];
    int age;
    double salary;
} Employee;

Employee emp = {"Grace", 32, 85000.0};

FILE *fp = fopen("employees.bin", "wb");
fwrite(&emp, sizeof(Employee), 1, fp);       // Write 1 employee
fwrite(emp_array, sizeof(Employee), 10, fp); // Write 10 employees
fclose(fp);

// Read binary data
FILE *fp2 = fopen("employees.bin", "rb");

Employee emp_read;
size_t count = fread(&emp_read, sizeof(Employee), 1, fp2);
if (count != 1) {
    fprintf(stderr, "Read error or EOF\n");
}

printf("Name: %s, Age: %d\n", emp_read.name, emp_read.age);
fclose(fp2);
```

### File Positioning

```c
FILE *fp = fopen("data.txt", "r+");

// Seek
fseek(fp, 0, SEEK_SET);    // Go to beginning
fseek(fp, 0, SEEK_END);    // Go to end
fseek(fp, 10, SEEK_SET);   // Go to byte 10
fseek(fp, -5, SEEK_CUR);   // Go back 5 bytes from current position
fseek(fp, -10, SEEK_END);  // 10 bytes before end

// Get current position
long pos = ftell(fp);
printf("Position: %ld\n", pos);

// Reset to beginning
rewind(fp);    // Same as fseek(fp, 0, SEEK_SET) + clears error flag

// Get file size
fseek(fp, 0, SEEK_END);
long size = ftell(fp);
rewind(fp);
printf("File size: %ld bytes\n", size);

fclose(fp);
```

### File Status and Error Handling

```c
FILE *fp = fopen("missing.txt", "r");
if (fp == NULL) {
    perror("Error");               // Prints "Error: No such file or directory"
    printf("errno: %d\n", errno);  // Numeric error code
    return 1;
}

// Check for errors after operations
if (ferror(fp)) {
    fprintf(stderr, "I/O error\n");
    clearerr(fp);   // Clear error flag
}

if (feof(fp)) {
    printf("End of file reached\n");
}

// Remove and rename files
remove("old_file.txt");
rename("old.txt", "new.txt");

// Temporary file
FILE *tmp = tmpfile();   // Creates unnamed temp file, auto-deleted on close
```

### Reading an Entire File

```c
char *read_file(const char *filename, long *out_size) {
    FILE *fp = fopen(filename, "rb");
    if (!fp) return NULL;

    // Get size
    fseek(fp, 0, SEEK_END);
    long size = ftell(fp);
    rewind(fp);

    // Allocate buffer (+1 for null terminator)
    char *buf = malloc(size + 1);
    if (!buf) { fclose(fp); return NULL; }

    // Read all at once
    size_t read = fread(buf, 1, size, fp);
    buf[read] = '\0';   // Null-terminate

    fclose(fp);
    if (out_size) *out_size = (long)read;
    return buf;   // Caller must free!
}

long size;
char *content = read_file("data.txt", &size);
if (content) {
    printf("%s", content);
    free(content);
}
```

---

## 13. Preprocessor & Macros

### `#include`

```c
#include <stdio.h>      // System header (searched in standard paths)
#include "myheader.h"   // Local header (searched in current directory first)
```

### `#define` — Object-like Macros

```c
#define PI         3.14159265358979
#define MAX_SIZE   256
#define GREETING   "Hello, World!"
#define NEWLINE    '\n'
#define TRUE       1
#define FALSE      0

// Undefine
#undef PI
```

### `#define` — Function-like Macros

```c
// Arguments are substituted textually — use parentheses!
#define SQUARE(x)    ((x) * (x))
#define MAX(a, b)    ((a) > (b) ? (a) : (b))
#define ABS(x)       ((x) < 0 ? -(x) : (x))
#define SWAP(T, a, b) do { T _tmp = a; a = b; b = _tmp; } while(0)

// Why parentheses? Without them:
// #define SQUARE(x)  x * x
// SQUARE(1+2)  →  1+2 * 1+2  →  1+2+2 = 5  (WRONG!)
// With them: ((1+2) * (1+2)) = 9  (correct)

// Why do { } while(0)? Allows use in if/else:
SWAP(int, a, b);   // Works in all contexts

// Variadic macros (C99)
#define DEBUG_PRINT(fmt, ...) fprintf(stderr, fmt, ##__VA_ARGS__)
#define LOG(level, fmt, ...) printf("[%s] " fmt "\n", level, ##__VA_ARGS__)

DEBUG_PRINT("x = %d\n", x);
LOG("ERROR", "Failed to open file: %s", filename);
```

### Conditional Compilation

```c
// #ifdef / #ifndef / #endif
#ifdef DEBUG
    printf("Debug: x = %d\n", x);
#endif

#ifndef MAX_SIZE
    #define MAX_SIZE 100
#endif

// #if / #elif / #else (can use expressions)
#if defined(PLATFORM_LINUX)
    // Linux-specific code
#elif defined(PLATFORM_WINDOWS)
    // Windows-specific code
#else
    // Other platforms
#endif

#if MAX_SIZE > 100
    // Large buffer code
#else
    // Small buffer code
#endif

// Header guard (prevents double inclusion)
#ifndef MYHEADER_H
#define MYHEADER_H

// ... header contents ...

#endif // MYHEADER_H

// Modern alternative (GCC, Clang, MSVC)
#pragma once   // Simpler than header guards (but not strictly standard)
```

### Predefined Macros

```c
__FILE__        // Current filename as string: "main.c"
__LINE__        // Current line number: 42
__DATE__        // Compilation date: "Jan 14 2025"
__TIME__        // Compilation time: "12:34:56"
__func__        // Current function name (C99): "main"
__STDC__        // 1 if compiler conforms to C standard
__STDC_VERSION__ // 199901L (C99), 201112L (C11), 201710L (C17)

// Debugging helper using predefined macros:
#define ASSERT(cond) do { \
    if (!(cond)) { \
        fprintf(stderr, "Assertion failed: %s, file %s, line %d\n", \
                #cond, __FILE__, __LINE__); \
        abort(); \
    } \
} while(0)

// Stringification with #
#define TO_STRING(x) #x
TO_STRING(hello)    // → "hello"
TO_STRING(42)       // → "42"

// Token pasting with ##
#define CONCAT(a, b) a##b
int CONCAT(my, Var) = 5;  // → int myVar = 5;
```

---

## 14. Bitwise Operations & Bit Fields

### Advanced Bit Manipulation

```c
// Test if n is a power of 2
int is_power_of_two(unsigned n) {
    return n > 0 && (n & (n - 1)) == 0;
}

// Count set bits (popcount)
int count_bits(unsigned n) {
    int count = 0;
    while (n) { count += n & 1; n >>= 1; }
    return count;
}

// Reverse bits (8-bit)
uint8_t reverse_bits(uint8_t b) {
    b = (b & 0xF0) >> 4 | (b & 0x0F) << 4;
    b = (b & 0xCC) >> 2 | (b & 0x33) << 2;
    b = (b & 0xAA) >> 1 | (b & 0x55) << 1;
    return b;
}

// Get/Set/Clear/Toggle bit at position pos
#define BIT_GET(x, pos)     (((x) >> (pos)) & 1)
#define BIT_SET(x, pos)     ((x) |=  (1U << (pos)))
#define BIT_CLEAR(x, pos)   ((x) &= ~(1U << (pos)))
#define BIT_TOGGLE(x, pos)  ((x) ^=  (1U << (pos)))

// Extract bits [high:low] from x
#define BITS(x, high, low)  (((x) >> (low)) & ((1U << ((high)-(low)+1)) - 1))

// Fast multiply/divide by powers of 2
x << 3    // x * 8
x >> 2    // x / 4 (unsigned — for signed, use division for negative values)

// Endianness conversion (swap bytes of 32-bit integer)
uint32_t swap_endian32(uint32_t x) {
    return ((x & 0xFF000000) >> 24) |
           ((x & 0x00FF0000) >>  8) |
           ((x & 0x0000FF00) <<  8) |
           ((x & 0x000000FF) << 24);
}
```

### Bit Fields in Structs

```c
// Bit fields allow packing flags into a struct tightly
struct Flags {
    unsigned int read    : 1;   // 1 bit
    unsigned int write   : 1;   // 1 bit
    unsigned int execute : 1;   // 1 bit
    unsigned int sticky  : 1;   // 1 bit
    unsigned int uid     : 1;   // 1 bit
    unsigned int         : 3;   // 3 unnamed padding bits
    unsigned int mode    : 4;   // 4 bits
    // Total: 12 bits, stored in one int
};

struct Flags f = {0};
f.read = 1;
f.write = 1;
f.execute = 0;

printf("read: %d, write: %d\n", f.read, f.write);

// Bit fields are NOT portable across compilers/platforms
// Avoid for network protocols or file formats — use explicit masking instead
```

---

## 15. Advanced Pointers & Function Pointers

### Function Pointers

```c
// A function pointer holds the address of a function
// Declaration: return_type (*name)(param_types)

int add(int a, int b) { return a + b; }
int sub(int a, int b) { return a - b; }
int mul(int a, int b) { return a * b; }

// Declare function pointer
int (*operation)(int, int);

// Assign
operation = add;         // No & needed (function name decays to pointer)
operation = &add;        // Also valid

// Call
int result = operation(3, 4);    // 7
int result2 = (*operation)(3,4); // Also valid (equivalent)

// Array of function pointers
int (*ops[3])(int, int) = {add, sub, mul};
printf("%d\n", ops[0](10, 5));  // 15 (add)
printf("%d\n", ops[1](10, 5));  // 5  (sub)
printf("%d\n", ops[2](10, 5));  // 50 (mul)

// Function pointer as parameter (callback)
void apply(int *arr, int n, int (*transform)(int)) {
    for (int i = 0; i < n; i++) arr[i] = transform(arr[i]);
}

int double_it(int x) { return x * 2; }
int negate(int x)    { return -x; }

int arr[] = {1, 2, 3, 4, 5};
apply(arr, 5, double_it);   // {2, 4, 6, 8, 10}
apply(arr, 5, negate);      // {-2,-4,-6,-8,-10}

// typedef for cleaner syntax
typedef int (*BinaryOp)(int, int);

BinaryOp op = add;
printf("%d\n", op(3, 4));

// Function returning a function pointer
BinaryOp get_operation(char op_char) {
    switch (op_char) {
        case '+': return add;
        case '-': return sub;
        case '*': return mul;
        default:  return NULL;
    }
}
```

### qsort and bsearch (Standard Library Callbacks)

```c
#include <stdlib.h>

// qsort — sort any array using a comparison function
int cmp_int(const void *a, const void *b) {
    int x = *(const int *)a;
    int y = *(const int *)b;
    return (x > y) - (x < y);   // Returns -1, 0, or 1
}

int cmp_str(const void *a, const void *b) {
    return strcmp(*(const char **)a, *(const char **)b);
}

int arr[] = {5, 2, 8, 1, 9, 3, 7, 4, 6};
qsort(arr, 9, sizeof(int), cmp_int);   // Sort ascending

// Sort descending:
int cmp_int_desc(const void *a, const void *b) {
    return -((*(const int *)a > *(const int *)b) -
             (*(const int *)a < *(const int *)b));
}

// bsearch — binary search (array must be sorted)
int target = 7;
int *found = bsearch(&target, arr, 9, sizeof(int), cmp_int);
if (found) printf("Found at index %ld\n", found - arr);
else       printf("Not found\n");
```

### Complex Pointer Declarations

```c
// How to read complex declarations: "inside-out, right-to-left"

int *p;                  // p: pointer to int
int *arr[5];             // arr: array[5] of pointers to int
int (*arr)[5];           // arr: pointer to array[5] of int
int *func(void);         // func: function returning pointer to int
int (*func)(void);       // func: pointer to function returning int
int (*arr[5])(void);     // arr: array[5] of pointers to functions returning int
const char *const p;     // p: const pointer to const char

// Use typedef to clarify
typedef int (*IntFunc)(void);   // IntFunc is: pointer to function returning int
IntFunc callbacks[5];           // array of 5 function pointers
```

---

## 16. Linked Lists & Data Structures

### Singly Linked List

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct Node {
    int data;
    struct Node *next;
} Node;

// Create new node
Node *new_node(int data) {
    Node *n = malloc(sizeof(Node));
    if (!n) { perror("malloc"); exit(1); }
    n->data = data;
    n->next = NULL;
    return n;
}

// Insert at head — O(1)
void push(Node **head, int data) {
    Node *n = new_node(data);
    n->next = *head;
    *head = n;
}

// Insert at tail — O(n)
void append(Node **head, int data) {
    Node *n = new_node(data);
    if (*head == NULL) { *head = n; return; }
    Node *curr = *head;
    while (curr->next) curr = curr->next;
    curr->next = n;
}

// Delete first node with value
void delete(Node **head, int value) {
    Node *curr = *head, *prev = NULL;
    while (curr && curr->data != value) {
        prev = curr;
        curr = curr->next;
    }
    if (!curr) return;   // Not found
    if (prev) prev->next = curr->next;
    else *head = curr->next;
    free(curr);
}

// Print list
void print_list(const Node *head) {
    while (head) {
        printf("%d -> ", head->data);
        head = head->next;
    }
    printf("NULL\n");
}

// Reverse list — O(n)
void reverse(Node **head) {
    Node *prev = NULL, *curr = *head, *next = NULL;
    while (curr) {
        next = curr->next;
        curr->next = prev;
        prev = curr;
        curr = next;
    }
    *head = prev;
}

// Free entire list
void free_list(Node **head) {
    Node *curr = *head;
    while (curr) {
        Node *next = curr->next;
        free(curr);
        curr = next;
    }
    *head = NULL;
}

int main(void) {
    Node *head = NULL;
    append(&head, 1);
    append(&head, 2);
    append(&head, 3);
    push(&head, 0);
    print_list(head);    // 0 -> 1 -> 2 -> 3 -> NULL
    reverse(&head);
    print_list(head);    // 3 -> 2 -> 1 -> 0 -> NULL
    delete(&head, 2);
    print_list(head);    // 3 -> 1 -> 0 -> NULL
    free_list(&head);
    return 0;
}
```

### Stack (using linked list)

```c
typedef struct {
    Node *top;
    int size;
} Stack;

void stack_init(Stack *s) { s->top = NULL; s->size = 0; }

void stack_push(Stack *s, int data) {
    Node *n = new_node(data);
    n->next = s->top;
    s->top = n;
    s->size++;
}

int stack_pop(Stack *s) {
    if (!s->top) { fprintf(stderr, "Stack underflow\n"); exit(1); }
    int val = s->top->data;
    Node *tmp = s->top;
    s->top = s->top->next;
    free(tmp);
    s->size--;
    return val;
}

int stack_peek(const Stack *s) {
    if (!s->top) { fprintf(stderr, "Stack empty\n"); exit(1); }
    return s->top->data;
}

int stack_empty(const Stack *s) { return s->top == NULL; }
```

### Queue (using linked list)

```c
typedef struct {
    Node *front;
    Node *rear;
    int size;
} Queue;

void queue_init(Queue *q) { q->front = q->rear = NULL; q->size = 0; }

void enqueue(Queue *q, int data) {
    Node *n = new_node(data);
    if (!q->rear) { q->front = q->rear = n; }
    else { q->rear->next = n; q->rear = n; }
    q->size++;
}

int dequeue(Queue *q) {
    if (!q->front) { fprintf(stderr, "Queue empty\n"); exit(1); }
    int val = q->front->data;
    Node *tmp = q->front;
    q->front = q->front->next;
    if (!q->front) q->rear = NULL;
    free(tmp);
    q->size--;
    return val;
}
```

### Hash Table (simple)

```c
#define TABLE_SIZE 64

typedef struct Entry {
    char *key;
    int value;
    struct Entry *next;   // Chaining for collision resolution
} Entry;

typedef struct {
    Entry *buckets[TABLE_SIZE];
} HashTable;

unsigned int hash(const char *key) {
    unsigned int h = 5381;
    while (*key) h = h * 33 ^ (unsigned char)*key++;
    return h % TABLE_SIZE;
}

void ht_set(HashTable *ht, const char *key, int value) {
    unsigned int idx = hash(key);
    Entry *e = ht->buckets[idx];
    while (e) {
        if (strcmp(e->key, key) == 0) { e->value = value; return; }
        e = e->next;
    }
    Entry *new_e = malloc(sizeof(Entry));
    new_e->key = strdup(key);
    new_e->value = value;
    new_e->next = ht->buckets[idx];
    ht->buckets[idx] = new_e;
}

int ht_get(HashTable *ht, const char *key, int *out) {
    unsigned int idx = hash(key);
    Entry *e = ht->buckets[idx];
    while (e) {
        if (strcmp(e->key, key) == 0) { *out = e->value; return 1; }
        e = e->next;
    }
    return 0;   // Not found
}
```

---

## 17. Error Handling & Debugging

### errno and perror

```c
#include <errno.h>
#include <string.h>

FILE *fp = fopen("missing.txt", "r");
if (fp == NULL) {
    // errno is set by failed system calls
    printf("errno: %d\n", errno);           // e.g., 2
    printf("Error: %s\n", strerror(errno)); // "No such file or directory"
    perror("fopen");                         // "fopen: No such file or directory"
    return 1;
}

// Common errno values (from <errno.h>)
// EPERM   = 1  — Operation not permitted
// ENOENT  = 2  — No such file or directory
// EINTR   = 4  — Interrupted system call
// EIO     = 5  — I/O error
// ENOMEM  = 12 — Out of memory
// EACCES  = 13 — Permission denied
// EEXIST  = 17 — File exists
// EINVAL  = 22 — Invalid argument
// ENOSPC  = 28 — No space left on device
```

### assert

```c
#include <assert.h>

void divide(int a, int b) {
    assert(b != 0);      // If false: prints message and calls abort()
    printf("%d\n", a/b);
}

assert(ptr != NULL);
assert(index >= 0 && index < MAX);

// Disable in release builds (define NDEBUG before including assert.h):
// gcc -DNDEBUG main.c -o app
// Or: #define NDEBUG

// C11 static assertion (compile-time check)
#include <assert.h>
_Static_assert(sizeof(int) == 4, "int must be 4 bytes");
_Static_assert(sizeof(void *) >= 4, "pointer too small");
```

### Custom Error Handling

```c
// Return codes pattern
#define ERR_OK         0
#define ERR_IO        -1
#define ERR_MEMORY    -2
#define ERR_INVALID   -3
#define ERR_NOT_FOUND -4

const char *err_to_str(int err) {
    switch (err) {
        case ERR_OK:        return "Success";
        case ERR_IO:        return "I/O error";
        case ERR_MEMORY:    return "Memory allocation failed";
        case ERR_INVALID:   return "Invalid argument";
        case ERR_NOT_FOUND: return "Not found";
        default:            return "Unknown error";
    }
}

int read_config(const char *path, Config *out) {
    FILE *fp = fopen(path, "r");
    if (!fp) return ERR_IO;

    char *buf = malloc(1024);
    if (!buf) { fclose(fp); return ERR_MEMORY; }

    // ... parse config ...

    free(buf);
    fclose(fp);
    return ERR_OK;
}

// Goto pattern for cleanup on error
int process(const char *filename) {
    int result = ERR_OK;
    FILE *fp = NULL;
    char *buffer = NULL;

    fp = fopen(filename, "r");
    if (!fp) { result = ERR_IO; goto cleanup; }

    buffer = malloc(4096);
    if (!buffer) { result = ERR_MEMORY; goto cleanup; }

    // ... process ...

cleanup:
    if (buffer) free(buffer);
    if (fp) fclose(fp);
    return result;
}
```

### Debugging Techniques

```c
// Conditional debug printing
#ifdef DEBUG
    #define DBG(fmt, ...) fprintf(stderr, "[DBG %s:%d] " fmt "\n", \
                                  __func__, __LINE__, ##__VA_ARGS__)
#else
    #define DBG(fmt, ...) ((void)0)   // No-op in release
#endif

DBG("x = %d, ptr = %p", x, (void *)ptr);

// Hex dump of memory
void hex_dump(const void *data, size_t size) {
    const unsigned char *p = data;
    for (size_t i = 0; i < size; i++) {
        if (i % 16 == 0) printf("\n%04zx: ", i);
        printf("%02x ", p[i]);
    }
    printf("\n");
}

// Stack trace (Linux/GCC)
#include <execinfo.h>
void print_trace(void) {
    void *array[20];
    int size = backtrace(array, 20);
    char **strings = backtrace_symbols(array, size);
    for (int i = 0; i < size; i++) printf("%s\n", strings[i]);
    free(strings);
}
```

---

## 18. C Standard Library Deep Dive

### `<stdlib.h>` — General Utilities

```c
#include <stdlib.h>

// Memory
malloc(size)
calloc(count, size)
realloc(ptr, new_size)
free(ptr)

// Math
abs(-5)          // 5 (int)
labs(-5L)        // 5 (long)
llabs(-5LL)      // 5 (long long)
div(17, 5)       // Returns div_t {quot=3, rem=2}
ldiv(17L, 5L)    // long version

// Random numbers
srand(time(NULL));           // Seed (do once at start!)
int r = rand();              // [0, RAND_MAX]
int r2 = rand() % 100;      // [0, 99] — not perfectly uniform
// Better approach:
int rand_range(int lo, int hi) {
    return lo + rand() / (RAND_MAX / (hi - lo + 1) + 1);
}

// Conversions
atoi("42")           // String to int (no error handling)
atol("100000")       // String to long
atof("3.14")         // String to double
strtol("42", NULL, 10)     // String to long (with error detection)
strtoul("FF", NULL, 16)    // String to unsigned long (hex)
strtod("3.14", NULL)       // String to double
itoa(42, buf, 10)          // Int to string (not standard — use sprintf)

// Process control
exit(EXIT_SUCCESS);    // Normal termination
exit(EXIT_FAILURE);    // Error termination
abort();               // Abnormal termination (generates SIGABRT)
atexit(cleanup_func);  // Register function to call on exit

// Environment
char *path = getenv("PATH");       // Get environment variable
setenv("KEY", "value", 1);        // Set (POSIX)
system("ls -l");                   // Run shell command (use carefully!)
```

### `<stdio.h>` — I/O Functions

```c
#include <stdio.h>

// Standard streams
stdin     // Standard input
stdout    // Standard output
stderr    // Standard error

// printf family
printf(fmt, ...)               // To stdout
fprintf(fp, fmt, ...)          // To file
sprintf(buf, fmt, ...)         // To string (unsafe — use snprintf)
snprintf(buf, n, fmt, ...)     // To string, max n bytes (safe)
vprintf, vfprintf, vsprintf, vsnprintf  // va_list versions

// scanf family
scanf(fmt, ...)                // From stdin
fscanf(fp, fmt, ...)           // From file
sscanf(str, fmt, ...)          // From string

// Character I/O
int c = getchar();             // Read one char from stdin
putchar('A');                  // Write one char to stdout
int c2 = fgetc(fp);            // Read one char from file
fputc('A', fp);                // Write one char to file
ungetc(c, fp);                 // Push char back to stream (one char max)

// String I/O
char *s = fgets(buf, n, fp);   // Read line from file (safe)
int ret = fputs(str, fp);      // Write string to file
puts(str);                     // Write string + newline to stdout

// Buffer control
fflush(stdout);    // Force write buffered output
fflush(NULL);      // Flush all output streams
setbuf(fp, NULL);  // Disable buffering
setvbuf(fp, buf, _IOFBF, size);  // Set buffer mode
// _IOFBF = fully buffered, _IOLBF = line buffered, _IONBF = unbuffered
```

### `<string.h>` — String and Memory Functions

```c
// (covered in Strings section, adding memory functions)
#include <string.h>

// Memory
void *memset(void *s, int c, size_t n);        // Fill n bytes with c
void *memcpy(void *dst, const void *src, size_t n);    // Copy n bytes
void *memmove(void *dst, const void *src, size_t n);   // Copy (handles overlap)
int   memcmp(const void *s1, const void *s2, size_t n); // Compare n bytes
void *memchr(const void *s, int c, size_t n);  // Find byte c in first n bytes
```

### `<time.h>` — Time Functions

```c
#include <time.h>

// Current time
time_t now = time(NULL);          // Seconds since epoch (Jan 1, 1970)
printf("Epoch: %ld\n", now);

// Convert to human-readable
struct tm *t = localtime(&now);   // Local time
struct tm *u = gmtime(&now);      // UTC time

// Format time as string
char buf[64];
strftime(buf, sizeof(buf), "%Y-%m-%d %H:%M:%S", t);
printf("%s\n", buf);              // "2025-01-14 15:30:00"

// Individual components
t->tm_year + 1900   // Year (years since 1900)
t->tm_mon + 1       // Month (0-11, add 1 for 1-12)
t->tm_mday          // Day of month (1-31)
t->tm_hour          // Hour (0-23)
t->tm_min           // Minute (0-59)
t->tm_sec           // Second (0-60)
t->tm_wday          // Day of week (0=Sunday)
t->tm_yday          // Day of year (0-365)

// Convert struct tm back to time_t
time_t t2 = mktime(t);

// High-resolution timing
clock_t start = clock();
// ... code to time ...
clock_t end = clock();
double seconds = (double)(end - start) / CLOCKS_PER_SEC;
printf("Elapsed: %.3f seconds\n", seconds);
```

### `<limits.h>` and `<float.h>`

```c
#include <limits.h>

CHAR_BIT      // Bits per char: 8
CHAR_MAX      // Max char: 127
CHAR_MIN      // Min char: -128
UCHAR_MAX     // Max unsigned char: 255
SHRT_MAX      // Max short: 32767
INT_MAX       // Max int: 2147483647
INT_MIN       // Min int: -2147483648
UINT_MAX      // Max unsigned int: 4294967295
LONG_MAX      // Max long: 2147483647 or 9223372036854775807
LLONG_MAX     // Max long long: 9223372036854775807

#include <float.h>

FLT_MAX       // Max float: ~3.4e+38
FLT_MIN       // Min positive float: ~1.2e-38
FLT_EPSILON   // Smallest x such that 1.0+x != 1.0: ~1.2e-7
DBL_MAX       // Max double: ~1.8e+308
DBL_EPSILON   // ~2.2e-16
FLT_DIG       // Significant decimal digits (float): 6
DBL_DIG       // Significant decimal digits (double): 15
```

---

## 19. Advanced Topics

### Multithreading (POSIX Threads)

```c
#include <pthread.h>
// Compile with: gcc prog.c -lpthread

// Basic thread creation
void *thread_func(void *arg) {
    int *num = (int *)arg;
    printf("Thread got: %d\n", *num);
    return NULL;
}

pthread_t tid;
int val = 42;
pthread_create(&tid, NULL, thread_func, &val);
pthread_join(tid, NULL);   // Wait for thread to finish

// Mutex (mutual exclusion lock)
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

void safe_increment(int *counter) {
    pthread_mutex_lock(&lock);
    (*counter)++;
    pthread_mutex_unlock(&lock);
}

// Cleanup
pthread_mutex_destroy(&lock);

// Thread-local storage
pthread_key_t tls_key;
pthread_key_create(&tls_key, free);
pthread_setspecific(tls_key, malloc(sizeof(int)));
int *local = pthread_getspecific(tls_key);
```

### Signal Handling

```c
#include <signal.h>

volatile sig_atomic_t running = 1;   // Safe to access in signal handler

void handle_sigint(int sig) {
    (void)sig;   // Suppress unused parameter warning
    running = 0;
    write(STDOUT_FILENO, "\nCaught SIGINT\n", 15);  // printf not signal-safe!
}

int main(void) {
    signal(SIGINT,  handle_sigint);   // Ctrl+C
    signal(SIGTERM, handle_sigint);   // Kill signal

    // More robust: sigaction
    struct sigaction sa = {0};
    sa.sa_handler = handle_sigint;
    sigemptyset(&sa.sa_mask);
    sa.sa_flags = SA_RESTART;         // Restart interrupted system calls
    sigaction(SIGINT, &sa, NULL);

    while (running) {
        printf("Running...\n");
        sleep(1);
    }
    return 0;
}
```

### `setjmp` / `longjmp` (Non-local jumps)

```c
#include <setjmp.h>

jmp_buf jump_point;

void deeply_nested(void) {
    longjmp(jump_point, 1);   // Jump back to setjmp call, return value 1
}

int main(void) {
    int code = setjmp(jump_point);   // Returns 0 on first call
    if (code == 0) {
        printf("Starting...\n");
        deeply_nested();             // This triggers the jump
        printf("This is never printed\n");
    } else {
        printf("Jumped back with code: %d\n", code);  // Prints: 1
    }
    return 0;
}
// Use sparingly — disrupts normal flow, interacts poorly with destructors
```

### Generic Programming with void*

```c
// qsort-style generic comparison
typedef int (*Comparator)(const void *, const void *);

void generic_sort(void *arr, int n, size_t elem_size, Comparator cmp) {
    qsort(arr, n, elem_size, cmp);
}

// Generic stack
typedef struct {
    void  *data;
    int    top;
    int    capacity;
    size_t elem_size;
} GenStack;

GenStack *gs_create(int capacity, size_t elem_size) {
    GenStack *s = malloc(sizeof(GenStack));
    s->data = malloc(capacity * elem_size);
    s->top = -1;
    s->capacity = capacity;
    s->elem_size = elem_size;
    return s;
}

void gs_push(GenStack *s, const void *elem) {
    s->top++;
    memcpy((char *)s->data + s->top * s->elem_size, elem, s->elem_size);
}

void gs_pop(GenStack *s, void *out) {
    memcpy(out, (char *)s->data + s->top * s->elem_size, s->elem_size);
    s->top--;
}
```

### `_Generic` (C11 — Type-generic expressions)

```c
// C11 allows selecting expressions based on type
#define print_value(x) _Generic((x),  \
    int:          printf("%d\n", x),   \
    double:       printf("%f\n", x),   \
    char *:       printf("%s\n", x),   \
    const char *: printf("%s\n", x),   \
    default:      printf("unknown\n")  \
)

print_value(42);       // Prints: 42
print_value(3.14);     // Prints: 3.140000
print_value("hello");  // Prints: hello

// Type-generic absolute value
#define ABS(x) _Generic((x),  \
    int:    abs(x),            \
    long:   labs(x),           \
    double: fabs(x),           \
    float:  fabsf(x)           \
)
```

### Flexible Array Members (C99)

```c
// A struct can end with an array of unspecified length
typedef struct {
    int length;
    int data[];   // Flexible array member — must be last
} IntArray;

// Allocate with extra space for data
IntArray *arr = malloc(sizeof(IntArray) + 10 * sizeof(int));
arr->length = 10;
arr->data[0] = 42;
arr->data[9] = 99;

free(arr);
```

### Alignment and `_Alignas` (C11)

```c
#include <stdalign.h>

// Request minimum alignment
_Alignas(16) int aligned_arr[4];   // 16-byte aligned
_Alignas(64) char cache_line[64];  // Cache-line aligned

// Get alignment requirement
printf("%zu\n", _Alignof(int));    // Usually 4
printf("%zu\n", _Alignof(double)); // Usually 8
printf("%zu\n", _Alignof(max_align_t)); // Max fundamental alignment
```

---

## 20. Best Practices, Security & Performance

### Security — Common Vulnerabilities

```c
// ❌ BUFFER OVERFLOW — most common C vulnerability
char buf[10];
strcpy(buf, "This is way too long!");   // Writes past end — stack corruption!
gets(buf);                              // NEVER USE gets() — removed in C11

// ✅ Safe alternatives
strncpy(buf, src, sizeof(buf) - 1);
buf[sizeof(buf) - 1] = '\0';           // Ensure null termination
fgets(buf, sizeof(buf), stdin);         // Reads max sizeof(buf)-1 chars + '\n'
snprintf(buf, sizeof(buf), "%s", src);  // Safest for formatting

// ❌ FORMAT STRING VULNERABILITY
printf(user_input);           // If user types "%s%s%s" — can crash/leak memory!
// ✅ Always use a literal format string
printf("%s", user_input);     // Safe

// ❌ INTEGER OVERFLOW
unsigned char x = 250;
x += 10;    // Wraps around to 4 (0-255 range)

int a = INT_MAX;
a++;        // Undefined behavior (signed overflow)!

// ✅ Check before operation
if (a > INT_MAX - b) { /* overflow would occur */ }

// ❌ SQL/Shell injection via system()
char cmd[100];
sprintf(cmd, "ls %s", user_dir);   // If user_dir = "; rm -rf /" → disaster!
system(cmd);
// ✅ Avoid system() with user input. Use execv/execvp (POSIX) instead.

// ❌ Use after free
free(ptr);
ptr->data = 5;    // Undefined behavior
// ✅ Always null after free
free(ptr);
ptr = NULL;

// ❌ Uninitialized memory reads
int arr[100];
int sum = 0;
for (int i = 0; i <= 100; i++) sum += arr[i];  // Off-by-one + uninitialized!
```

### Performance Tips

```c
// 1. Cache efficiency — access memory sequentially (row-major)
// BAD (column-major access — cache miss on every iteration)
for (int j = 0; j < N; j++)
    for (int i = 0; i < N; i++)
        sum += matrix[i][j];

// GOOD (row-major access — cache-friendly)
for (int i = 0; i < N; i++)
    for (int j = 0; j < N; j++)
        sum += matrix[i][j];

// 2. Minimize allocations in loops
// BAD
for (int i = 0; i < 1000; i++) {
    int *tmp = malloc(size);
    process(tmp);
    free(tmp);   // Repeated alloc/free is slow
}

// GOOD
int *tmp = malloc(size);
for (int i = 0; i < 1000; i++) {
    process(tmp);
}
free(tmp);

// 3. Use appropriate data types
// Prefer int over char/short for loop counters (avoids sign extension)
for (int i = 0; i < n; i++) { }  // Good

// 4. Avoid function call overhead in tight loops
// Consider static inline for small, frequently called functions
static inline int min(int a, int b) { return a < b ? a : b; }

// 5. String operations — use memcpy/memset for bulk operations
memset(buf, 0, sizeof(buf));         // Faster than a loop
memcpy(dst, src, n);                 // Faster than a byte-by-byte loop

// 6. Compiler hints
// Avoid variable-length arrays for large sizes (stack overflow risk)
// Use restrict to tell compiler pointers don't alias:
void add_arrays(int n,
                int * restrict a,
                const int * restrict b,
                const int * restrict c) {
    for (int i = 0; i < n; i++) a[i] = b[i] + c[i];
}
```

### Defensive Coding Checklist

```c
// 1. Initialize ALL variables
int x = 0;
char buf[100] = {0};
int *ptr = NULL;

// 2. Always check return values
FILE *fp = fopen(...);
if (!fp) { /* handle error */ }

int *p = malloc(n);
if (!p) { /* handle error */ }

int n = scanf("%d", &x);
if (n != 1) { /* handle error */ }

// 3. Bounds check arrays
if (index >= 0 && index < size) arr[index] = val;

// 4. Validate all input
if (n < 0 || n > MAX_SIZE) { return ERR_INVALID; }

// 5. Null-terminate all strings explicitly
strncpy(dst, src, sizeof(dst) - 1);
dst[sizeof(dst) - 1] = '\0';

// 6. Free memory exactly once, set to NULL
free(ptr);
ptr = NULL;

// 7. Use const for read-only parameters
void process(const char *str, const int *arr, int n);

// 8. Minimize global variables
// 9. Keep functions small and single-purpose
// 10. Use meaningful names: num_students not n, is_valid not v
```

### Code Organization (Header / Source Split)

```c
// math_utils.h — header file
#ifndef MATH_UTILS_H
#define MATH_UTILS_H

// Public API declarations only
int gcd(int a, int b);
int lcm(int a, int b);
double power(double base, int exp);

#endif // MATH_UTILS_H

// math_utils.c — implementation
#include "math_utils.h"

int gcd(int a, int b) {
    while (b) { int t = b; b = a % b; a = t; }
    return a;
}

int lcm(int a, int b) {
    return (a / gcd(a, b)) * b;   // Divide first to reduce overflow risk
}

double power(double base, int exp) {
    if (exp == 0) return 1.0;
    if (exp < 0) return 1.0 / power(base, -exp);
    double half = power(base, exp / 2);
    if (exp % 2 == 0) return half * half;
    return base * half * half;
}

// main.c
#include <stdio.h>
#include "math_utils.h"

int main(void) {
    printf("GCD(48, 18) = %d\n", gcd(48, 18));
    printf("LCM(4, 6) = %d\n", lcm(4, 6));
    return 0;
}
```

---

## Quick Reference

### All Format Specifiers

| Specifier | Type | Example |
|-----------|------|---------|
| `%d` or `%i` | `int` | `printf("%d", 42)` → `42` |
| `%u` | `unsigned int` | `printf("%u", 42u)` → `42` |
| `%ld` | `long` | `printf("%ld", 1L)` |
| `%lld` | `long long` | `printf("%lld", 1LL)` |
| `%lu` | `unsigned long` | `printf("%lu", 1UL)` |
| `%llu` | `unsigned long long` | `printf("%llu", 1ULL)` |
| `%hd` | `short` | `printf("%hd", s)` |
| `%f` | `float` / `double` | `printf("%f", 3.14)` → `3.140000` |
| `%lf` | `double` (scanf) | `scanf("%lf", &d)` |
| `%Lf` | `long double` | `printf("%Lf", ld)` |
| `%e` | Scientific (float) | `printf("%e", 12345.6)` → `1.234560e+04` |
| `%g` | Shorter of %f/%e | `printf("%g", 12345.6)` → `12345.6` |
| `%c` | `char` | `printf("%c", 'A')` → `A` |
| `%s` | `char *` (string) | `printf("%s", "hi")` → `hi` |
| `%p` | pointer | `printf("%p", ptr)` → `0x7ffd...` |
| `%x` | Hex (lowercase) | `printf("%x", 255)` → `ff` |
| `%X` | Hex (uppercase) | `printf("%X", 255)` → `FF` |
| `%o` | Octal | `printf("%o", 8)` → `10` |
| `%zu` | `size_t` | `printf("%zu", sizeof(int))` → `4` |
| `%%` | Literal `%` | `printf("100%%")` → `100%` |

### Essential Headers

| Header | Key Contents |
|--------|-------------|
| `<stdio.h>` | `printf`, `scanf`, `fopen`, `fclose`, `fgets`, `fprintf` |
| `<stdlib.h>` | `malloc`, `free`, `exit`, `atoi`, `rand`, `srand`, `qsort` |
| `<string.h>` | `strlen`, `strcpy`, `strncpy`, `strcmp`, `strcat`, `memcpy`, `memset` |
| `<math.h>` | `sqrt`, `pow`, `abs`, `sin`, `cos`, `log`, `ceil`, `floor` |
| `<ctype.h>` | `isalpha`, `isdigit`, `toupper`, `tolower`, `isspace` |
| `<time.h>` | `time`, `clock`, `localtime`, `strftime` |
| `<stdbool.h>` | `bool`, `true`, `false` (C99) |
| `<stdint.h>` | `int8_t`, `int32_t`, `uint64_t`, etc. (C99) |
| `<stddef.h>` | `NULL`, `size_t`, `ptrdiff_t`, `offsetof` |
| `<errno.h>` | `errno`, error constants (`ENOENT`, `ENOMEM`, etc.) |
| `<assert.h>` | `assert`, `_Static_assert` (C11) |
| `<limits.h>` | `INT_MAX`, `INT_MIN`, `CHAR_MAX`, etc. |
| `<float.h>` | `FLT_MAX`, `DBL_EPSILON`, etc. |
| `<signal.h>` | `signal`, `sigaction`, `SIGINT`, `SIGTERM` |
| `<setjmp.h>` | `setjmp`, `longjmp` |
| `<stdarg.h>` | `va_list`, `va_start`, `va_arg`, `va_end` |

### Common Compilation Commands

```bash
# Standard compilation
gcc -std=c11 -Wall -Wextra -o app main.c

# With math library
gcc -std=c11 -Wall -o app main.c -lm

# Debug build with sanitizers
gcc -std=c11 -g -fsanitize=address,undefined -o app main.c

# Optimized release build
gcc -std=c11 -O2 -DNDEBUG -o app main.c

# Multiple source files
gcc -std=c11 -Wall -o app main.c utils.c io.c

# Check for memory errors
valgrind --leak-check=full --show-leak-kinds=all ./app

# Static analysis
gcc -Wall -Wextra -Wpedantic -Wshadow -Wformat=2 main.c
```
