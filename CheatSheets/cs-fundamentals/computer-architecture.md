# Computer Architecture: A Complete Progressive Tutorial

---

## 1. What & Why

Computer architecture is the study of how computers are designed and built — the organization of processors, memory, caches, buses, and the instruction set that ties them together. It sits at the boundary between software and hardware, defining the contract that lets programmers write code without building circuits.

Why should a software engineer understand computer architecture? Because architecture determines performance. Why is accessing an array in order faster than random access? Cache locality. Why does multiplying by a power of two with a bit shift beat integer multiplication? CPU instruction latency. Why can a modern CPU appear to do several things simultaneously in a single core? Out-of-order execution and instruction pipelining. Understanding these mechanisms lets you write code that works with the hardware rather than against it.

---

## 2. Mental Model

Modern CPUs are organized around a hierarchy of speed versus capacity:

```
Speed    Storage        Size        Latency      Who manages it
────────────────────────────────────────────────────────────────
Fastest  Registers      ~1 KB       ~0.3 ns      Compiler/CPU
         L1 Cache       32-64 KB    ~1 ns        CPU (hardware)
         L2 Cache       256 KB-1MB  ~4 ns        CPU (hardware)
         L3 Cache       8-32 MB     ~10-40 ns    CPU (hardware)
         RAM            4-64 GB     ~80-120 ns   OS + hardware
Slowest  SSD/NVMe       1-8 TB      ~100,000 ns  OS + filesystem

Key insight: RAM is 100x slower than L1 cache.
A cache miss costs as much as hundreds of arithmetic instructions.
The single most impactful optimization is often: improve cache locality.
```

The CPU's job is to execute the Instruction Set Architecture (ISA) — the formal specification of what instructions exist and what they do. The microarchitecture is how the ISA is physically implemented in silicon: how many instructions can execute per cycle, how deep the pipeline is, how the branch predictor works.

---

## 3. Progressive Examples

### Level 1: The CPU Pipeline

```
INSTRUCTION PIPELINE: modern CPUs process multiple instructions simultaneously
by overlapping their execution stages.

Classic 5-stage pipeline (RISC-V, MIPS, simplified ARM):

Instruction:  ADD  SUB  MUL  AND  OR
              │    │    │    │    │
IF (Fetch)    [ADD] [SUB] [MUL] [AND] [OR]
ID (Decode)        [ADD] [SUB] [MUL] [AND]
EX (Execute)            [ADD] [SUB] [MUL]
MA (Mem Access)              [ADD] [SUB]
WB (Writeback)                    [ADD]

Time ────────────────────────────────────────────►

Each clock cycle, a new instruction enters the pipeline.
At peak: 5 instructions in flight simultaneously.
Ideal throughput: 1 instruction per cycle (IPC = 1)

Modern CPUs go further:
  - Superscalar: multiple pipelines → IPC > 1 (e.g., 4-8 IPC)
  - Out-of-order execution: reorder instructions to avoid stalls
  - Branch prediction: speculatively execute before branch resolves
  - SIMD: one instruction operates on 8/16/32 values simultaneously
```

```c
// Pipeline hazards and how code affects them

// DATA HAZARD: instruction needs result not yet written back
int a = 5;
int b = a + 1;  // must wait for 'a' to be computed
int c = b * 2;  // must wait for 'b' — 3 dependent instructions in series
// CPU can't fully pipeline these — each must wait for previous result
// This is called a "read-after-write" (RAW) hazard

// INDEPENDENT instructions can pipeline fully:
int x = 5, y = 6, z = 7;
int r1 = x + 1;   // independent of r2, r3
int r2 = y + 2;   // independent of r1, r3
int r3 = z + 3;   // independent of r1, r2
// CPU can execute all three in parallel (with multiple execution units)

// CONTROL HAZARD: branch target not known until instruction executes
for (int i = 0; i < n; i++) {
    if (data[i] > threshold) {   // branch prediction needed
        process(data[i]);
    }
}
// Branch predictor learns the pattern. Random data = many mispredictions = slow.
// Sorted data (all < threshold, then all >) = predictable = fast.
// This explains the classic "sorted vs unsorted array" benchmark difference.
```

### Level 2: Memory Hierarchy and Cache Behavior

