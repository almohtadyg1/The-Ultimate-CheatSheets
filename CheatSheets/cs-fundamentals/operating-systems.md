# Operating Systems: A Complete Progressive Tutorial

---

## 1. What & Why

An operating system is the software layer between raw hardware and the applications running on it. It manages physical resources — CPU time, RAM, disk I/O, network — and provides abstractions that make programming possible without dealing with hardware directly. Every web server, database, containerized application, and CLI tool runs on top of an OS's services.

Why does a software engineer need to understand OS concepts? Because the OS is why your program behaves the way it does. When a web server handles 10,000 concurrent connections, the OS decides how to schedule those threads. When your database writes to disk, the OS's file system and page cache determines whether that's fast or slow. When your process crashes with a segfault, the OS's memory manager triggered that. Understanding processes, threads, memory management, scheduling, and file I/O turns OS symptoms into diagnosable causes.

---

## 2. Mental Model

The OS acts as a resource manager and referee, providing three core abstractions:

```
Hardware Reality          OS Abstraction          What You Use
─────────────────────────────────────────────────────────────────
Raw CPU cores         →   Process / Thread    →   Your running program
Physical RAM          →   Virtual Memory      →   Pointer arithmetic
Hard drive sectors    →   File System         →   open(), read(), write()
Physical addresses    →   Virtual addresses   →   Any pointer you dereference
Real time (CPU)       →   CPU scheduling      →   Just run your code

Two privilege levels (x86):
  Ring 0 (kernel):  full hardware access, any instruction
  Ring 3 (user):    restricted — must ask kernel via system calls

System call crossing the boundary:
  User code calls open("/etc/passwd", O_RDONLY)
       ↓
  CPU executes SYSCALL instruction (switches to ring 0)
       ↓
  Kernel validates parameters, does the real work
       ↓
  Returns result to user process (CPU switches back to ring 3)
  Cost: ~100-1000 ns per syscall — avoid in tight loops
```

---

## 3. Progressive Examples

### Level 1: Processes — The Unit of Execution

```c
// A process is a running program with its own:
// - Address space (virtual memory)
// - Open file descriptors
// - Program counter and registers
// - Process ID (PID), parent PID (PPID)
// - User/group ownership
// - Signal handlers

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>   // fork(), exec(), getpid()
#include <sys/wait.h>

int main() {
    printf("Parent PID: %d\n", getpid());

    pid_t child_pid = fork();   // Create an exact copy of this process

    if (child_pid < 0) {
        perror("fork failed");
        exit(1);
    }

    if (child_pid == 0) {
        // ---- CHILD PROCESS ----
        // fork() returns 0 in the child
        printf("Child PID: %d, Parent: %d\n", getpid(), getppid());

        // Replace child process image with a new program
        execlp("ls", "ls", "-la", NULL);   // exec never returns on success
        perror("exec failed");              // only reached if exec fails
        exit(1);
    } else {
        // ---- PARENT PROCESS ----
        // fork() returns child's PID in the parent
        printf("Parent spawned child %d\n", child_pid);

        int status;
        waitpid(child_pid, &status, 0);    // wait for child to finish
        if (WIFEXITED(status)) {
            printf("Child exited with status: %d\n", WEXITSTATUS(status));
        }
    }
    return 0;
}

// Process lifecycle:
// new → ready → running → (blocked/waiting) → terminated
//
// ready: in scheduler queue, waiting for CPU
// running: executing on CPU
// blocked: waiting for I/O, a lock, or a signal
```

```python
# Process operations from Python
import os
import subprocess
import signal

# Spawn a child process
result = subprocess.run(
    ["ls", "-la", "/tmp"],
    capture_output=True, text=True, timeout=10
)
print(result.stdout)
print(result.returncode)   # 0 = success

# Long-running process
proc = subprocess.Popen(["sleep", "60"])
print(f"Started process {proc.pid}")

# Send a signal
os.kill(proc.pid, signal.SIGTERM)   # polite termination
proc.wait()

# Fork in Python (Unix only)
pid = os.fork()
if pid == 0:
    # Child
    print(f"Child: {os.getpid()}")
    os._exit(0)   # use _exit() in child, not exit() (avoids atexit handlers)
else:
    # Parent
    os.waitpid(pid, 0)
```

