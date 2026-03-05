# The Complete Operating Systems Cheat Sheet
### From Zero to Expert

---

## 1. What is an Operating System?

An OS is the software layer between hardware and user applications. It manages resources and provides abstractions that make programming possible.

### Core Responsibilities

```
┌─────────────────────────────────────────┐
│         User Applications               │
├─────────────────────────────────────────┤
│      System Libraries (libc, etc.)      │
├─────────────────────────────────────────┤
│         System Call Interface           │
├──────────────┬──────────────────────────┤
│   Process    │  Memory  │  File  │  I/O │
│  Management  │  Manager │ System │  Mgr │
├──────────────┴──────────┴────────┴──────┤
│            Hardware Abstraction         │
└─────────────────────────────────────────┘
│         CPU  │  RAM  │  Disk  │  Net   │
└─────────────────────────────────────────┘
```

### Kernel vs User Space

```
Two CPU privilege levels (rings in x86: ring 0 and ring 3):

KERNEL SPACE (ring 0 — privileged):
  • Full hardware access
  • Can execute any instruction
  • Manages memory, devices, processes
  • A bug here crashes the whole system

USER SPACE (ring 3 — restricted):
  • No direct hardware access
  • Cannot execute privileged instructions
  • Isolated: a crash only kills that process
  • Must ask the kernel via system calls

Crossing the boundary — System Call:
  User process executes SYSCALL instruction
  → CPU switches to ring 0, jumps to kernel syscall handler
  → Kernel performs the operation
  → CPU switches back to ring 3, returns result
  Cost: ~100–1000 ns (context switch overhead)
```

### Kernel Architectures

```
Monolithic Kernel (Linux, BSD):
  All OS services run in kernel space.
  + Fast (no inter-process communication between components)
  - A driver bug can crash the system
  - Large, complex codebase

Microkernel (Mach, QNX, MINIX 3):
  Only bare minimum in kernel: IPC, scheduling, basic memory.
  File systems, drivers, network stack run as user-space servers.
  + Fault isolation (a driver crash doesn't crash the kernel)
  - Slower (every service requires IPC message passing)

Hybrid (Windows NT, macOS XNU):
  Pragmatic mix: some services in kernel for speed, some outside for isolation.

Exokernel (research):
  Kernel does only protection; applications manage their own resources directly.
  Maximum performance; minimal abstraction.
```

### Key System Calls (POSIX)

```
Process:    fork(), exec(), exit(), wait(), getpid(), kill()
Memory:     mmap(), munmap(), brk(), mprotect()
File:       open(), read(), write(), close(), lseek(), stat(), unlink()
Directory:  mkdir(), rmdir(), opendir(), readdir()
IPC:        pipe(), socket(), shmget(), semget(), msgget()
Sync:       futex(), pthread_mutex_*
Thread:     clone() (Linux), pthread_create()
Signal:     signal(), sigaction(), raise(), pause()
```

---

## 2. Processes

A **process** is a running program — an instance of an executable with its own address space, resources, and execution state.

### Process vs Program

```
Program:  static file on disk (instructions + data)
Process:  dynamic execution of a program
          = program code + current state (PC, registers, memory, open files)

Same program → multiple processes possible (e.g., two terminal windows running bash)
```

### Process Address Space

```
High address ┌─────────────────┐
             │   Kernel Space  │  (mapped into every process but inaccessible)
             ├─────────────────┤ 0xFFFFFFFF... (top of user space)
             │      Stack      │  local variables, function frames
             │   (grows ↓)     │  limited size (typically 8 MB default)
             │                 │
             │    (gap)        │  unmapped — stack overflow → segfault
             │                 │
             │   (grows ↑)     │
             │      Heap       │  malloc/new allocations
             ├─────────────────┤
             │  BSS Segment    │  uninitialized global/static variables (zeroed)
             ├─────────────────┤
             │  Data Segment   │  initialized global/static variables
             ├─────────────────┤
             │  Text Segment   │  program code (read-only, executable)
Low address  └─────────────────┘ 0x400000 (typical)
```

### Process States

```
            fork()
              │
              ▼
           ┌──────┐
           │ NEW  │
           └──┬───┘
              │ admitted
              ▼
  ┌──────────────────────┐
  │        READY         │◄──────────────────────┐
  └──────────┬───────────┘                       │
             │ scheduled (dispatch)               │ I/O complete /
             ▼                                    │ event occurs
  ┌──────────────────────┐                       │
  │       RUNNING        │──── I/O request ────►┌┴──────────────┐
  └──────────┬───────────┘     or event wait     │    WAITING    │
             │                                   └───────────────┘
             │ exit()
             ▼
         ┌────────┐
         │ ZOMBIE │ (exit status not yet collected by parent)
         └────────┘
              │ parent calls wait()
              ▼
         ┌────────┐
         │  DEAD  │
         └────────┘

Preemption: OS can move RUNNING → READY at any time (e.g., timer interrupt)
```

### Process Control Block (PCB)

The kernel's data structure representing a process:

```
PCB contains:
  Process ID (PID), Parent PID (PPID)
  Process state (ready, running, waiting, zombie)
  CPU registers (saved on context switch)
  Program counter
  Memory management info (page table pointer)
  Open file descriptors
  Signal handlers
  Scheduling info (priority, CPU time used)
  Accounting info (creation time, CPU time)
  I/O status info
```

### fork() and exec()

```
fork():
  Creates an exact copy of the calling process.
  Returns: child PID to parent, 0 to child, -1 on error.
  Copy-on-Write: pages are shared until one process writes → then copied.

  pid_t pid = fork();
  if (pid == 0) {
      // Child process
  } else if (pid > 0) {
      // Parent process
      wait(NULL);  // wait for child to finish
  }

exec():
  Replaces the current process image with a new program.
  Does NOT create a new process — same PID, new code/data/stack.
  On success, exec() does not return.

  execl("/bin/ls", "ls", "-la", NULL);

Fork-then-exec: the Unix way to launch new programs
  Shell: fork() → (child) exec("cmd") → (parent) wait()
```

### Inter-Process Communication (IPC)