```c
#include <time.h>
#include <stdio.h>
#include <stdlib.h>

#define N 1024
int matrix[N][N];

// SLOW: column-major traversal — cache-unfriendly
void column_major(int mat[N][N]) {
    for (int col = 0; col < N; col++) {
        for (int row = 0; row < N; row++) {
            mat[row][col] += 1;
            // Each access: mat[0][col], mat[1][col], mat[2][col]...
            // These are N ints apart in memory = N * 4 bytes between accesses
            // With N=1024: 4KB between accesses — thrashes cache constantly
        }
    }
}

// FAST: row-major traversal — cache-friendly
void row_major(int mat[N][N]) {
    for (int row = 0; row < N; row++) {
        for (int col = 0; col < N; col++) {
            mat[row][col] += 1;
            // Each access: mat[row][0], mat[row][1], mat[row][2]...
            // Consecutive in memory — all land in the same cache lines!
        }
    }
}

// WHY THE DIFFERENCE:
// C stores 2D arrays in row-major order:
// mat[0][0], mat[0][1], ..., mat[0][1023], mat[1][0], mat[1][1], ...
// 
// A cache line is typically 64 bytes = 16 integers.
// When you access mat[0][0], the hardware prefetches mat[0][0]..mat[0][15].
// Row-major: the next 15 accesses are already in cache (L1 hit).
// Column-major: the next access (mat[1][0]) is 4096 bytes away — cache miss.
//
// Benchmark result: row_major is typically 5-10x faster.
```

```python
# Demonstrating cache effects in Python
import time
import array

def measure_cache_effects():
    N = 1024
    flat = array.array('i', [0] * (N * N))

    # Sequential access (cache-friendly)
    start = time.perf_counter()
    for i in range(N * N):
        flat[i] += 1
    sequential_time = time.perf_counter() - start

    # Strided access (cache-unfriendly) — access every 64th element
    stride = 64
    start = time.perf_counter()
    for i in range(0, N * N, stride):
        flat[i] += 1
    strided_time = time.perf_counter() - start

    print(f"Sequential: {sequential_time*1000:.2f}ms ({N*N} accesses)")
    print(f"Strided:    {strided_time*1000:.2f}ms ({N*N//stride} accesses, but cache-hostile)")
    # Despite fewer accesses, strided may be slower per-access due to cache misses

measure_cache_effects()
```

### Level 3: SIMD — Data-Level Parallelism

```c
#include <immintrin.h>  // AVX2 intrinsics
#include <stdint.h>

// SCALAR: process one element at a time
void add_arrays_scalar(float *a, float *b, float *c, int n) {
    for (int i = 0; i < n; i++) {
        c[i] = a[i] + b[i];   // 1 addition per loop iteration
    }
}

// SIMD (AVX2): process 8 floats simultaneously
void add_arrays_simd(float *a, float *b, float *c, int n) {
    int i;
    for (i = 0; i <= n - 8; i += 8) {
        __m256 va = _mm256_loadu_ps(&a[i]);   // load 8 floats from a
        __m256 vb = _mm256_loadu_ps(&b[i]);   // load 8 floats from b
        __m256 vc = _mm256_add_ps(va, vb);    // add 8 pairs simultaneously
        _mm256_storeu_ps(&c[i], vc);          // store 8 results
    }
    // Handle remainder
    for (; i < n; i++) {
        c[i] = a[i] + b[i];
    }
}
// AVX2: 256-bit registers = 8 × float32 or 4 × float64 or 32 × int8
// With AVX-512: 512-bit = 16 × float32 simultaneously

// Modern compilers auto-vectorize loops — you often don't write intrinsics directly.
// Help the compiler: use -O3 -march=native, avoid pointer aliasing (use restrict).

// NumPy automatically uses SIMD:
// import numpy as np
// a, b = np.random.rand(1000000), np.random.rand(1000000)
// c = a + b  # uses AVX/AVX2/AVX-512 depending on CPU — ~8x faster than a Python loop
```

### Level 4: Memory, Addressing, and Virtual Memory

