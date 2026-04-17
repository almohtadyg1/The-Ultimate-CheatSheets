# C Programming: A Complete Progressive Tutorial

---

## 1. What & Why

C is a compiled, statically typed, procedural systems programming language created in the early 1970s. It sits one level above assembly — giving you direct control over memory, bit manipulation, and hardware — while being portable across architectures.

Why learn C? Because every major operating system kernel (Linux, Windows, macOS), many interpreters and runtimes (CPython, Ruby MRI, V8), and most embedded systems are written in C. C defines the interface to the operating system (POSIX, Win32), so every language that calls syscalls ultimately does so through C conventions. Understanding C explains memory layout, pointers, undefined behavior, and performance characteristics that affect every language you use.

C is deliberately low-level: no garbage collector, no bounds checking, no exceptions. You get maximum control — and maximum responsibility. A single wrong pointer operation crashes the program. Understanding C makes you a significantly better programmer in any language.

---

## 2. Mental Model

C has a remarkably simple model: programs are functions that manipulate memory. Memory is a flat array of bytes. Variables are named regions of memory. Pointers are the addresses of those regions.

```
Program memory layout (simplified):
  Code    (.text)  — compiled instructions, read-only
  Globals (.data)  — global variables, static storage duration
  Heap             — manual allocation (malloc/free), grows upward
  Stack            — function call frames, local variables, grows downward

int x = 42;         // stored in .data (initialized global)
int y;              // stored in .bss (uninitialized global, zeroed)

void f() {
    int z = 10;     // stored on the stack (function's stack frame)
    int *p = malloc(sizeof(int)); // allocated on heap, p is on stack
    *p = 5;         // dereference: write 5 to the address stored in p
    free(p);        // return heap memory — forgetting this = memory leak
}
```

---

## 3. Progressive Examples

### Level 1: Fundamentals

```c
#include <stdio.h>    // Standard I/O (printf, scanf)
#include <stdlib.h>   // malloc, free, exit
#include <string.h>   // strlen, strcpy, strcmp, memset, memcpy

int main(void) {
    // Built-in types and their sizes (platform-dependent except for exact-width types)
    char    c = 'A';        // 1 byte, -128 to 127 (or 0-255 for unsigned char)
    short   s = 1000;       // typically 2 bytes
    int     i = 42;         // typically 4 bytes
    long    l = 1000000L;   // at least 4 bytes (8 on 64-bit Linux)
    float   f = 3.14f;      // 4 bytes, IEEE 754 single-precision
    double  d = 3.14159;    // 8 bytes, IEEE 754 double-precision

    // Use exact-width types from <stdint.h> when size matters:
    #include <stdint.h>
    int8_t  a8  = -128;     // exactly 8 bits
    uint32_t u32 = 4000000000U;  // exactly 32 bits, unsigned
    int64_t  i64 = -9000000000000LL;  // exactly 64 bits

    // Printf format specifiers
    printf("char: %c\n", c);
    printf("int: %d  hex: %x  octal: %o\n", i, i, i);
    printf("long: %ld\n", l);
    printf("float: %f  scientific: %e\n", f, f);
    printf("double: %.6f\n", d);
    printf("string: %s\n", "hello");
    printf("pointer: %p\n", (void*)&i);  // address of i

    // Control flow
    for (int j = 0; j < 5; j++) {
        if (j % 2 == 0) printf("%d is even\n", j);
        else             printf("%d is odd\n", j);
    }

    int n = 0;
    while (n < 3) { printf("n=%d\n", n); n++; }

    // Switch
    char grade = 'B';
    switch (grade) {
        case 'A': printf("Excellent\n"); break;
        case 'B': printf("Good\n");      break;
        default:  printf("OK\n");        break;
    }

    return 0;  // return 0 from main = success
}
```

### Level 2: Pointers — The Core of C