```
Pipes (anonymous):
  Unidirectional byte stream between related processes (parent-child).
  int fds[2]; pipe(fds);  // fds[0]=read end, fds[1]=write end
  Kernel buffer: typically 64 KB. Blocks when full/empty.

Named Pipes (FIFOs):
  Like pipes but with a file system name → unrelated processes can use them.
  mkfifo("/tmp/myfifo", 0666);

Signals:
  Asynchronous notifications sent to a process.
  Common signals: SIGINT(2)=Ctrl+C, SIGKILL(9)=force kill, SIGTERM(15)=graceful kill,
                  SIGSEGV(11)=segfault, SIGCHLD(17)=child stopped, SIGUSR1/2=user defined
  kill(pid, SIGTERM);  // send signal
  signal(SIGINT, handler);  // register handler (use sigaction() in production)
  SIGKILL and SIGSTOP cannot be caught or ignored.

Shared Memory (fastest IPC):
  Map same physical pages into multiple process address spaces.
  shmget() + shmat()  (System V)
  mmap(MAP_SHARED)    (POSIX — preferred)
  Must use semaphores/mutexes for synchronization.

Message Queues:
  Kernel-maintained queue; processes send/receive typed messages.
  msgget(), msgsnd(), msgrcv()

Sockets:
  Can be local (Unix domain sockets) or networked (TCP/UDP).
  Most general IPC mechanism.
  AF_UNIX sockets are ~10× faster than TCP loopback.
```

---

## 3. Threads & Concurrency

A **thread** is the unit of execution within a process. All threads in a process share the same address space.

### Thread vs Process

```
                Process A          Process B
               ┌──────────┐       ┌──────────┐
               │ Heap     │       │ Heap     │  ← Separate
               │ Code     │       │ Code     │  ← Separate
               │ Files    │       │ Files    │  ← Separate
               ├──────────┤       └──────────┘
               │ Thread 1 │  Thread 2 │ Thread 3
               │  Stack   │  Stack    │ Stack   ← Per-thread
               │  Regs    │  Regs     │ Regs    ← Per-thread
               │  PC      │  PC       │ PC      ← Per-thread
               └──────────┴───────────┴─────────

Threads share: heap, code, globals, open files, signal handlers
Threads own:   stack, registers, PC, thread-local storage (TLS), errno

Creating threads:                      vs creating processes:
  pthread_create(): ~10 µs                fork(): ~100 µs
  Context switch:   ~1 µs                 Context switch: ~10 µs
  Memory:           ~8 MB stack/thread    Memory: full copy (CoW)
```

### Thread Models

```
User-Level Threads (M:1):
  Thread library manages N user threads on 1 kernel thread.
  + Fast creation and context switch (no syscall)
  - One thread blocks → entire process blocks (kernel sees 1 thread)
  Examples: early Java green threads, goroutines (partially)

Kernel-Level Threads (1:1):
  Each user thread = one kernel thread.
  + Blocking one thread doesn't block others
  + True parallelism on multi-core
  - More expensive (syscall for every operation)
  Examples: Linux pthreads (NPTL), Windows threads

Hybrid (M:N):
  M user threads mapped to N kernel threads (M >= N).
  + Best of both worlds (in theory)
  - Complex scheduler required
  Examples: Solaris, Go goroutines, Erlang processes

Linux implementation:
  clone() syscall creates threads (same as fork but with shared address space flag)
  Linux has no distinction between processes and threads internally (both are "tasks")
```

### POSIX Threads (pthreads)

```c
#include <pthread.h>

// Create thread
pthread_t tid;
pthread_create(&tid, NULL, thread_function, (void*)arg);

// Wait for thread to finish
pthread_join(tid, &return_value);

// Detach (don't need return value, auto-cleanup on exit)
pthread_detach(tid);

// Thread function signature
void* thread_function(void* arg) {
    int* n = (int*)arg;
    // ... do work ...
    return NULL;
}

// Thread-local storage
__thread int tls_var;  // each thread gets its own copy
pthread_key_t key;
pthread_key_create(&key, destructor);
pthread_setspecific(key, value);
pthread_getspecific(key);
```

---

## 4. Synchronization

Concurrent access to shared data causes **race conditions**. Synchronization prevents them.

### The Race Condition Problem

```c
// Global counter, incremented by two threads:
int counter = 0;

// Thread 1:                  Thread 2:
counter++;                     counter++;
// Compiles to:                // Compiles to:
// LOAD  r1, counter           // LOAD  r1, counter
// ADD   r1, 1                 // ADD   r1, 1
// STORE counter, r1           // STORE counter, r1

// Interleaving that loses an increment:
// T1: LOAD  r1 = 0
// T2: LOAD  r1 = 0
// T1: ADD   r1 = 1
// T2: ADD   r1 = 1
// T1: STORE counter = 1
// T2: STORE counter = 1   ← Should be 2, got 1!

Critical section: the code that accesses shared data
Mutual exclusion: only one thread in a critical section at a time
```

### Mutex (Mutual Exclusion Lock)

```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

pthread_mutex_lock(&lock);   // acquire; blocks if already held
    // --- critical section ---
    counter++;
    // --- end critical section ---
pthread_mutex_unlock(&lock); // release

// Try-lock (non-blocking):
if (pthread_mutex_trylock(&lock) == 0) {
    // got the lock
    pthread_mutex_unlock(&lock);
}

// Timed lock:
struct timespec ts = { .tv_sec = time(NULL) + 1 };
pthread_mutex_timedlock(&lock, &ts);

Properties:
  • Only the thread that locked can unlock (ownership)
  • Recursive mutexes: same thread can lock multiple times (must unlock same times)
  • Non-recursive: second lock by same thread → deadlock
```

### Spinlock

```c
// Busy-waits instead of blocking:
pthread_spinlock_t spin;
pthread_spin_init(&spin, 0);
pthread_spin_lock(&spin);
    // critical section — must be VERY short
pthread_spin_unlock(&spin);

// When to use spinlocks vs mutexes:
// Spinlock:  critical section < ~100 cycles, multicore, no sleeping allowed (interrupt handlers)
// Mutex:     critical section > ~100 cycles, single-core possible, can sleep
```

### Semaphore

```c
// Generalized synchronization primitive with a counter
#include <semaphore.h>

sem_t sem;
sem_init(&sem, 0, initial_value);  // 0 = thread-shared

sem_wait(&sem);   // P() — decrement; block if 0
    // critical section
sem_post(&sem);   // V() — increment; wake one waiter

// Binary semaphore (value 0 or 1) ≈ mutex
// Counting semaphore: limits concurrent access to N (e.g., connection pool of 10)

// Key difference from mutex: any thread can post (release)
// Use case: signaling between threads (producer-consumer)

sem_t full, empty;
sem_init(&full, 0, 0);       // items available
sem_init(&empty, 0, BUF_SIZE); // empty slots

// Producer:
sem_wait(&empty);
buffer[in] = item; in = (in+1) % BUF_SIZE;
sem_post(&full);

// Consumer:
sem_wait(&full);
item = buffer[out]; out = (out+1) % BUF_SIZE;
sem_post(&empty);
```

### Condition Variable