### Level 2: Threads and Concurrency

```python
import threading
import multiprocessing
import time
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

# Thread vs Process:
# Thread: shares memory with other threads in same process (faster communication)
# Process: isolated memory (safer, but IPC required to share data)

# Python's GIL: only one thread runs Python bytecode at a time.
# Threads ARE parallel for I/O-bound work (GIL released during I/O).
# Threads are NOT parallel for CPU-bound work.
# Use multiprocessing for CPU-bound parallelism.

# ---- Threads for I/O-bound work ----

def fetch_url(url):
    import urllib.request
    with urllib.request.urlopen(url, timeout=5) as r:
        return len(r.read())

urls = ["https://httpbin.org/delay/1"] * 4

# Sequential: ~4 seconds
start = time.time()
results = [fetch_url(u) for u in urls]
print(f"Sequential: {time.time()-start:.1f}s")

# Concurrent threads: ~1 second (GIL released during I/O)
start = time.time()
with ThreadPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(fetch_url, urls))
print(f"Threaded:   {time.time()-start:.1f}s")

# ---- Processes for CPU-bound work ----

def compute_heavy(n):
    return sum(i*i for i in range(n))

# Sequential: slow
start = time.time()
[compute_heavy(1_000_000) for _ in range(4)]
print(f"Sequential: {time.time()-start:.1f}s")

# Parallel processes: ~4x speedup on quad-core
start = time.time()
with ProcessPoolExecutor() as executor:
    list(executor.map(compute_heavy, [1_000_000]*4))
print(f"Processes:  {time.time()-start:.1f}s")

# ---- Thread synchronization ----
counter = 0
lock = threading.Lock()

def increment(n):
    global counter
    for _ in range(n):
        with lock:    # acquire lock, guaranteed release even on exception
            counter += 1

threads = [threading.Thread(target=increment, args=(10000,)) for _ in range(10)]
for t in threads: t.start()
for t in threads: t.join()
print(f"Counter: {counter}")   # exactly 100000 (without lock: race condition)
```

### Level 3: Memory Management — Virtual Memory and the Page Table

```
VIRTUAL MEMORY: Every process gets its own address space (0 to 2^64 on 64-bit)

Virtual address space of a process (Linux x86-64):
  0xFFFFFFFFFFFFFFFF ← kernel space (inaccessible to user code)
  ─────────────────
  0x7FFFFFFFFFFF    ← stack (grows down, stores local variables, return addresses)
  ...
  [large gap — unmapped, any access = segfault]
  ...
  heap (grows up — malloc/new allocates here)
  BSS  (zero-initialized globals: int global_array[1000])
  Data (initialized globals: int x = 42)
  Text (executable code — read-only)
  0x0000000000000000  ← NULL pointer dereference lands here

PAGES: The OS works in pages (typically 4 KB).
Each virtual page maps to a physical page frame (or is swapped to disk).
The page table stores this mapping: virtual page number → physical frame number.

Page fault: accessing an unmapped page.
  Minor fault: page exists (in another process, swap, or cow) → OS maps it, resume
  Major fault: page is on disk → read it in, map it, resume (expensive!)
  Invalid fault: bad pointer → SIGSEGV (segfault)

COPY-ON-WRITE (COW): fork() is efficient because of COW.
After fork(), parent and child share the same physical pages (read-only).
When either writes, the OS copies that page and gives each its own copy.
This is why fork() + exec() (spawning a process) is cheap.
```

```c
#include <sys/mman.h>
#include <stdio.h>
#include <string.h>

// mmap: map memory directly (used by malloc, file I/O, shared memory)

// Anonymous mapping (like malloc but directly from OS)
void *buf = mmap(NULL, 4096,
                 PROT_READ | PROT_WRITE,    // read and write permission
                 MAP_PRIVATE | MAP_ANONYMOUS, // private, not backed by file
                 -1, 0);
memset(buf, 0, 4096);
munmap(buf, 4096);

// File-backed mapping: treat a file as memory
int fd = open("data.bin", O_RDONLY);
void *data = mmap(NULL, file_size, PROT_READ, MAP_PRIVATE, fd, 0);
// Access data[offset] instead of read() calls — OS handles I/O lazily
munmap(data, file_size);

// mprotect: change permissions on mapped memory
// Used by JIT compilers: allocate as RW (write code), then mark as RX (execute)
mprotect(buf, 4096, PROT_READ | PROT_EXEC);
```