```
VIRTUAL MEMORY: every process sees a flat address space (0 to 2^64 on 64-bit)
The MMU (Memory Management Unit) translates virtual → physical addresses.

Virtual address space (x86-64 Linux, simplified):
  0x0000000000000000    NULL (unmapped — SIGSEGV on access)
  0x0000000000400000    Program code (.text section)
  0x0000000000600000    Initialized data (.data)
  0x0000000000601000    Uninitialized data (.bss, zero-filled)
  ...
  [heap grows up from program break]
  ...
  0x00007fffffffffff    Stack (grows down)
  0xFFFF800000000000    Kernel space (inaccessible)

ADDRESS TRANSLATION (TLB → Page Table walk):
  Virtual address: [47:12] page number  | [11:0] byte offset
  
  1. CPU checks TLB (Translation Lookaside Buffer) — hardware cache of recent translations
  2. TLB hit: physical address = TLB[virtual_page] | offset  (fast, ~1 cycle)
  3. TLB miss: CPU walks the 4-level page table in memory  (slow, ~4 memory accesses)
  
  Page table walk: CR3 register → PML4 → PDPT → PD → PT → physical frame
  Each level: one memory read → 4 reads total → potential 4 cache misses

PAGE SIZE: 4KB (standard), 2MB (huge pages), 1GB (giant pages)
  Huge pages: fewer TLB entries needed, fewer TLB misses
  Linux: /proc/sys/vm/nr_hugepages
  Used by: databases (PostgreSQL, Redis), high-performance networking (DPDK)
```

```python
# Observing virtual memory in Python
import os
import resource

# Check memory mappings of current process
with open(f"/proc/{os.getpid()}/maps") as f:
    for line in f.readlines()[:20]:
        print(line.strip())
# Output shows: address range, permissions, offset, device, inode, path
# 7f8b2c000000-7f8b2c001000 r-xp 00000000 08:01 1234 /lib/x86_64-linux-gnu/libc.so.6

# Memory usage of current process
with open(f"/proc/{os.getpid()}/status") as f:
    for line in f:
        if line.startswith(("VmRSS", "VmVirt", "VmPeak")):
            print(line.strip())
# VmRSS: actual physical memory used (resident set size)
# VmVirt: total virtual address space (including unused pages)

# Page faults
rusage = resource.getrusage(resource.RUSAGE_SELF)
print(f"Minor faults: {rusage.ru_minflt}")   # page found in RAM (no disk I/O)
print(f"Major faults: {rusage.ru_majflt}")   # page loaded from disk (slow!)
```

### Level 5: CPU Microarchitecture Features

```
BRANCH PREDICTION:
Every conditional jump in your code is a guess for the CPU.
The branch predictor tries to predict which way the branch will go
before the condition is actually computed.

Why: a 15-stage pipeline that mispredicts a branch must flush 15 stages → 15 wasted cycles.
With 3GHz CPU: 15 cycles = 5 nanoseconds. At 10M branches/sec: 50ms wasted per second!

Modern branch predictors: >99% accuracy on regular code.
Worst case: random data in a loop condition = ~50% accuracy = severe stall.

OUT-OF-ORDER EXECUTION:
The CPU reorders instructions to avoid stalls.
If instruction 3 depends on instruction 1 but instruction 2 is independent:
  Normal:    1 → 2 → 3  (2 waits for 1)
  Out-of-order: 1,2 execute simultaneously → 3 follows

The instruction window: modern CPUs can look 100-300 instructions ahead
to find independent operations to execute.

SPECULATIVE EXECUTION:
CPU executes code speculatively before knowing if it's needed.
If the speculation is correct: no penalty.
If wrong: discard speculative results, restore state.

Security implication: Spectre/Meltdown exploits timing side-channels
from speculative execution to leak memory from other processes.
The fix: flush-on-branch, IBRS, retpolines — all add overhead.

HYPERTHREADING / SMT (Simultaneous Multithreading):
One physical core appears as two logical cores to the OS.
Two threads share the execution units of one physical core.
If one thread stalls (cache miss), the other can use the execution units.
Benefit: 10-30% throughput increase. Not useful when threads are compute-bound.
```

```c
// Measuring CPU frequencies and instruction throughput
// Use perf on Linux for hardware performance counters

/*
perf stat ./program
    Performance counter stats for './program':
         1,234 cache-misses        # 2.34% of all cache refs
         52,789 cache-references
         3,827 cycles
         4,123 instructions         # 1.08 insn per cycle (IPC)
         12 branch-misses           # 0.01% of all branches
*/

// For performance-critical code, optimize for:
// 1. IPC close to max for your CPU (4-6 on modern x86)
// 2. Low cache miss rate (< 1% for L1, < 5% for L2)
// 3. Low branch mispredict rate (< 0.1% for predictable code)
// 4. High memory bandwidth utilization (for memory-bound code)
```

### Level 6: Modern CPU Considerations for Software Engineers