```c
// Waiting for a condition to become true without busy-waiting
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

// Thread waiting for condition:
pthread_mutex_lock(&mutex);
while (!condition_is_true) {        // ALWAYS use while, not if (spurious wakeups)
    pthread_cond_wait(&cond, &mutex); // atomically release mutex and sleep
}
// condition is now true, mutex is held again
pthread_mutex_unlock(&mutex);

// Thread signaling condition:
pthread_mutex_lock(&mutex);
condition_is_true = 1;
pthread_cond_signal(&cond);    // wake ONE waiting thread
// or: pthread_cond_broadcast(&cond);  // wake ALL waiting threads
pthread_mutex_unlock(&mutex);

// Why while loop? Spurious wakeups: pthread_cond_wait can return without signal
// Also: another thread may consume the condition between wakeup and mutex reacquire
```

### Read-Write Lock

```c
// Multiple readers OR one writer at a time
pthread_rwlock_t rwlock = PTHREAD_RWLOCK_INITIALIZER;

// Reading (many concurrent readers OK):
pthread_rwlock_rdlock(&rwlock);
    // read shared data
pthread_rwlock_unlock(&rwlock);

// Writing (exclusive access):
pthread_rwlock_wrlock(&rwlock);
    // modify shared data
pthread_rwlock_unlock(&rwlock);

// Good for: read-heavy workloads (caches, config, lookup tables)
// Bad for:  write-heavy workloads (overhead > mutex)
```

### Atomic Operations

```c
// Hardware-supported indivisible operations — no lock needed
#include <stdatomic.h>

atomic_int counter = 0;
atomic_fetch_add(&counter, 1);   // atomic increment
atomic_load(&counter);           // atomic read
atomic_store(&counter, 5);       // atomic write
atomic_compare_exchange_strong(&counter, &expected, desired); // CAS

// Memory ordering (from weakest to strongest):
memory_order_relaxed   // no ordering guarantees, just atomicity
memory_order_acquire   // no reads/writes can be reordered before this load
memory_order_release   // no reads/writes can be reordered after this store
memory_order_seq_cst   // total sequential consistency (default, safest, slowest)

// Use atomics for: counters, flags, lock-free data structures
// CAS (Compare-And-Swap) is the foundation of all lock-free algorithms
```

### Classic Synchronization Problems

```
Producer-Consumer (Bounded Buffer):
  Producers add items to buffer; consumers remove them.
  Solution: semaphores (full/empty) + mutex for buffer access.

Readers-Writers:
  Multiple readers can read simultaneously; writers need exclusive access.
  Solution: rwlock, or semaphore + counter.
  Variants: reader-preference, writer-preference, fair.

Dining Philosophers (5 philosophers, 5 forks):
  Each needs two forks to eat; forks are shared.
  Naive solution: deadlock if all grab left fork simultaneously.
  Solutions:
    • Resource ordering: always grab lower-numbered fork first
    • Arbitrator: a waiter grants permission to pick up both forks
    • Chandy-Misra: message-passing solution

Sleeping Barber:
  Barber sleeps when no customers; customers leave if waiting room is full.
  Solution: semaphores for barber state + mutex for waiting count.
```

### Memory Model & Reordering

```
Compilers and CPUs reorder instructions for performance.
This breaks assumptions about shared memory in concurrent code.

Example (x86, two threads):
  Thread 1:          Thread 2:
  x = 1;             y = 1;
  print(y);          print(x);

Expected impossible outcome "0 0" can happen due to:
  • Compiler reordering (moved for optimization)
  • CPU store buffers (write not yet visible to other CPU)
  • Cache coherency delay

Fixes:
  • Memory barriers/fences:  __sync_synchronize(), mfence, atomic_thread_fence()
  • Volatile (C/C++):        prevents compiler reordering (not CPU reordering)
  • Atomic operations:       include implicit memory fences
  • Mutex lock/unlock:       always include full barriers

Memory models by strictness:
  x86/x64:   TSO (Total Store Order) — very strict, few surprises
  ARM/RISC-V: weak memory model — needs explicit barriers more often
  Java/C++:  language-level memory model specifies what compilers/hardware must guarantee
```

---

## 5. CPU Scheduling

The scheduler decides which process/thread runs on which CPU core and for how long.

### Scheduling Goals (Often Conflicting)

```
Throughput:      maximize completed jobs per unit time
Turnaround time: minimize time from job submission to completion
Response time:   minimize time from request to first response (interactive)
Fairness:        each process gets a fair share of CPU
CPU utilization: keep CPU busy (avoid idle time)
Deadlines:       meet timing requirements (real-time systems)
```

### Scheduling Metrics

```
Arrival time (A):    when job enters ready queue
Burst time (B):      how long the job needs CPU (total or one burst)
Completion time (C): when job finishes
Turnaround time:     C - A
Waiting time:        turnaround - B = time spent waiting in queue
Response time:       time from arrival to first CPU allocation
```

### Scheduling Algorithms

#### First Come First Served (FCFS)

```
Non-preemptive. Run jobs in arrival order.
+ Simple
- Convoy effect: short jobs wait behind long ones

Jobs: A(burst=8), B(burst=4), C(burst=1) all arrive at t=0
Gantt: [A:0-8][B:8-12][C:12-13]
Avg turnaround: (8 + 12 + 13) / 3 = 11
```

#### Shortest Job First (SJF) / Shortest Job Next (SJN)

```
Non-preemptive. Run the job with shortest burst time.
+ Optimal average turnaround (provable)
- Starvation: long jobs may never run if short jobs keep arriving
- Requires knowing burst time in advance (usually estimated)

Same jobs as above (optimal order: C, B, A):
Gantt: [C:0-1][B:1-5][A:5-13]
Avg turnaround: (1 + 5 + 13) / 3 = 6.33  ← much better!
```

#### Shortest Remaining Time First (SRTF)

```
Preemptive version of SJF.
When a new job arrives, preempt if its burst < remaining time of current job.
+ Optimal average turnaround among all preemptive policies
- Starvation of long jobs, high context switch overhead
```

#### Round Robin (RR)

```
Preemptive. Each process gets a time quantum (slice), then preempted.
+ Fair, good response time for interactive workloads
- Higher context switch overhead; turnaround worse than SJF

Time quantum Q = 4, same 3 jobs:
Gantt: [A:0-4][B:4-8][C:8-9][A:9-13]

Quantum too small → too many context switches (overhead dominates)
Quantum too large → degenerates to FCFS
Typical quantum: 10–100 ms
```

#### Priority Scheduling

```
Each process has a priority; highest priority runs first.
Can be preemptive or non-preemptive.
+ Allows important tasks to run first
- Starvation of low-priority processes
- Solution: aging — gradually increase priority of waiting processes

Static priority: fixed at creation
Dynamic priority: changes based on behavior (e.g., I/O-bound processes get boosted)
```

#### Multilevel Feedback Queue (MLFQ) — Most Practical