### Level 4: File Systems and I/O

```
FILE SYSTEM STRUCTURE (ext4, NTFS, APFS all follow similar principles):

  Superblock: filesystem metadata (size, block size, free blocks, UUID)
  Inodes:     one per file/directory — stores metadata (size, permissions, timestamps,
              pointers to data blocks) but NOT the filename
  Data blocks: actual file content
  Directory:  a file containing (filename → inode number) mappings

Finding /home/alice/report.pdf:
  1. Start at root inode (always inode 2)
  2. Read root directory: "home" → inode 17
  3. Read inode 17 (directory): "alice" → inode 83
  4. Read inode 83 (directory): "report.pdf" → inode 217
  5. Read inode 217: permissions, size, data block pointers
  6. Read data blocks → file contents

Journaling: ext4, NTFS keep a journal of pending changes.
Before modifying the filesystem, write the intent to the journal.
If a crash occurs mid-write, replay the journal on next mount.
Without journaling: corrupted filesystem on crash.

Page cache: the OS caches file data in RAM.
read() first checks the page cache. If data is there, no disk I/O.
This is why repeated reads of the same file are fast.
The page cache is also why free memory appears low on Linux — it's in use as cache.
```

```python
import os
import io

# File I/O modes and their OS-level implications
with open("data.txt", "r") as f:      # O_RDONLY
    content = f.read()                 # buffered in Python's stdio buffer

with open("log.txt", "a") as f:       # O_WRONLY | O_APPEND (atomic append)
    f.write("log entry\n")

# Direct I/O (bypass page cache) — useful for databases
# O_DIRECT: writes go straight to disk, no kernel buffering
# Requires alignment to block size (usually 512 or 4096 bytes)

# stat: file metadata without reading content
stat = os.stat("/etc/passwd")
print(f"Size: {stat.st_size}")
print(f"Inode: {stat.st_ino}")
print(f"Permissions: {oct(stat.st_mode)}")
print(f"Last modified: {stat.st_mtime}")

# Hard links vs symbolic links
# Hard link: another directory entry pointing to the SAME inode
# Deleting one doesn't remove the file until all hard links are gone
os.link("file.txt", "hard_link.txt")      # same inode

# Symbolic link: a file containing a path to another file
os.symlink("/etc/passwd", "passwd_link")   # different inode, stores path

# /proc filesystem on Linux: virtual filesystem, not on disk
# Each process has /proc/<pid>/ with info about it
with open(f"/proc/{os.getpid()}/status") as f:
    for line in f:
        if "VmRSS" in line or "VmSize" in line:
            print(line.strip())   # memory usage of current process
```

### Level 5: Scheduling, IPC, and Signals

```python
import os
import signal
import time
import multiprocessing

# ---- CPU SCHEDULING ----
# The scheduler decides which thread runs on which CPU core and for how long.
#
# Linux Completely Fair Scheduler (CFS):
#   Tracks "virtual runtime" (vruntime) for each task.
#   The task with lowest vruntime runs next.
#   Lower-priority tasks accumulate vruntime faster (run less).
#   Preemptive: any task can be interrupted when its time slice expires.
#
# Priority (nice values):
#   -20 (highest priority) to 19 (lowest)
#   Default: 0. Only root can set negative nice values.

import resource
# Set CPU time limit
resource.setrlimit(resource.RLIMIT_CPU, (10, 10))   # soft=10s, hard=10s

# Process priority
os.nice(10)   # lower priority (cooperative, don't hog CPU)

# ---- INTER-PROCESS COMMUNICATION (IPC) ----

# 1. Pipes: one-way byte stream between related processes
r_fd, w_fd = os.pipe()   # file descriptors

pid = os.fork()
if pid == 0:              # child: writes to pipe
    os.close(r_fd)
    os.write(w_fd, b"Hello from child\n")
    os.close(w_fd)
    os._exit(0)
else:                     # parent: reads from pipe
    os.close(w_fd)
    data = os.read(r_fd, 1024)
    os.close(r_fd)
    os.waitpid(pid, 0)
    print(data.decode())

# 2. Shared memory (fastest IPC — zero-copy)
from multiprocessing import shared_memory

# Create shared memory block
shm = shared_memory.SharedMemory(create=True, size=1024)
# Write
shm.buf[0:5] = b"hello"
# Another process can attach by name and read it
shm.close()
shm.unlink()

# 3. Message queues / multiprocessing.Queue
q = multiprocessing.Queue()
def producer(q):
    for i in range(5):
        q.put(f"item_{i}")
    q.put(None)   # sentinel

def consumer(q):
    while True:
        item = q.get()
        if item is None: break
        print(f"Got: {item}")

# ---- SIGNALS ----
# Asynchronous notifications sent to processes

# Register a handler
def sigterm_handler(signum, frame):
    print("Got SIGTERM — cleaning up...")
    # flush buffers, close connections, etc.
    exit(0)

signal.signal(signal.SIGTERM, sigterm_handler)
signal.signal(signal.SIGINT, signal.SIG_IGN)   # ignore Ctrl+C

# Common signals:
# SIGTERM (15): graceful termination request — handle it for clean shutdown
# SIGKILL (9):  force kill, CANNOT be caught or ignored
# SIGINT (2):   Ctrl+C from terminal
# SIGHUP (1):   terminal hangup (often used to reload config)
# SIGSEGV (11): segfault — invalid memory access
# SIGCHLD:      child process state changed (exited, stopped)
```