```c
#include <stdio.h>
#include <stdlib.h>

// Pointers: variables that hold memory addresses
int x = 42;
int *p = &x;     // & = "address of". p now holds the address of x.

printf("x = %d\n", x);    // 42
printf("&x = %p\n", &x);  // address like 0x7fff5fbff58c
printf("p = %p\n", p);    // same address
printf("*p = %d\n", *p);  // 42 — * dereferences: reads the value at the address

*p = 100;    // write through the pointer
printf("x = %d\n", x);    // 100 — x changed because p points to x

// Pass by reference (C only has pass-by-value — use pointers to simulate)
void increment(int *n) {
    (*n)++;    // dereference and increment
}
int count = 5;
increment(&count);   // pass address of count
printf("%d\n", count);   // 6

// Pointer arithmetic
int arr[] = {10, 20, 30, 40, 50};
int *ptr = arr;    // ptr points to arr[0]

printf("%d\n", *ptr);       // 10
printf("%d\n", *(ptr + 1)); // 20 — ptr+1 moves to next int (4 bytes ahead)
printf("%d\n", *(ptr + 2)); // 30
ptr++;                      // advance ptr to arr[1]
printf("%d\n", *ptr);       // 20

// Array name is a pointer to the first element
printf("%d\n", arr[0]);     // 10
printf("%d\n", *arr);       // 10 — equivalent!
printf("%d\n", arr[2]);     // 30
printf("%d\n", *(arr + 2)); // 30 — equivalent!

// NULL pointer: pointer that doesn't point to anything
int *null_ptr = NULL;
if (null_ptr == NULL) printf("null pointer\n");
// Dereferencing NULL (*null_ptr) → segfault (undefined behavior)

// void pointer: generic pointer, can be cast to any pointer type
void *generic = &x;
int *specific = (int*)generic;  // cast needed to dereference
```

### Level 3: Memory Management

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Stack memory: automatic duration, freed when function returns
void stack_example() {
    int arr[100];    // allocated on the stack
    arr[0] = 42;
    // arr is automatically freed when this function returns
    // NEVER return a pointer to a local variable — dangling pointer!
}

// Heap memory: manual duration, must be explicitly freed
int *heap_example(int n) {
    int *arr = malloc(n * sizeof(int));  // allocate n ints on heap
    if (arr == NULL) {
        fprintf(stderr, "malloc failed\n");
        exit(1);   // handle allocation failure
    }

    for (int i = 0; i < n; i++) {
        arr[i] = i * i;   // access like a regular array
    }
    return arr;   // safe: heap memory persists after function returns
}

int main(void) {
    int n = 5;
    int *squares = heap_example(n);

    for (int i = 0; i < n; i++) {
        printf("%d\n", squares[i]);
    }

    free(squares);   // MUST free heap memory — every malloc needs a free
    squares = NULL;  // convention: set to NULL after free to prevent use-after-free

    // Resizing heap memory
    int *resized = malloc(4 * sizeof(int));
    resized = realloc(resized, 8 * sizeof(int));  // resize to 8 ints
    if (resized == NULL) { /* handle error */ }
    free(resized);

    // Zeroed allocation
    int *zeroed = calloc(10, sizeof(int));  // allocates 10 ints, all zero
    free(zeroed);

    // Common memory errors (USE SANITIZERS: -fsanitize=address,undefined)
    // 1. Memory leak: malloc without free
    // 2. Use-after-free: accessing freed memory
    // 3. Double free: calling free twice on the same pointer
    // 4. Buffer overflow: writing past allocated bounds
    // 5. Reading uninitialized memory: undefined behavior

    return 0;
}
```

### Level 4: Strings, Structs, and Functions

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

// Strings in C: null-terminated char arrays
char s1[] = "hello";            // 6 chars: h,e,l,l,o,\0
char *s2 = "world";             // string literal, read-only
char s3[256];                   // buffer for building strings

printf("length: %zu\n", strlen(s1));  // 5 (doesn't count \0)

// String operations (CAREFUL: no bounds checking)
strcpy(s3, s1);                 // UNSAFE: can overflow s3
strncpy(s3, s1, sizeof(s3)-1); // SAFER: limits copy to buffer size
strncat(s3, " world", sizeof(s3)-strlen(s3)-1);  // safe concatenation
printf("%s\n", s3);

int cmp = strcmp(s1, "hello"); // 0 if equal, <0 if s1<s2, >0 if s1>s2

// Safer string functions (POSIX and C11 Annex K)
snprintf(s3, sizeof(s3), "Pi = %.4f", 3.14159);  // printf to buffer with size limit

// Structs: group related data
typedef struct {
    char name[64];
    int age;
    double gpa;
} Student;

Student alice;
strncpy(alice.name, "Alice", sizeof(alice.name)-1);
alice.name[sizeof(alice.name)-1] = '\0';  // ensure null termination
alice.age = 20;
alice.gpa = 3.8;

printf("%s: age %d, GPA %.1f\n", alice.name, alice.age, alice.gpa);

// Pointer to struct: use -> operator
Student *ptr = &alice;
printf("%s\n", ptr->name);   // equivalent to (*ptr).name

// Dynamic struct allocation
Student *s = malloc(sizeof(Student));
if (!s) { perror("malloc"); exit(1); }
strncpy(s->name, "Bob", sizeof(s->name)-1);
s->age = 22;
s->gpa = 3.5;
// ... use s ...
free(s);

// Function pointers: store functions as values
int compare_ints(const void *a, const void *b) {
    return (*(int*)a) - (*(int*)b);  // qsort-compatible comparator
}

int arr[] = {3, 1, 4, 1, 5, 9, 2, 6};
int n = sizeof(arr) / sizeof(arr[0]);  // idiom: array length
qsort(arr, n, sizeof(int), compare_ints);  // sorts in-place

// Variadic functions (like printf)
#include <stdarg.h>
double average(int count, ...) {
    va_list args;
    va_start(args, count);
    double sum = 0;
    for (int i = 0; i < count; i++) {
        sum += va_arg(args, double);
    }
    va_end(args);
    return sum / count;
}
printf("%.2f\n", average(4, 1.0, 2.0, 3.0, 4.0));  // 2.50
```