```
Multiple queues with different priorities and quantum sizes.
Rules:
  1. Higher priority queue runs first
  2. Within a queue, use Round Robin
  3. New jobs start at highest priority
  4. If a job uses its full quantum, demote to next lower queue
  5. If a job yields CPU (I/O), stay at current priority (or boost)
  6. Periodically boost all jobs back to top priority (prevents starvation)

Queue 0 (highest): Q=8ms   ← interactive, short jobs stay here
Queue 1:           Q=16ms
Queue 2 (lowest):  Q=∞ (FCFS) ← long batch jobs end up here

Effect: automatically identifies and favors interactive (I/O-bound) processes.
This is what Linux and Windows approximate with their schedulers.
```

#### Linux Completely Fair Scheduler (CFS)

```
Default Linux scheduler (since 2.6.23).
Goal: every runnable task gets an equal share of CPU time.

Key concept: virtual runtime (vruntime)
  vruntime = actual_runtime × (default_weight / task_weight)
  Tasks with lower vruntime run next (they've received less CPU time).
  Lower priority → higher weight divisor → vruntime accumulates faster → less CPU

Data structure: red-black tree ordered by vruntime
  O(log n) to pick next task

Timeslice: dynamically calculated
  timeslice = sched_latency / n_tasks   (sched_latency ≈ 48ms by default)
  Minimum granularity: 1ms (prevents too many context switches)

Nice values: -20 (highest priority) to +19 (lowest)
  Each nice level changes weight by ~10%
```

#### Real-Time Scheduling

```
Hard real-time: missing a deadline = system failure (pacemaker, ABS brakes)
Soft real-time: missing a deadline = degraded quality (video playback)

FIFO (SCHED_FIFO in Linux):
  Fixed priority, run until yield or block. No time slicing within priority.

Rate-Monotonic (RM):
  Static priorities; higher frequency tasks get higher priority.
  Schedulable if: sum(Ci/Ti) ≤ n(2^(1/n) - 1)  (utilization bound ≈ 69% for n→∞)

Earliest Deadline First (EDF):
  Dynamic priorities; task with earliest absolute deadline runs first.
  Optimal: can schedule any feasible task set with utilization ≤ 100%
```

### Context Switch

```
What the OS saves/restores when switching between processes:
  CPU registers (all general-purpose, PC, SP, FLAGS)
  FPU/SIMD state (if used)
  Memory management registers (page table base: CR3 in x86)
  Kernel stack pointer

Cost:
  Direct: ~1–10 µs (save/restore state)
  Indirect: TLB flush (if no ASID), cache pollution
  Total effective cost: can be 10–100 µs including cache effects

Voluntary vs involuntary:
  Voluntary:   process yields, does I/O, or waits on lock
  Involuntary: timer interrupt fires (preemption)
```

---

## 6. Memory Management

### Physical vs Virtual Memory

```
Physical memory: actual RAM chips (e.g., 16 GB)
Virtual memory:  each process's private address space (e.g., 128 TB on 64-bit)

MMU (Memory Management Unit): hardware translates virtual → physical
  Every memory access goes through the MMU.
  Translation: page table lookup (with TLB cache for speed).
```

### Paging

```
Memory divided into fixed-size pages (virtual) and frames (physical).
Typical page size: 4 KB (also: 2 MB and 1 GB "huge pages")

Benefits of paging:
  • No external fragmentation (any page fits any frame)
  • Easy to implement shared memory (map same frame into multiple page tables)
  • Easy to implement virtual memory (page not in RAM → swap to disk)

Page table entry (PTE) contains:
  Physical frame number, Present bit, Dirty bit, Accessed bit,
  Read/Write/Execute permissions, User/Kernel bit

Multi-level page tables (x86-64, 4 levels, 48-bit VA):
  Saves memory: only allocate page table pages that are actually needed.
  Full flat page table for 48-bit space: 512 GB!
  With 4-level table: only the pages actually mapped consume memory.

Huge pages (2 MB, 1 GB):
  + Fewer TLB entries needed → fewer TLB misses
  + Fewer page table levels to traverse
  - Internal fragmentation (2 MB page for 100 bytes wastes 2 MB - 100 bytes)
  Use for: large in-memory databases, HPC, when working set >> TLB coverage
```

### Segmentation

```
Memory divided into variable-size logical segments (code, data, stack, heap).
Each segment has a base address and limit.

x86: uses segments (CS, DS, SS, ES, FS, GS) but mostly ignored in 64-bit mode.
Modern use: segments for per-CPU or per-thread data (FS/GS in Linux for TLS).

Segmentation + Paging:
  Intel used both historically. Modern 64-bit systems: paging only.
```

### Memory Allocation

#### Kernel-Level

```
Physical frame allocation:
  Free list or buddy system.

Buddy System:
  Memory divided into power-of-2 sized blocks.
  To allocate 3 KB: find smallest fitting block (4 KB) → split or use directly.
  To free: merge with "buddy" (adjacent block of same size) if buddy is also free.
  + O(log n) allocation and deallocation
  + Minimal external fragmentation
  - Internal fragmentation (allocating 33 KB requires 64 KB block)

Slab Allocator (Linux):
  For frequently-allocated kernel objects (inodes, PCBs, network buffers).
  Pre-allocates slabs (groups of objects of same size).
  + Zero initialization cost (object constructors run once, destructors reset)
  + Reduced fragmentation for same-size objects
  + Per-CPU caches eliminate lock contention
```

#### User-Level (Heap)

```
malloc() / free() implementation:

Free list: linked list of free blocks.
First fit:  allocate first block that fits       — fast, may fragment
Best fit:   allocate smallest block that fits    — less waste, slower, fragments small blocks
Worst fit:  allocate largest block available     — keeps large blocks available
Next fit:   like first fit but start from last allocation point

Problems:
  External fragmentation: enough total free memory but no single contiguous block
  Internal fragmentation: allocated block larger than requested

Modern allocators (jemalloc, tcmalloc, mimalloc):
  • Size classes: separate free lists per size (reduces fragmentation)
  • Thread-local caches: per-thread arenas (eliminates lock contention for most allocations)
  • Huge page awareness: align large allocations to huge pages
  
mmap() for large allocations (typically > 128 KB):
  OS allocates directly; returned to OS on free (not reused in heap)
```

### Virtual Memory: Paging to Disk