### Level 6: Containers and Virtualization

```
VIRTUALIZATION HIERARCHY:

Hardware (CPU, RAM, NIC, disk)
   │
   ├── Hypervisor (Type 1: VMware ESXi, KVM, Hyper-V)
   │       │
   │       ├── Virtual Machine: complete OS + kernel + user space
   │       │   Isolation: strong (separate kernel, virtualized hardware)
   │       │   Overhead:  high (full OS + hypervisor overhead)
   │       │   Boot time: seconds to minutes
   │
   └── Host OS (Linux, macOS, Windows)
           │
           └── Container Runtime (Docker, containerd)
                   │
                   └── Container: isolated user space, SHARES the host kernel
                       Isolation: good (namespaces + cgroups + seccomp)
                       Overhead:  minimal (processes, not VMs)
                       Boot time: milliseconds

LINUX NAMESPACES (what makes containers isolated):
  pid:    container has its own PID 1 (init)
  net:    own network interfaces, routing table, ports
  mnt:    own filesystem root (pivot_root)
  uts:    own hostname
  ipc:    own shared memory, semaphores
  user:   own UID/GID mapping (user namespace)
  cgroup: own resource limits view

CGROUPS (resource limits):
  cpu:    max CPU usage (e.g., 0.5 cores)
  memory: max RAM + swap (e.g., 512MB hard limit)
  io:     max disk I/O throughput
  net_cls: tag traffic for QoS shaping
```

```bash
# See what namespaces a process is in
ls -la /proc/self/ns/

# Create a minimal container namespace manually
# (what Docker does under the hood, simplified)
unshare --pid --fork --mount-proc /bin/bash  # new PID namespace
# Inside: ps shows only this bash and its children

# cgroups v2: resource limits
mkdir /sys/fs/cgroup/mycontainer
echo "512M" > /sys/fs/cgroup/mycontainer/memory.max
echo "100000 1000000" > /sys/fs/cgroup/mycontainer/cpu.max  # 10% of one CPU
echo $$ > /sys/fs/cgroup/mycontainer/cgroup.procs  # add this shell to the cgroup

# strace: trace system calls of a process
strace -p 1234              # trace running process
strace -c ls                # summary of syscalls made by ls
strace -e trace=open,read,write ls  # only these syscalls

# lsof: list open file descriptors
lsof -p 1234               # all files open by process 1234
lsof /etc/passwd           # what processes have this file open
lsof -i :8080              # what process is using port 8080
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Assuming threads always improve performance**

Python threads do not parallelize CPU-bound work due to the GIL. Adding more threads to a CPU-intensive task slows it down (context switching overhead, GIL contention). Use `multiprocessing.ProcessPoolExecutor` for CPU-bound work. Threads genuinely help for I/O-bound work where threads wait on network or disk rather than competing for the CPU.

**Mistake 2: Not handling SIGTERM in long-running processes**

```python
# WRONG: process ignores SIGTERM, killed forcefully, data corrupted
# No signal handler → default behavior: terminate immediately