### Level 5: Files, Preprocessor, and Error Handling

```c
#include <stdio.h>
#include <errno.h>
#include <string.h>

// File I/O
FILE *f = fopen("data.txt", "r");   // r=read, w=write, a=append, rb=read binary
if (f == NULL) {
    perror("fopen");   // prints: "fopen: No such file or directory"
    return 1;
}

// Read line by line
char line[256];
while (fgets(line, sizeof(line), f)) {
    // fgets includes '\n' in the string
    line[strcspn(line, "\n")] = '\0';  // remove trailing newline
    printf("Line: %s\n", line);
}

// Or read character by character
int c;
while ((c = fgetc(f)) != EOF) {
    putchar(c);
}

fclose(f);

// Write to file
FILE *out = fopen("output.txt", "w");
fprintf(out, "Value: %d\n", 42);
fputs("Another line\n", out);
fclose(out);

// Error handling: check return values and errno
#include <errno.h>
FILE *missing = fopen("/nonexistent/path.txt", "r");
if (missing == NULL) {
    fprintf(stderr, "Error %d: %s\n", errno, strerror(errno));
    // "Error 2: No such file or directory"
}

// Preprocessor
#define MAX_SIZE 1024           // constant (prefer const int in C99+)
#define SQUARE(x) ((x) * (x))  // macro (use inline functions in C99+)
#define DEBUG_PRINT(fmt, ...) \
    fprintf(stderr, "[DEBUG] " fmt "\n", ##__VA_ARGS__)

#ifdef DEBUG
    DEBUG_PRINT("Value: %d", 42);
#endif

// Include guards (prevent double-inclusion in header files)
// In myheader.h:
#ifndef MYHEADER_H
#define MYHEADER_H
// ... declarations ...
#endif
```

### Level 6: Data Structures — Linked List in C

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct Node {
    int value;
    struct Node *next;  // pointer to the same struct type
} Node;

typedef struct {
    Node *head;
    int size;
} LinkedList;

LinkedList *list_create() {
    LinkedList *list = malloc(sizeof(LinkedList));
    if (!list) return NULL;
    list->head = NULL;
    list->size = 0;
    return list;
}

void list_push_front(LinkedList *list, int value) {
    Node *node = malloc(sizeof(Node));
    if (!node) return;
    node->value = value;
    node->next = list->head;
    list->head = node;
    list->size++;
}

int list_pop_front(LinkedList *list, int *out) {
    if (!list->head) return 0;   // empty list
    Node *temp = list->head;
    *out = temp->value;
    list->head = temp->next;
    free(temp);
    list->size--;
    return 1;
}

void list_print(const LinkedList *list) {
    for (Node *n = list->head; n != NULL; n = n->next) {
        printf("%d -> ", n->value);
    }
    printf("NULL\n");
}

void list_free(LinkedList *list) {
    Node *current = list->head;
    while (current) {
        Node *temp = current;
        current = current->next;
        free(temp);   // free each node
    }
    free(list);   // free the list struct itself
}