```
When physical memory is full, the OS pages (swaps) frames to disk.

Page fault handler:
  1. Find a free frame (if none, choose victim using replacement policy)
  2. If victim is dirty: write to swap space on disk
  3. Read requested page from disk into frame
  4. Update page table, clear fault
  5. Restart faulting instruction

Page replacement policies:
  OPT (Optimal):  replace page that won't be used for longest time — theoretical baseline
  LRU:            replace least recently used — good but expensive (need timestamps)
  Clock (approx LRU): reference bit + hand pointer — efficient, used in Linux
  NFU (Not Frequently Used): count access frequency
  Working Set:    keep pages in the "working set" (pages used in last τ time units)

Thrashing:
  Process doesn't have enough frames for its working set.
  Most time spent paging rather than executing.
  Detected by: high page fault rate + low CPU utilization.
  Fix: reduce multiprogramming level (suspend some processes) or add RAM.
```

### Memory-Mapped Files (mmap)

```c
// Map a file directly into virtual address space
void* addr = mmap(NULL, file_size,
                  PROT_READ | PROT_WRITE,
                  MAP_SHARED,
                  fd, 0);

// Now access file like a normal array:
char* data = (char*)addr;
data[0] = 'X';  // writes to file!

munmap(addr, file_size);

Benefits:
  + File I/O as simple as memory access
  + Pages loaded on demand (no upfront read)
  + Multiple processes can share the same physical pages
  + OS handles caching (page cache)
  Used for: databases (SQLite, PostgreSQL, LMDB), shared libraries, large files
```

---

## 7. File Systems

A file system organizes persistent data on storage devices.

### File System Concepts

```
File:        named sequence of bytes (abstraction over storage blocks)
Directory:   maps file names to file metadata (inodes/file IDs)
Inode:       metadata structure (size, permissions, timestamps, block pointers)
             NOT the filename — directories hold name→inode mappings
Superblock:  describes the whole filesystem (size, free block count, etc.)
Block:       unit of storage allocation (typically 4 KB)

Hard link:   another directory entry pointing to same inode
             (file deleted only when link count reaches 0)
Soft link:   symlink; stores path string; target can be non-existent
Mount:       attach filesystem to a directory in the namespace
```

### On-Disk Layout (ext4-style)

```
┌────────────────────────────────────────────────────────────┐
│ Boot  │ Super │ Group │ Block  │ Inode  │ Inode │  Data   │
│ block │ Block │ Descr │ Bitmap │ Bitmap │ Table │ Blocks  │
└────────────────────────────────────────────────────────────┘

Block groups: disk divided into groups, each with its own bitmaps + inode table.
  Keeps inodes close to their data blocks (locality).

Inode structure:
  ┌──────────────────────────────────────────┐
  │ mode │ uid │ gid │ size │ times │ links  │
  ├──────────────────────────────────────────┤
  │  12 direct block pointers (0–48 KB)      │
  │  1 single indirect pointer (→ 4 MB)      │
  │  1 double indirect pointer (→ 4 GB)      │
  │  1 triple indirect pointer (→ 4 TB)      │
  └──────────────────────────────────────────┘
  Small files: all data in direct pointers (fast)
  Large files: follow indirection chain
```

### Modern File Systems

```
ext4 (Linux default):
  Journaling: before modifying metadata, write intent to journal.
  If crash mid-operation: replay journal to reach consistent state.
  Extents: contiguous run of blocks (replaces indirect blocks for large files).
  Delayed allocation: buffer writes in memory, allocate blocks when flushing.

XFS (high-performance, large files):
  B+ tree directories (fast for huge directories)
  Allocation groups (parallel access on multi-disk)

Btrfs (Linux):
  Copy-on-Write: writes go to new blocks; old blocks remain until committed.
  Snapshots: nearly free (just copy the B-tree root pointer)
  Built-in RAID, checksums, compression, deduplication

ZFS (Oracle/FreeBSD):
  All-or-nothing writes (transactional)
  End-to-end checksums on all data AND metadata
  RAID-Z, snapshots, clones, deduplication

NTFS (Windows):
  Master File Table (MFT) — database of all files
  Journaling (NTFS log)
  Alternate data streams

FAT32 / exFAT:
  Simple; no permissions, no journaling
  Universal compatibility (flash drives, SD cards)
  exFAT: supports > 4 GB files (FAT32 limit)

APFS (Apple):
  CoW, space sharing between volumes, snapshots
  Strong encryption (per-file or per-volume)
  Optimized for SSDs and flash
```

### Journaling

```
Problem: crash mid-write leaves filesystem inconsistent.
  Creating a file requires: update inode, update directory, update bitmaps
  If crash after first write: data structure corrupted
  
Without journal: fsck scans entire disk to find/fix inconsistencies (~hours)

With journal: write-ahead log
  Before modifying structures, write the intended changes to journal.
  On crash: replay the journal (seconds to recover).
  
Journaling modes:
  writeback:  only journal metadata (fastest, possible data loss)
  ordered:    journal metadata; data written before metadata commit (default ext4)
  full/data:  journal both data and metadata (slowest, safest)
```

### Directory Structures

```
Linear list:  simple, slow for large directories O(n)
Hash table:   O(1) lookup, collision handling needed
B-tree:       O(log n) ordered traversal + lookup (XFS, NTFS, HFS+)
              Good for directories with millions of files

Path resolution: /usr/local/bin/python
  1. Start at root inode (inode 2 always)
  2. Look up "usr" in root directory → inode X
  3. Look up "local" in inode X → inode Y
  4. Look up "bin" in inode Y → inode Z
  5. Look up "python" in inode Z → inode of python binary
  6. Each step: disk read for directory block + inode
  Cost: 2 × path_components disk reads (without caching)
  With dentry cache: nearly free for hot paths
```

### VFS — Virtual File System

```
OS abstraction layer — same system calls work on any filesystem:

Application: open("/etc/passwd", O_RDONLY)
    ↓
VFS Layer: find mount, call filesystem's open() method
    ↓
ext4 / XFS / NFS / procfs / tmpfs ...

VFS objects:
  superblock:  represents mounted filesystem
  inode:       represents a file (in memory version)
  dentry:      directory entry (name → inode mapping, cached)
  file:        represents open file (per-process)
```

### File Descriptors & Open File Table

```
Per-process file descriptor table:
  fd 0 → stdin  (read)
  fd 1 → stdout (write)
  fd 2 → stderr (write)
  fd 3 → ... next open file

System-wide open file table entry contains:
  File offset (current read/write position)
  Access mode (read/write/append)
  Reference count
  Pointer to inode

After fork(): child inherits file descriptors, sharing the SAME open file table entry
  → parent and child share the file offset!

After dup2(fd, 1): redirect stdout to a file
  → both fd and 1 point to same open file table entry

File permissions (Unix):
  ls -la shows:   -rwxr-xr-x  1  user  group  size  date  name
                   │└┬┘└┬┘└┬┘
                   │ owner  group  others
                   │
                   └─ type: - = file, d = dir, l = symlink, c/b = device

  rwx = read(4) + write(2) + execute(1)
  chmod 755 = owner:rwx  group:r-x  others:r-x
  SUID bit (4xxx): execute with owner's permissions (sudo mechanism)
  Sticky bit (1xxx on dir): only owner can delete their files (e.g., /tmp)
```