```c
// False sharing: two threads modify different data in the same cache line
#include <pthread.h>
#include <stdint.h>

// BAD: counters on adjacent addresses → same cache line → false sharing
typedef struct {
    int counter_a;   // Thread A modifies this
    int counter_b;   // Thread B modifies this
    // Both fit in one 64-byte cache line!
} SharedCounters_Bad;

// GOOD: pad to separate cache lines (64 bytes apart)
typedef struct {
    int counter_a;
    char padding_a[60];    // 60 + 4 = 64 bytes (one cache line)
    int counter_b;
    char padding_b[60];
} SharedCounters_Good;
// Now thread A and thread B work on different cache lines
// No ping-ponging the cache line between cores → ~10x faster

// Memory ordering (atomics and barriers)
#include <stdatomic.h>

atomic_int flag = ATOMIC_VAR_INIT(0);
int data = 0;

// Thread 1:
void producer() {
    data = 42;
    atomic_store_explicit(&flag, 1, memory_order_release);
    // release: all writes before this point are visible before flag=1 is seen
}

// Thread 2:
void consumer() {
    while (!atomic_load_explicit(&flag, memory_order_acquire));
    // acquire: all writes before the store (data=42) are visible here
    printf("%d\n", data);   // guaranteed to print 42
}
// Without the acquire/release semantics: CPU/compiler reordering could print 0!
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Assuming RAM access is fast**

RAM access takes 80-120 nanoseconds. L1 cache takes ~1 nanosecond. At 3GHz, 80ns is 240 clock cycles. A loop with cache misses every iteration is ~240 cycles per iteration instead of ~3 cycles — an 80x slowdown that no algorithmic optimization can overcome. Always think about cache behavior when writing performance-critical code.

**Mistake 2: Thinking branch prediction doesn't matter**

```c
// Slow: random data, 50% prediction accuracy, constant mispredictions
for (int i = 0; i < n; i++) {
    if (random_data[i] > 128) sum += random_data[i];
}

// Fast: the same operation, but branchless (no branch to mispredct)
for (int i = 0; i < n; i++) {
    sum += (random_data[i] > 128) ? random_data[i] : 0;
    // A good compiler generates a conditional move (CMOV) — no branch!
}
```

**Mistake 3: Not understanding that SIMD requires data alignment**

Most SIMD load/store instructions perform best with aligned data (address divisible by vector width: 16, 32, or 64 bytes). Unaligned access often works but is slower. Use `aligned_alloc()` or `posix_memalign()` for SIMD buffers. NumPy arrays returned by normal operations are typically 64-byte aligned.

---

## 5. The "Why Does This Work" Layer

### Why the Cache Hierarchy Speeds Up Programs

Temporal locality: data recently accessed is likely to be accessed again soon. Spatial locality: data near recently accessed data is likely to be accessed soon. Caches exploit both. When you access `arr[i]`, the cache loads a whole cache line (64 bytes = 16 ints) into L1. The next 15 accesses to consecutive elements are L1 cache hits at ~1ns each instead of RAM accesses at ~80ns.

### Why Out-of-Order Execution Matters

A CPU's functional units (ALU, FPU, load/store, branch) can operate independently. Out-of-order execution extracts instruction-level parallelism from a sequential stream: if instruction 5 doesn't depend on instruction 4, it can execute in parallel with instruction 4 even though it comes later in the program. The CPU maintains a "reorder buffer" that commits results in-order while allowing out-of-order execution. This is transparent to the programmer — your code appears to execute sequentially even though the CPU is executing instructions in a different order internally.

---

## 6. Quick Reference

### Memory Hierarchy Latencies

| Level | Capacity | Latency | Bandwidth |
|-------|---------|---------|-----------|
| L1 Cache | 32-64 KB | ~1 ns | ~2 TB/s |
| L2 Cache | 256 KB-1MB | ~4 ns | ~1 TB/s |
| L3 Cache | 8-64 MB | ~10-40 ns | ~500 GB/s |
| DRAM | 4-256 GB | ~80-120 ns | ~50 GB/s |
| NVMe SSD | 1-8 TB | ~100,000 ns | ~7 GB/s |

### Cache Optimization Rules

1. Access memory sequentially (row-major for 2D arrays)
2. Keep frequently accessed data in hot structs, cold data separate
3. Pad structs to avoid false sharing between threads
4. Use huge pages for large working sets (databases, ML)
5. Profile cache misses with `perf stat` before optimizing

### SIMD Width by Generation

| Extension | Width | Float32 | Float64 | int8 |
|-----------|-------|---------|---------|------|
| SSE2 | 128-bit | 4 | 2 | 16 |
| AVX | 256-bit | 8 | 4 | 32 |
| AVX2 | 256-bit | 8 | 4 | 32 (integers) |
| AVX-512 | 512-bit | 16 | 8 | 64 |