int main(void) {
    LinkedList *list = list_create();
    list_push_front(list, 30);
    list_push_front(list, 20);
    list_push_front(list, 10);
    list_print(list);   // 10 -> 20 -> 30 -> NULL

    int val;
    if (list_pop_front(list, &val)) {
        printf("Popped: %d\n", val);  // 10
    }
    list_print(list);   // 20 -> 30 -> NULL

    list_free(list);    // must free when done
    return 0;
}
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Not checking return values**

```c
// WRONG: ignoring malloc failure
int *arr = malloc(n * sizeof(int));
arr[0] = 42;   // crash if malloc returned NULL

// CORRECT: always check
int *arr = malloc(n * sizeof(int));
if (arr == NULL) {
    fprintf(stderr, "Out of memory\n");
    exit(EXIT_FAILURE);
}
```

**Mistake 2: Buffer overflow with strcpy**

```c
char buf[10];
char *user_input = "this is way too long for the buffer";

// WRONG: no bounds check, overwrites adjacent memory
strcpy(buf, user_input);

// CORRECT: use strncpy or snprintf with the buffer size
strncpy(buf, user_input, sizeof(buf) - 1);
buf[sizeof(buf) - 1] = '\0';   // strncpy doesn't guarantee null termination!

// OR: use snprintf, which always null-terminates
snprintf(buf, sizeof(buf), "%s", user_input);
```

**Mistake 3: Returning pointers to local variables**

```c
// WRONG: stack frame is gone when function returns — dangling pointer!
int *bad_function() {
    int local = 42;
    return &local;   // UNDEFINED BEHAVIOR — compiler may warn
}

// CORRECT: use heap allocation if data must outlive the function
int *good_function() {
    int *result = malloc(sizeof(int));
    if (result) *result = 42;
    return result;   // caller must free() this
}
```

**Mistake 4: Integer overflow**

```c
int a = 2000000000;
int b = 2000000000;
int c = a + b;   // UNDEFINED BEHAVIOR: signed integer overflow in C!
// Result is implementation-defined; AddressSanitizer won't catch this

// CORRECT: use larger types or check before adding
#include <limits.h>
if (a > INT_MAX - b) { /* handle overflow */ }
long long c_safe = (long long)a + b;  // promotes to 64-bit
```

---

## 5. The "Why Does This Work" Layer

### Why C Doesn't Check Array Bounds

Array bounds checking requires the array's size to be known at runtime and a comparison before every access. C was designed to be as close to the machine as possible — to generate code that is essentially equivalent to hand-written assembly. Adding bounds checks would:
1. Require passing array sizes everywhere
2. Add a comparison + branch to every array access
3. Make C programs noticeably slower

The trade-off was deliberate: C trusts the programmer. Modern tools (AddressSanitizer, Valgrind, fuzzing) compensate for this by detecting violations during testing.

### Why Pointers Enable Efficiency

Passing a struct by value copies every byte. Passing a pointer copies only 8 bytes (the address). For large data structures, this is critical. This is why C functions that modify their inputs take pointers, and why high-performance algorithms pass pointers to avoid copying.

---

## 6. Quick Reference

```bash
# Compilation with all safety checks
gcc -Wall -Wextra -Wpedantic -std=c17 \
    -fsanitize=address,undefined \
    -g -O0 program.c -o program

# Production build
gcc -O2 -std=c17 -DNDEBUG program.c -o program
```

### Common Functions

| Category | Function | Use |
|----------|---------|-----|
| I/O | `printf`, `scanf`, `fgets` | Text I/O |
| Memory | `malloc`, `calloc`, `realloc`, `free` | Heap allocation |
| Strings | `strlen`, `strncpy`, `snprintf`, `strcmp` | String operations |
| Memory | `memset`, `memcpy`, `memmove` | Raw memory operations |
| Math | `abs`, `sqrt`, `pow`, `rand` | Math (link -lm) |
| Files | `fopen`, `fclose`, `fread`, `fwrite` | File I/O |

### Pointer Syntax

```c
int x = 5;
int *p = &x;   // p holds address of x
*p = 10;       // write through pointer (x is now 10)
int **pp = &p; // pointer to pointer
(*pp) = NULL;  // p is now NULL

void *v = p;       // generic pointer
int *q = (int*)v;  // cast to use

int arr[5];
int *a = arr;      // arr decays to pointer to first element
a[2] = 30;         // same as *(a+2) = 30
```