---

## 8. I/O & Device Management

### I/O Models

```
1. Blocking I/O (synchronous):
   Process issues read(), blocks until data is ready.
   Simple; CPU is idle while waiting.
   
   [Process] ──read()──► [kernel] ──► [device, waiting...] ──► [data ready] ──► [return]

2. Non-blocking I/O:
   read() returns immediately with EAGAIN if no data ready.
   Process must poll repeatedly.
   Wastes CPU if polling too fast; adds latency if polling too slow.
   fcntl(fd, F_SETFL, O_NONBLOCK);

3. I/O Multiplexing (select/poll/epoll):
   Monitor multiple file descriptors; block until one is ready.
   
   select():  O(n) scan, limited to FD_SETSIZE (1024) fds — legacy
   poll():    O(n) scan, no fd limit — improved
   epoll():   O(1) notification, Linux-specific, scales to millions of fds
              Used by: nginx, Node.js, Redis, every high-performance server
   
   // epoll example:
   int epfd = epoll_create1(0);
   struct epoll_event ev = { .events = EPOLLIN, .data.fd = sock_fd };
   epoll_ctl(epfd, EPOLL_CTL_ADD, sock_fd, &ev);
   int n = epoll_wait(epfd, events, MAX_EVENTS, timeout_ms);

4. Signal-driven I/O (SIGIO):
   Kernel sends signal when fd becomes ready.
   Rarely used in practice.

5. Asynchronous I/O (AIO):
   Submit I/O operation; get notified when complete (callback/event).
   True async: process continues while I/O runs.
   Linux: io_uring (modern), aio (limited)
   Windows: IOCP (I/O Completion Ports)
   
   io_uring (Linux 5.1+):
     Ring buffers shared between kernel and userspace.
     Batch submissions, zero-copy, no syscall per I/O in fast path.
     60–100% faster than epoll for high I/O rates.
```

### Device Drivers

```
Character devices: byte-stream interface (keyboard, serial, /dev/random)
Block devices:     random access in fixed-size blocks (disks, SSDs)
Network devices:   packet-oriented interface (eth0, wlan0)

Driver in kernel:
  Registers with kernel via device major/minor numbers.
  Implements VFS-like operations: open, read, write, ioctl, mmap
  
  /dev/sda     → block device (disk)
  /dev/tty     → character device (terminal)
  /dev/null    → character device (discards all writes, EOF on read)
  /dev/zero    → character device (infinite zeros)
  /dev/urandom → character device (random bytes)

ioctl(): escape hatch for device-specific commands not covered by read/write
  ioctl(fd, TIOCGWINSZ, &ws);  // get terminal window size
```

### Disk Scheduling

```
Goal: minimize rotational latency and seek time (HDD only; irrelevant for SSD).

FCFS:     service in arrival order — fair but slow
SSTF:     Shortest Seek Time First — fast but starvation
SCAN:     move arm in one direction servicing requests, reverse at end (elevator)
C-SCAN:   like SCAN but only service on sweep in one direction (fairer)
LOOK:     like SCAN but only go as far as last request (not disk edge)

For SSDs: FCFS or FIFO, possibly reordering for write combining.
          Seek time is irrelevant; wear leveling is the constraint.
```

### Buffer Cache / Page Cache

```
OS caches disk blocks in RAM (the "page cache"):
  Read: check cache first; if hit, return from RAM (no disk I/O)
  Write: write to cache (dirty); background writeback to disk

write() is asynchronous by default:
  Returns to application immediately after writing to page cache.
  Data persists on disk eventually (dirty page writeback: every 30 seconds by default).
  
fsync(fd):   flush this file's dirty pages to disk NOW (guaranteed durable after return)
fdatasync():  like fsync but skips metadata unless needed for data recovery
sync():       flush ALL dirty pages (system-wide)
O_DIRECT:    bypass page cache (for databases with their own caching)
O_SYNC:      every write() is an implicit fsync() (very slow but safe)

Page cache sizing:
  Linux: uses all free RAM for page cache; shrinks when processes need memory.
  "free" output: "buff/cache" column = page cache (not really "used")
```

---

## 9. Deadlocks

### Four Necessary Conditions (Coffman Conditions)

All four must hold for a deadlock to occur:

```
1. Mutual Exclusion:
   Resources cannot be shared — only one process uses a resource at a time.

2. Hold and Wait:
   A process holds at least one resource while waiting to acquire more.

3. No Preemption:
   Resources cannot be forcibly taken; must be voluntarily released.

4. Circular Wait:
   A cycle exists: P1 waits for P2, P2 waits for P3, ..., Pn waits for P1.
```

### Resource Allocation Graph

```
Circle = process   Rectangle = resource   Dots inside = instances
P → R:  process P requests resource R
R → P:  resource R is held by process P

Deadlock ↔ cycle in graph (when resources have single instances)

P1 ──►[R1]──► P2
│              │
▼              ▼
[R2]◄── ...  [R3]◄──
              
Cycle: P1→R1→P2→R2→P1 → DEADLOCK
```

### Deadlock Handling Strategies

#### Prevention (eliminate one Coffman condition)

```
Eliminate Mutual Exclusion:
  Make resources shareable (read-only data, spooling for printers).
  Not always possible (mutex must be exclusive).

Eliminate Hold and Wait:
  Request all resources at once before starting.
  - Wastes resources (hold entire DB while waiting for printer)
  - May starve if popular resources always busy

Eliminate No Preemption:
  If a process can't get a resource, preempt its currently held resources.
  Only works for resources whose state can be saved/restored (CPU registers, memory).
  Doesn't work for printers, database locks.

Eliminate Circular Wait:
  Impose a total ordering on resource types; always request in order.
  If R1 < R2 < R3: any process must request them in that order.
  → Cycle impossible (a cycle requires a "backwards" edge)
  ✅ Most practical prevention technique.
```

#### Avoidance (Banker's Algorithm)

```
Dynamically check if granting a request leads to a safe state.
Safe state: an ordering of processes exists where each can complete.

Banker's Algorithm (Dijkstra):
  Data: Available[m], Max[n][m], Allocation[n][m], Need[n][m]
  Need[i][j] = Max[i][j] - Allocation[i][j]

  Safety check:
  1. Work = Available; Finish[i] = false for all i
  2. Find i: Finish[i]==false AND Need[i] ≤ Work
  3. Work += Allocation[i]; Finish[i] = true; goto 2
  4. If all Finish[i]==true: SAFE state

  On resource request: tentatively grant, run safety check.
  If safe: grant. If unsafe: deny (make process wait).

  ✅ Avoids deadlock without killing processes.
  ✗ Requires knowing max resource needs in advance.
  ✗ O(n²m) per request — too slow for many resources.
  ✗ Rarely used in practice (too conservative, complex).
```