# CORRECT: handle SIGTERM for graceful shutdown
import signal, sys

def graceful_shutdown(signum, frame):
    print("Received SIGTERM — flushing buffers, closing connections...")
    # db.close(), cache.flush(), log.info("shutdown")
    sys.exit(0)

signal.signal(signal.SIGTERM, graceful_shutdown)
```

**Mistake 3: Zombie processes from not waiting for children**

```python
# WRONG: parent never calls wait() — child becomes a zombie (entry in process table)
pid = os.fork()
if pid == 0:
    os._exit(0)   # child exits immediately
# Parent never calls waitpid() → zombie lingers until parent exits

# CORRECT: always wait for children
pid = os.fork()
if pid == 0:
    os._exit(0)
else:
    os.waitpid(pid, 0)   # reap the zombie

# OR: set SIGCHLD to SIG_IGN to auto-reap children
signal.signal(signal.SIGCHLD, signal.SIG_IGN)
```

**Mistake 4: Treating virtual memory as if it equals physical RAM**

A process can have a virtual address space of 100 GB but only use 1 GB of physical RAM if most pages are never accessed. `ps` shows VSZ (virtual) and RSS (physical resident). Monitoring RSS gives the actual memory footprint. Programs can allocate large virtual ranges cheaply; physical RAM is only committed when pages are actually written to.

---

## 5. The "Why Does This Work" Layer

### Why fork() Is Fast Despite Copying the Address Space

`fork()` appears to copy the entire address space, which could be gigabytes. It's actually near-instant because of Copy-on-Write (COW). After fork(), the parent and child share all physical pages, marked read-only. The page table entries in both processes point to the same physical frames. Only when either process writes to a page does the OS allocate a new physical frame, copy the page, and update the writing process's page table. Pages that are never written are never copied.

### Why Context Switches Are Expensive

When the OS switches from thread A to thread B, it must: save A's CPU registers (16 general-purpose registers + FP/SIMD state = ~2KB), save A's program counter, invalidate TLB entries (virtual-to-physical address cache) if switching processes (not threads), load B's registers and program counter. The TLB flush is the most expensive part — after switching, the next 100+ memory accesses will all miss the TLB and require page table walks. This is why fine-grained locking with many context switches can be slower than a single-threaded approach for cache-heavy workloads.

### How the File System Page Cache Works

When you `read()` a file, the OS first checks if the relevant pages are already in the page cache (RAM). If yes: zero disk I/O, just copy from kernel memory to your buffer. If not: read from disk, place in page cache, copy to your buffer. Subsequent reads get the cached version.

The page cache is shared across all processes. If two processes read the same file, both see the same physical pages. `free` on Linux shows RAM as "used" even when it's just page cache — the OS will evict cached pages under memory pressure. This is why Linux's "available" memory is more informative than "free".

---

## 6. Quick Reference

### Process States

```
new → ready ⇆ running → terminated
                ↓  ↑
              blocked
              (waiting for I/O, lock, or signal)
```

### Key System Calls

| Call | Purpose |
|------|---------|
| `fork()` | Create child process (copy-on-write) |
| `exec()` | Replace process image with new program |
| `wait()` | Wait for child process to exit |
| `mmap()` | Map files or anonymous memory |
| `open()/read()/write()` | File I/O |
| `pipe()` | Create IPC pipe |
| `socket()` | Create network socket |
| `clone()` | Linux thread/process creation (fine-grained) |

### IPC Mechanisms

| Mechanism | Speed | Use Case |
|-----------|-------|---------|
| Pipe | Fast | Parent-child, unidirectional |
| Socket | Medium | Network or local (AF_UNIX) |
| Shared memory | Fastest | Large data, zero-copy |
| Message queue | Medium | Structured messages |
| File | Slow | Persistent, any process |
| Signal | Fastest | Notifications only |

### Memory Segments

| Segment | Contents | Grows |
|---------|---------|-------|
| Text | Compiled code | Fixed |
| Data | Initialized globals | Fixed |
| BSS | Zero-init globals | Fixed |
| Heap | malloc/new | Upward |
| Stack | Local vars, return addrs | Downward |
| mmap | Shared libs, mmap'd files | Variable |