#### Detection & Recovery

```
Allow deadlocks to occur, detect them, then recover.

Detection:
  Run resource allocation graph cycle detection periodically.
  Or: detect when CPU utilization drops despite ready processes.

Recovery options:
  1. Kill a process: choose victim (lowest priority, most resources, most work done)
  2. Preempt resources: roll back process to a checkpoint, release resources
  3. Kill all deadlocked processes (drastic but simple)

OS approach:
  Linux/Windows: typically ignore or kill deadlocked processes.
  Databases (PostgreSQL, MySQL): detect deadlock via lock wait graph; abort one transaction.
```

#### Ostrich Algorithm

```
Ignore the problem. Reboot/restart if it happens.
Used by most general-purpose OSes for user-space processes.
Rationale: deadlocks are rare; overhead of prevention/avoidance not worth it.
```

---

## 10. Virtualization & Containers

### Hardware Virtualization

```
Hypervisor: software that creates and manages virtual machines (VMs).

Type 1 (Bare-metal): runs directly on hardware (no host OS)
  Examples: VMware ESXi, Microsoft Hyper-V, Xen, KVM (Linux kernel module)
  + Better performance (no host OS overhead)
  + Used in cloud providers (AWS EC2, Azure, GCP)

Type 2 (Hosted): runs on top of a host OS
  Examples: VMware Workstation, VirtualBox, Parallels
  + Easier to install/use on desktop
  - More overhead (host OS in the way)

Key mechanisms:
  CPU virtualization:
    Trap-and-emulate: privileged instructions in guest trap to hypervisor.
    Binary translation (pre-VT-x): rewrite privileged instructions.
    Hardware-assisted (Intel VT-x, AMD-V): VMX root/non-root modes.
      VM exits: guest executes sensitive instruction → CPU traps to hypervisor.
      VMCS: per-VM data structure holding guest/host state.
  
  Memory virtualization:
    Shadow page tables (software): hypervisor maintains guest VA → host PA mapping.
    EPT/NPT (hardware): two-level page table walk in hardware (guest PA → host PA).
    IOMMU: protects DMA from devices directly to guest memory.
  
  I/O virtualization:
    Emulation: hypervisor emulates device in software (slow).
    Para-virtualization: guest knows it's virtualized, uses efficient hypercalls (virtio).
    Pass-through (SR-IOV): guest accesses hardware device directly (PCIe).
```

### Containers

```
Containers virtualize the OS, not the hardware.
All containers share the same kernel.

Linux building blocks:
  namespaces:  isolate what a process can see
    pid:    process can only see processes in its namespace
    net:    separate network stack (interfaces, routing, firewall)
    mnt:    separate filesystem view (mount points)
    uts:    separate hostname and domain name
    ipc:    separate IPC (message queues, semaphores, shared memory)
    user:   map user/group IDs (root in container ≠ root on host)
    cgroup: separate cgroup hierarchy view (Linux 4.6+)
  
  cgroups (control groups): limit and account for resource usage
    cpu:     limit CPU time (cpu.shares, cpu.quota/period)
    memory:  limit RAM + swap; OOM kill when exceeded
    blkio:   limit block I/O bandwidth
    net_cls: tag network packets for traffic control
    devices: allow/deny access to device files

  Union filesystems (overlayfs):
    Layers stacked: base image (read-only) + container layer (read-write).
    Copy-on-write: modifications go to top layer; base unchanged.
    Enables: image sharing, fast container start, small diffs.

VM vs Container:
            VMs               Containers
  Isolation  Strong (hardware)  Weaker (kernel shared)
  Overhead   ~5-10% CPU/RAM     ~1% CPU/RAM
  Boot time  Seconds-minutes    Milliseconds
  Density    10s per host        100s-1000s per host
  OS         Any                 Linux (mostly)
  Use case   Multi-tenant cloud  Microservices, CI/CD
```

### Docker & Container Runtime

```
Image: read-only layered filesystem + metadata (entrypoint, env vars, ports)
Container: running instance of an image (adds writable layer)
Registry: storage for images (Docker Hub, ECR, GCR)

Container lifecycle:
  docker pull image:tag       → download image layers
  docker run image cmd        → create + start container
  docker exec -it cid bash    → attach to running container
  docker stop cid             → SIGTERM, then SIGKILL after timeout
  docker rm cid               → remove stopped container

Networking:
  bridge: default; containers on private virtual network; NAT to host
  host:   container shares host's network namespace (no isolation, max performance)
  none:   no network
  overlay: multi-host networking (Docker Swarm, Kubernetes)

Resource limits:
  docker run --cpus=2 --memory=512m image
  → sets cgroup cpu quota and memory limit
```

---

## 11. Security & Protection

### Protection Mechanisms

```
Principle of least privilege:
  Every process/user should have only the minimum permissions needed.

Protection rings (x86):
  Ring 0: kernel (full access)
  Ring 1,2: rarely used (hypervisors sometimes use ring 1)
  Ring 3: user applications (restricted)

Access Control:
  DAC (Discretionary Access Control): owner sets permissions (Unix rwx)
  MAC (Mandatory Access Control): system policy overrides owner (SELinux, AppArmor)
  RBAC (Role-Based Access Control): permissions assigned to roles, roles to users

Capabilities (Linux):
  Split root's power into fine-grained capabilities.
  CAP_NET_ADMIN: configure network
  CAP_SYS_PTRACE: trace processes
  CAP_CHOWN: change file ownership
  A process needs only specific capabilities, not full root.
  getcap / setcap; docker --cap-add/--cap-drop
```

### Common Attack Vectors & OS Defenses

```
Buffer Overflow:
  Write past end of buffer → overwrite return address → execute attacker code.
  Defenses:
    ASLR (Address Space Layout Randomization): randomize stack/heap/library addresses
    Stack canaries: sentinel value before return address; check before return
    NX/DEP (No-Execute): mark stack/heap non-executable (W^X: Write XOR Execute)
    SafeStack: separate stacks for sensitive data

Return-Oriented Programming (ROP):
  Chains of existing code snippets ("gadgets") ending in RET to execute arbitrary logic.
  Bypasses NX by reusing existing executable code.
  Defenses: CFI (Control Flow Integrity), shadow stack (Intel CET)

Use-After-Free:
  Access memory after it's been freed.
  Defenses: ASAN (AddressSanitizer), safe allocators, memory tagging (ARM MTE)

Privilege Escalation:
  Exploit kernel vulnerability or SUID program to gain root.
  Defenses: kernel hardening (SMEP/SMAP), seccomp-BPF, namespace isolation

Spectre / Meltdown (microarchitectural attacks):
  Exploit CPU speculation and cache timing to read across privilege boundaries.
  Meltdown: user process reads kernel memory via speculative execution.
  Spectre: trick other programs into leaking data via speculative execution.
  Defenses:
    Meltdown: KPTI (Kernel Page-Table Isolation) — separate page tables for user/kernel
    Spectre: retpoline, microcode updates, index masking in kernel
    Cost: 5–30% performance hit for I/O-heavy workloads
```

### Secure System Calls

```
seccomp (secure computing mode):
  Whitelist/blacklist system calls a process can make.
  Violation → SIGKILL or error.
  Used by: Docker, Chrome, Firefox, systemd sandboxing.

  seccomp(SECCOMP_SET_MODE_STRICT)   // only read, write, exit, sigreturn allowed
  seccomp(SECCOMP_SET_MODE_FILTER, &prog)  // BPF program to filter syscalls

SELinux / AppArmor:
  Mandatory Access Control — policy specifies what every process can access.
  Policy violations are logged and blocked regardless of DAC permissions.
  SELinux:  label-based (every file/process/socket has a label; policy = allowed transitions)
  AppArmor: path-based (simpler; rules based on file paths)
```

### Authentication & Credentials

```
Unix user/group model:
  UID (User ID): 0 = root (superuser), 1-999 = system, 1000+ = normal users
  GID (Group ID): primary group + supplementary groups
  Stored in /etc/passwd (users) and /etc/shadow (hashed passwords)

Password hashing:
  Never store plaintext. Store: hash(password + salt)
  Modern: bcrypt, scrypt, argon2 (intentionally slow to resist brute force)
  Legacy (insecure): MD5, SHA1, unsalted hashes

Setuid/Setgid:
  Executable with SUID bit runs with FILE OWNER's privileges, not caller's.
  Example: /usr/bin/passwd has SUID root → any user can change their password
           but only writes to /etc/shadow (kernel enforces what passwd can do)
  
File capabilities vs SUID:
  Prefer capabilities (finer-grained) over full SUID root.
```

---

## 12. Quick Reference

### System Call Overhead Summary

| Operation | Approximate Cost |
|-----------|-----------------|
| System call (no work) | ~100–500 ns |
| Context switch | ~1–10 µs |
| Thread create (pthread) | ~10–50 µs |
| Process create (fork) | ~100–500 µs |
| Pipe write/read (small) | ~1–5 µs |
| Unix socket (localhost) | ~5–15 µs |
| TCP loopback | ~20–50 µs |
| mmap (small file) | ~1 µs (after first fault) |
| Page fault (present page) | ~100 ns |
| Page fault (disk) | ~100 µs–10 ms |

### Scheduling Algorithm Comparison

| Algorithm | Preemptive | Starvation | Overhead | Best For |
|-----------|-----------|------------|----------|---------|
| FCFS | No | No | Low | Batch |
| SJF | No | Yes | Low | Batch (known bursts) |
| SRTF | Yes | Yes | Medium | Minimizing turnaround |
| Round Robin | Yes | No | Medium | Interactive/time-sharing |
| Priority | Both | Yes | Low | Mixed workloads |
| MLFQ | Yes | No (aging) | High | General purpose |
| CFS | Yes | No | Medium | Linux general purpose |
| EDF | Yes | No | High | Real-time |

### IPC Mechanism Comparison

| Mechanism | Speed | Persistence | Multi-host | Best For |
|-----------|-------|-------------|------------|---------|
| Shared memory | ★★★★★ | No | No | High-bandwidth same-machine |
| Pipe | ★★★★ | No | No | Simple parent-child |
| Unix socket | ★★★★ | No | No | Local client-server |
| TCP socket | ★★★ | No | Yes | Network / general |
| Message queue | ★★★ | Yes (kernel) | No | Decoupled async |
| mmap file | ★★★★ | Yes (disk) | No | Shared persistent data |
| Signal | ★★★★ | No | No | Notification only |

### Memory Allocation Quick Reference

| Allocator | Use Case | Properties |
|-----------|---------|------------|
| Buddy system | Kernel page allocator | Power-of-2 sizes, O(log n) |
| Slab/SLUB | Kernel object allocator | Per-type caches, zero-init cost |
| jemalloc | Userspace (Firefox, Redis) | Size classes, per-thread arenas |
| tcmalloc | Userspace (Google) | Thread caching, fast small allocs |
| mimalloc | Userspace (Microsoft) | Low overhead, security features |
| Arena/pool | Application-level | O(1) alloc, bulk free |

### Key Kernel Data Structures

| Structure | Purpose | Location |
|-----------|---------|----------|
| PCB / task_struct | Process state | Per process |
| Page table | VA → PA translation | Per process |
| File descriptor table | Open files | Per process |
| Inode | File metadata | Per file (on disk + in memory) |
| Dentry | Name → inode mapping | In-memory cache |
| VMA (vm_area_struct) | Virtual memory regions | Per process |
| Run queue | Ready processes | Per CPU |
| Free list / buddy | Physical frame tracking | Per NUMA node |

### Critical Numbers to Know

```
Page size:         4 KB (standard), 2 MB / 1 GB (huge pages)
TLB entries:       32–2048 typical
L1 cache hit:      4 cycles
Context switch:    ~1–10 µs
System call:       ~100–500 ns
Pipe buffer:       64 KB (Linux default)
Default stack:     8 MB per thread (Linux)
Max open files:    1,048,576 (Linux default soft limit: 1024)
PID max:           4,194,304 (Linux)
Clock tick (HZ):   100–1000 Hz (10–1ms timer interrupt)
OOM score:         0–1000 (higher = killed first)
```

### File Permission Quick Reference

```
Octal  Binary  Symbolic  Meaning
  7    111     rwx       read + write + execute
  6    110     rw-       read + write
  5    101     r-x       read + execute
  4    100     r--       read only
  3    011     -wx       write + execute
  2    010     -w-       write only
  1    001     --x       execute only
  0    000     ---       no permission

Common modes:
  755 = rwxr-xr-x  (executable/directory: owner full, others read+exec)
  644 = rw-r--r--  (regular file: owner read+write, others read)
  700 = rwx------  (private executable)
  600 = rw-------  (private file, SSH keys)
  1777 = rwxrwxrwt (sticky bit: /tmp — anyone write, only owner delete)
```

---

*For deeper study: "Operating Systems: Three Easy Pieces" (Arpaci-Dusseau) — free online, excellent. "The Linux Programming Interface" (Kerrisk) for Linux-specific detail. "Modern Operating Systems" (Tanenbaum) for theory.*
