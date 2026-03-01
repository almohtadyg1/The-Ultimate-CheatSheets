# 🖥️ The Complete Computer Architecture Cheat Sheet
### From Zero to Expert

---

## 1. What is Computer Architecture?

Computer architecture is the set of rules and methods that describe the **functionality, organization, and implementation** of a computer system. It sits at the boundary between hardware and software.

### The Three Abstraction Layers

```
         Software (what you write)
              ↕
    ISA — Instruction Set Architecture
    (the contract between SW and HW)
              ↕
         Microarchitecture
    (how the ISA is implemented in silicon)
              ↕
         Digital Logic
    (gates, transistors, electrons)
```

### Von Neumann Architecture (Foundation of Modern Computers)

```
┌──────────────────────────────────────────┐
│                   CPU                    │
│  ┌────────────┐    ┌──────────────────┐  │
│  │   Control  │    │       ALU        │  │
│  │    Unit    │    │ (arithmetic &    │  │
│  │            │    │  logic ops)      │  │
│  └─────┬──────┘    └────────┬─────────┘  │
│        │                    │            │
│  ┌─────▼────────────────────▼─────────┐  │
│  │            Registers               │  │
│  └─────────────────┬──────────────────┘  │
└────────────────────│────────────────────┘
                     │  Bus (address/data/control)
          ┌──────────┴──────────┐
          │                     │
    ┌─────▼──────┐        ┌─────▼──────┐
    │   Memory   │        │    I/O     │
    │ (programs  │        │  Devices   │
    │  + data)   │        │            │
    └────────────┘        └────────────┘

Key insight: Program instructions and data share the same memory.
This is both elegant and a bottleneck (Von Neumann bottleneck).
```

### Harvard vs Von Neumann

| | Von Neumann | Harvard |
|---|---|---|
| Memory | Unified (code + data) | Separate (code vs data) |
| Buses | Shared | Separate |
| Flexibility | Higher | Lower |
| Speed | Limited by bus sharing | Fetch instruction + data simultaneously |
| Used in | x86, ARM general purpose | Microcontrollers, DSPs, CPU caches |

Modern CPUs are **Modified Harvard** — unified main memory but separate L1 instruction and data caches.

---

## 2. Digital Logic — The Foundation

Everything in a computer reduces to transistors acting as switches.

### Logic Gates

```
AND:   A·B     Output is 1 only when BOTH inputs are 1
OR:    A+B     Output is 1 when EITHER input is 1
NOT:   Ā       Inverts the input
NAND:  ̄(A·B)  AND then invert — universal gate (can build everything from NAND alone)
NOR:   ̄(A+B)  OR then invert  — also universal
XOR:   A⊕B    Output is 1 when inputs DIFFER

NAND and NOR are universal:
  NOT A  = NAND(A, A)
  A AND B = NAND(NAND(A,B), NAND(A,B))
  A OR B  = NAND(NAND(A,A), NAND(B,B))
```

### Combinational vs Sequential Logic

```
Combinational: output depends ONLY on current inputs
  Examples: adders, multiplexers, decoders, ALU

Sequential: output depends on current inputs AND past state (has memory)
  Examples: flip-flops, registers, counters, RAM
```

### The Flip-Flop (Memory Element)

A D flip-flop stores one bit. On each clock edge, output Q takes the value of input D.

```
      D ──┐
          │  ┌──────┐
Clock ────┼──► D  Q ├──── Q
          │  │      │
          └──► CLK  │
             └──────┘

This is the foundation of all registers and RAM.
```

### Building an Adder

```
Half Adder (adds two 1-bit numbers):
  Sum   = A XOR B
  Carry = A AND B

Full Adder (adds two bits + carry-in):
  Sum   = A XOR B XOR Cin
  Cout  = (A AND B) OR (B AND Cin) OR (A AND Cin)

Ripple Carry Adder: chain N full adders for N-bit addition
  Problem: carry must ripple through all N stages → O(N) delay
  Solution: Carry Lookahead Adder computes carries in parallel → O(log N) delay
```

### Clock & Timing

```
Clock signal: square wave oscillating between 0 and 1
Clock speed: 3.5 GHz = 3.5 × 10⁹ cycles per second
Clock period: 1 / 3.5GHz ≈ 0.28 nanoseconds per cycle

Critical path: the longest combinational path between flip-flops
  → determines maximum clock speed
  → deeper pipelines allow faster clocks but add latency

Setup time: how long before clock edge data must be stable
Hold time:  how long after clock edge data must remain stable
Violating these → metastability → unpredictable behavior
```

---

## 3. The CPU — Anatomy & Function

### CPU Components

```
┌─────────────────────────────────────────────────────┐
│                        CPU                          │
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │    PC    │  │    IR    │  │   Register File  │  │
│  │(Program  │  │(Instruc- │  │  r0 r1 r2 ... rN │  │
│  │ Counter) │  │  tion    │  │                  │  │
│  │          │  │Register) │  │  SP (stack ptr)  │  │
│  └──────────┘  └──────────┘  │  FP (frame ptr)  │  │
│                               │  LR (link reg)   │  │
│  ┌──────────────────────────┐ └──────────────────┘  │
│  │           ALU            │                       │
│  │  + - * /  AND OR XOR     │ ┌──────────────────┐  │
│  │  comparisons, shifts     │ │   Flags/Status   │  │
│  └──────────────────────────┘ │  Z N C V         │  │
│                               │  zero neg carry  │  │
│  ┌──────────────────────────┐ │  overflow        │  │
│  │      Control Unit        │ └──────────────────┘  │
│  │  decodes instructions,   │                       │
│  │  orchestrates execution  │                       │
│  └──────────────────────────┘                       │
└─────────────────────────────────────────────────────┘
```

### Registers

Small, extremely fast storage inside the CPU. Access time: ~0 extra cycles.

```
General Purpose Registers: hold operands and results (r0–r15 in ARM, rax–r15 in x86-64)
Special Purpose Registers:
  PC  / RIP  — Program Counter: address of next instruction to fetch
  SP  / RSP  — Stack Pointer:   top of the current stack
  LR  / RA   — Link Register:   return address for function calls
  FP  / RBP  — Frame Pointer:   base of current stack frame
  FLAGS/RFLAGS — condition codes (Zero, Negative, Carry, Overflow)
  IR         — Instruction Register: currently executing instruction
  MAR        — Memory Address Register: address to read/write
  MDR        — Memory Data Register:   data read from / to write to memory
```

### The Fetch-Decode-Execute Cycle

The fundamental loop every CPU runs forever:

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  1. FETCH    Read instruction at address PC         │
│              IR ← Memory[PC]                        │
│              PC ← PC + instruction_size             │
│                                                     │
│  2. DECODE   Control unit interprets IR             │
│              Identifies: opcode, operands, mode     │
│                                                     │
│  3. EXECUTE  ALU performs the operation             │
│              Result written to register or memory   │
│                                                     │
│  4. WRITEBACK Store result (in pipelined CPUs       │
│               this is a separate stage)             │
│                                                     │
│  → repeat forever (until halt/interrupt)            │
└─────────────────────────────────────────────────────┘
```

### The ALU

```
Inputs:  two operands (A, B) + operation select
Outputs: result + condition flags

Operations:
  Arithmetic:  ADD, SUB, MUL, DIV, NEG, INC, DEC
  Logical:     AND, OR, XOR, NOT
  Shift/Rotate: SHL, SHR, SAR, ROL, ROR
  Compare:      CMP (sets flags without storing result)

Flags set after most ALU operations:
  Z (Zero):     result == 0
  N (Negative): result < 0 (MSB of result)
  C (Carry):    unsigned overflow occurred
  V (oVerflow): signed overflow occurred

Example: CMP rA, rB  computes A - B, sets flags, discards result
         Then: BEQ (branch if equal) checks Z flag
               BLT (branch if less than) checks N and V flags
```

---

## 4. Instruction Set Architecture (ISA)

The ISA is the interface between software and hardware — the set of instructions a CPU understands.

### RISC vs CISC

```
RISC (Reduced Instruction Set Computer)
  Examples: ARM, RISC-V, MIPS, PowerPC
  Philosophy: simple, fixed-length instructions; complexity in compiler
  • Fixed instruction width (32 bits usually)
  • Load/store architecture (only load/store access memory)
  • Many general-purpose registers (16–32+)
  • Simple addressing modes
  • Pipelining-friendly
  • Examples of instructions: ADD, SUB, LOAD, STORE, BRANCH, CMP

CISC (Complex Instruction Set Computer)
  Examples: x86, x86-64 (Intel/AMD)
  Philosophy: complex, variable-length instructions; complexity in hardware
  • Variable instruction width (1–15 bytes in x86)
  • Memory operands allowed in arithmetic instructions
  • Fewer programmer-visible registers (historically)
  • Many addressing modes
  • x86 internally decodes to micro-ops (RISC-like internally!)
```

### Instruction Types

```
Data Transfer:   LOAD, STORE, MOV, PUSH, POP
Arithmetic:      ADD, SUB, MUL, DIV, INC, DEC, NEG
Logical:         AND, OR, XOR, NOT, SHL, SHR
Control Flow:    JMP, BEQ, BNE, BLT, BGT, CALL, RET
Comparison:      CMP, TEST
System:          INT (interrupt), SYSCALL, HLT, NOP
```

### Instruction Encoding

```
Typical RISC 32-bit instruction formats:

R-Type (register operations):
  [6: opcode][5: rs][5: rt][5: rd][5: shamt][6: funct]
  ADD rd, rs, rt

I-Type (immediate / load / store / branch):
  [6: opcode][5: rs][5: rt][16: immediate]
  ADDI rt, rs, 100
  LW   rt, 100(rs)   ← load from address rs+100

J-Type (jump):
  [6: opcode][26: address]
  JMP target_address
```

### Addressing Modes

```
Immediate:      ADD r1, #5          operand is the literal value 5
Register:       ADD r1, r2          operand is the value in r2
Direct:         LOAD r1, [1000]     operand is at memory address 1000
Register Indirect: LOAD r1, [r2]   operand is at address stored in r2
Indexed:        LOAD r1, [r2 + 8]  operand is at address r2+8
PC-Relative:    BEQ +20            branch to PC+20 (used for branches)
Stack:          PUSH r1 / POP r1   implicit SP usage
```

### Function Calls & The Stack

```
Calling convention (e.g., x86-64 System V):
  Arguments 1-6:  rdi, rsi, rdx, rcx, r8, r9
  Arguments 7+:   pushed on stack (right to left)
  Return value:   rax (rdx for 128-bit)
  Caller-saved:   rax, rcx, rdx, rsi, rdi, r8–r11 (caller must save if needed)
  Callee-saved:   rbx, rbp, r12–r15 (callee must restore before returning)

CALL instruction:
  1. PUSH return address (PC+4) onto stack
  2. JMP to function address

RET instruction:
  1. POP return address from stack
  2. JMP to that address

Stack frame layout:
  ┌──────────────┐  ← high addresses
  │  ...caller.. │
  │  arg 7, 8..  │  (if more than 6 args)
  │ return addr  │  ← pushed by CALL
  │  saved rbp   │  ← pushed by PUSH rbp
  │              │  ← rbp points here (frame base)
  │  local vars  │
  │  saved regs  │
  └──────────────┘  ← rsp (stack grows downward)
```

---

## 5. The Memory Hierarchy

Speed and cost are inversely related. The hierarchy exploits **locality** to give the illusion of fast+large+cheap storage.

```
                    Size      Latency      $/GB
  ┌─────────────┐
  │  Registers  │   ~1 KB    0 cycles     —
  ├─────────────┤
  │   L1 Cache  │  32–64 KB  4 cycles     —
  ├─────────────┤
  │   L2 Cache  │  256KB–1MB 12 cycles    —
  ├─────────────┤
  │   L3 Cache  │   4–64 MB  40 cycles    —
  ├─────────────┤
  │     RAM     │   4–64 GB  100 cycles   ~$4
  ├─────────────┤
  │   SSD/NVMe  │  100GB–4TB 100µs        ~$0.10
  ├─────────────┤
  │     HDD     │   1–20 TB  10ms         ~$0.02
  ├─────────────┤
  │   Network/  │  Unlimited  100ms+      ~$0.02
  │    Tape     │
  └─────────────┘

Latency in perspective (if 1 CPU cycle = 1 second):
  L1 cache hit:    4 seconds
  L2 cache hit:   12 seconds
  L3 cache hit:   40 seconds
  RAM access:    100 seconds (~1.7 minutes)
  SSD access:    4 months
  HDD access:    30 years
  Network (CA→NY): 150 years
```

### Locality Principles

```
Temporal Locality:
  If you accessed address X recently, you'll likely access X again soon.
  → Cache keeps recently accessed data
  Example: loop variable accessed every iteration

Spatial Locality:
  If you accessed address X, you'll likely access addresses near X soon.
  → Cache loads a whole cache line (64 bytes) on any miss
  Example: iterating through an array accesses sequential addresses

These two principles are why caches work so well in practice.
```

### Why Memory Layout Matters

```python
# Spatial locality GOOD — sequential access, one cache miss per 64 bytes
for i in range(N):
    total += arr[i]      # arr[i] and arr[i+1] are adjacent in memory

# Spatial locality BAD — column-major access on row-major array
for j in range(N):       # iterating columns first in a C row-major array
    for i in range(N):
        total += A[i][j] # jumps N*sizeof(element) bytes each step → cache miss every access

# Rule: iterate innermost loop over the dimension that is contiguous in memory
```

---

## 6. Pipelining

Pipelining overlaps the execution of multiple instructions, like an assembly line.

### Classic 5-Stage RISC Pipeline

```
Stage:    IF        ID        EX        MEM       WB
          Fetch     Decode    Execute   Memory    Writeback
          instr     read      ALU op    read/     write
          from mem  regs                write     result to
                                        memory    register

Time →
Instr 1:  [IF]──── [ID]──── [EX]──── [MEM]─── [WB]
Instr 2:           [IF]──── [ID]──── [EX]──── [MEM]─── [WB]
Instr 3:                    [IF]──── [ID]──── [EX]──── [MEM]─── [WB]
Instr 4:                             [IF]──── [ID]──── [EX]──── [MEM]─── [WB]
Instr 5:                                      [IF]──── [ID]──── [EX]──── [MEM]─── [WB]

After pipeline fills: one instruction completes per cycle (ideally)
Without pipelining: 5 cycles per instruction
With pipelining: 1 cycle per instruction (throughput, not latency!)
```

### Pipeline Hazards

Situations that prevent the next instruction from executing in the next cycle.

#### 1. Structural Hazards

```
Two instructions need the same hardware resource simultaneously.
Example: single memory port — can't fetch instruction AND load data same cycle.
Solution: Separate instruction memory and data memory (I$ and D$), or stall.
```

#### 2. Data Hazards

```
An instruction needs the result of a previous instruction not yet complete.

RAW (Read After Write) — most common:
  ADD r1, r2, r3    ← writes r1 in WB stage
  SUB r4, r1, r5    ← reads r1 in ID stage → reads stale value!

  Timing without solution:
  ADD:  IF  ID  EX  MEM  WB
  SUB:      IF  ID  EX   ...  ← r1 read in ID, but ADD writes in WB (3 cycles later)

Solutions:
  1. Stall (bubble): insert NOPs until data is ready
     ADD:  IF  ID  EX  MEM  WB
     NOP:      IF  ID  EX   MEM  WB
     NOP:          IF  ID   EX   MEM  WB
     SUB:              IF   ID   EX   ...  ← now reads correct r1
     Cost: 2 stall cycles

  2. Forwarding (bypassing): route result directly from EX/MEM output to EX input
     ADD:  IF  ID  [EX]─────────────────┐ MEM  WB
     SUB:      IF   ID   [EX]←──forward─┘  MEM  WB
     Cost: 0 stall cycles (hardware detects and reroutes)
     Forwards from: EX→EX, MEM→EX, MEM→MEM

  3. Load-Use Hazard (unavoidable 1-cycle stall even with forwarding):
     LW  r1, 0(r2)    ← data not available until END of MEM stage
     ADD r3, r1, r4   ← needs r1 at START of EX stage
     → must stall 1 cycle (load-use hazard)
     Compiler can reorder instructions to fill the load delay slot.

WAR (Write After Read) — hazard in out-of-order CPUs:
  instr i reads r1, instr j (later) writes r1 — if j completes first, i reads wrong value
  Solution: register renaming

WAW (Write After Write):
  instr i writes r1, instr j (later) also writes r1 — wrong final value if i finishes last
  Solution: register renaming
```

#### 3. Control Hazards (Branch Hazards)

```
A branch changes the PC — but we've already fetched the next N instructions!

Without solution:
  BEQ r1, r2, target   ← branch decision made at end of EX (or MEM)
  ADD r3, r4, r5        ← already fetched! do we execute it?
  SUB r6, r7, r8        ← already fetched!
  ...

Solutions:
  1. Stall: wait until branch resolved (2–3 cycle penalty each branch)

  2. Branch Prediction: guess which way the branch goes, keep executing
     If correct: no penalty
     If wrong:   flush pipeline, restart from correct address
                 Penalty = pipeline depth until branch resolves (typically 15–20 cycles in modern CPUs!)

  3. Delayed Branch (MIPS): always execute the instruction AFTER the branch
     (the "branch delay slot") — compiler fills it with useful work or NOP

  4. Branch Prediction Strategies:
     Static:  always predict not-taken / always predict backward-taken
     Dynamic: use Branch History Table (BHT) — 2-bit saturating counter per branch
              00=strong not-taken, 01=weak not-taken, 10=weak taken, 11=strong taken
     Tournament Predictor: uses multiple predictors + selector
     Modern CPUs: >99% accuracy on integer workloads

  Branch Target Buffer (BTB): cache of (branch address → predicted target)
  Return Address Stack (RAS): special predictor for function returns
```

---

## 7. Cache Deep Dive

### Cache Organization

A cache is divided into **sets**, each containing **ways** (lines).

```
Cache parameters:
  Total size   = S sets × W ways × L bytes per line
  Index bits   = log₂(S)
  Offset bits  = log₂(L)
  Tag bits     = address_bits - index_bits - offset_bits

Address breakdown for a cache access:
  ┌─────────────┬─────────────┬──────────────┐
  │     Tag     │    Index    │    Offset    │
  │  (rest of   │  (which     │  (byte       │
  │   address)  │   set?)     │  within line)│
  └─────────────┴─────────────┴──────────────┘
```

### Mapping Policies

```
Direct Mapped (1-way):
  Each memory block maps to exactly ONE cache set.
  + Simple, fast
  - Conflict misses: two frequently used blocks competing for same set
  
Fully Associative (1 set, all ways):
  Any memory block can go in any cache line.
  + No conflict misses
  - Expensive: must search all lines in parallel
  - Only practical for small caches (TLBs)

N-Way Set Associative (most common):
  S sets, each with N ways. Block maps to one set, can go in any of N ways.
  + Balance of speed and conflict miss reduction
  - Typical: 4-way, 8-way, 16-way

Example — 4-way set associative, 64-byte lines, 32KB total:
  Number of sets = 32KB / (4 ways × 64 bytes) = 128 sets
  Offset bits = log₂(64) = 6
  Index bits  = log₂(128) = 7
  Tag bits    = 64 - 7 - 6 = 51 bits
```

### Replacement Policies

```
LRU (Least Recently Used):  evict the line not used for the longest time
  + Best hit rate in practice
  - Expensive to implement exactly for high associativity (use pseudo-LRU)

FIFO: evict the oldest loaded line (regardless of usage)
  - Can evict frequently used lines

Random: evict a random line
  + Simple hardware
  - Surprisingly competitive with LRU for large caches

LFU (Least Frequently Used): evict the least accessed line
  - Doesn't handle changing access patterns well
```

### Write Policies

```
Write-Hit (data is in cache):
  Write-Through: write to cache AND memory simultaneously
    + Memory always consistent
    - Every write goes to slow memory
  Write-Back: write only to cache; write to memory when line is evicted
    + Much faster (most writes stay in cache)
    - Memory may be stale (dirty bit tracks this)
    - More complex hardware

Write-Miss (data is NOT in cache):
  Write-Allocate: load the block into cache, then write to it
    → Paired with write-back (makes sense to load if you'll write more)
  No-Write-Allocate: write directly to memory, don't load into cache
    → Paired with write-through

Typical combination:
  Write-back + Write-allocate  (most CPUs)
  Write-through + No-write-allocate  (some L1 caches, GPUs)
```

### Cache Misses — The 3 Cs

```
Compulsory (Cold) Miss:
  First access to any block — unavoidable.
  Reduction: prefetching.

Capacity Miss:
  Cache is too small to hold all needed data.
  Reduction: bigger cache, better data layout.

Conflict Miss:
  Two blocks compete for the same set, even if cache isn't full.
  Reduction: higher associativity, data padding/alignment.

(4th C) Coherency Miss:
  In multiprocessor systems, another CPU invalidated the line.
  Reduction: cache coherency protocols (MESI).
```

### MESI Cache Coherency Protocol

In multi-core CPUs, each core has its own L1/L2 cache. They must agree on data values.

```
M — Modified:   this cache has the only valid copy; it's been written (dirty)
E — Exclusive:  this cache has the only valid copy; clean (matches memory)
S — Shared:     multiple caches may have valid copies; all clean
I — Invalid:    this cache line is not valid

State transitions (simplified):
  CPU reads,  line not present → fetch from memory/other cache → E or S
  CPU writes, line in S state  → invalidate all other copies → M
  Another CPU reads my M line  → I write back to memory, transition to S
  Another CPU writes           → my S/E line transitions to I

False Sharing:
  Two CPUs access DIFFERENT variables that happen to be on the SAME cache line.
  CPU A writes var_a, CPU B writes var_b — they invalidate each other's lines constantly.
  Fix: pad structures so hot variables are on separate cache lines (64-byte alignment).

  struct bad  { int a; int b; };           // a and b on same cache line
  struct good { int a; char pad[60]; int b; }; // a and b on different lines
```

---

## 8. Virtual Memory

Virtual memory gives each process the illusion of having its own large, contiguous address space.

### Why Virtual Memory?

```
1. Isolation: process A cannot access process B's memory
2. Larger-than-RAM programs: OS pages out unused parts to disk
3. Simplified loading: every process can be loaded at the same virtual address
4. Shared memory: map same physical page into multiple virtual spaces
5. Memory-mapped files: files accessed like arrays
```

### Address Translation

```
CPU generates Virtual Address (VA)
  ↓
MMU (Memory Management Unit) translates using Page Table
  ↓
Physical Address (PA) used to access RAM

Virtual address breakdown (x86-64, 4KB pages):
  ┌────────────┬────────────┬────────────┬────────────┬──────────────┐
  │  PML4 idx  │  PDPT idx  │   PD idx   │   PT idx   │    Offset    │
  │   9 bits   │   9 bits   │   9 bits   │   9 bits   │   12 bits    │
  └────────────┴────────────┴────────────┴────────────┴──────────────┘

  Page offset = 12 bits → page size = 2¹² = 4096 bytes = 4 KB
  Total virtual address space = 2⁴⁸ = 256 TB (with 48-bit addressing)

Multi-level page table walk (4 levels in x86-64):
  CR3 register → PML4 table → PDPT → Page Directory → Page Table → Physical Page
  Each level indexes into a 4KB table of 512 entries (8 bytes each)
```

### TLB — Translation Lookaside Buffer

```
A page table walk = 4 memory accesses = very slow.
TLB: a small, fully associative cache of recent virtual→physical translations.

  TLB hit:  ~1 cycle overhead (most accesses)
  TLB miss: must walk page table (4 memory accesses, ~hundreds of cycles)

TLB sizes: typically 32–2048 entries
  With 4KB pages and 64-entry TLB: covers 64 × 4KB = 256KB of address space
  With 2MB huge pages and 64-entry TLB: covers 128MB → far fewer TLB misses

Context switch: OS flushes TLB (or uses Address Space IDs to avoid flush)
```

### Page Faults

```
Access a virtual address whose page is not in RAM:
  1. MMU raises a page fault exception
  2. OS page fault handler runs
  3. OS finds the page on disk (in swap space)
  4. OS loads page into a free physical frame (evicting another if needed)
  5. OS updates the page table
  6. CPU restarts the faulting instruction

Page replacement algorithms:
  LRU:   evict least recently used page (ideal but expensive)
  Clock: approximate LRU using a reference bit (efficient)
  Working Set: evict pages outside the process's recent working set
```

### Page Table Entry

```
┌───┬───┬───┬───┬───┬───┬─────────────────────────────────────────┐
│ P │ W │ U │ A │ D │ PS│        Physical Frame Number            │
└───┴───┴───┴───┴───┴───┴─────────────────────────────────────────┘
  P = Present (1 = in RAM, 0 = page fault on access)
  W = Writeable
  U = User accessible (0 = kernel only)
  A = Accessed (set by MMU on read/write)
  D = Dirty (set by MMU on write)
  PS = Page Size (0 = 4KB, 1 = 2MB huge page)
```

---

## 9. I/O & Buses

### How the CPU Communicates with Devices

```
Three methods:

1. Port-Mapped I/O (x86 legacy):
   Special IN/OUT instructions access a separate I/O address space.
   OUT 0x3F8, AL  ← write byte to COM1 serial port
   
2. Memory-Mapped I/O (MMIO, modern standard):
   Device registers appear as memory addresses.
   CPU reads/writes to those addresses; no special instructions needed.
   *(volatile uint32_t*)0xFEC00000 = value;  ← write to APIC register
   Volatile keyword prevents compiler from caching or reordering the access.

3. DMA (Direct Memory Access):
   CPU programs a DMA controller with: source, destination, length.
   DMA controller transfers data directly between device and RAM without CPU involvement.
   CPU receives an interrupt when transfer is complete.
   Used for: disk I/O, network cards, USB, GPU transfers.
   Without DMA: CPU cycles wasted polling/moving every byte.
```

### Interrupts

```
Hardware interrupt: device signals CPU that it needs attention.
  1. Device asserts interrupt line (IRQ)
  2. CPU finishes current instruction
  3. CPU saves PC + FLAGS on stack
  4. CPU jumps to Interrupt Service Routine (ISR) via Interrupt Vector Table
  5. ISR handles the event (e.g., read keyboard buffer)
  6. ISR executes IRET → restores PC + FLAGS, resumes normal execution

Software interrupt: instruction that triggers OS call (syscall, INT 0x80)

Interrupt latency: time from interrupt signal to first ISR instruction
  Modern CPUs: ~100 ns typical

Interrupt priority:
  Non-Maskable Interrupt (NMI): cannot be disabled, used for critical hardware errors
  Maskable interrupts: can be disabled (IF flag in x86) for critical sections
```

### The Bus System

```
System Bus components:
  Address bus:  CPU sends address (where to read/write) — unidirectional
  Data bus:     transfers actual data — bidirectional
  Control bus:  read/write signals, interrupt lines, bus request/grant

Modern interconnects:
  PCIe (PCI Express): GPU, NVMe SSDs, network cards
    PCIe 4.0 x16: 32 GB/s bidirectional
  SATA:  traditional hard drives and SSDs (up to 600 MB/s)
  USB:   peripherals (USB 3.2: up to 20 Gb/s)
  DDR5:  CPU to RAM (up to ~100 GB/s total bandwidth)
  QPI/UPI: Intel CPU-to-CPU in multi-socket systems
  Infinity Fabric: AMD inter-die and inter-socket

Memory bus bandwidth:
  Single DDR5-6400 channel: 51.2 GB/s
  Modern CPUs: 2–8 channels → 100–400 GB/s peak
```

---

## 10. Parallelism & Modern CPUs

### Instruction-Level Parallelism (ILP)

```
Superscalar Execution:
  Multiple execution units operating in parallel each cycle.
  Modern CPUs issue 3–6 instructions per cycle (IPC).
  Requires: no data dependencies between concurrent instructions.

Out-of-Order Execution (OoO):
  CPU reorders instructions to avoid stalls (within the constraint of correct results).
  
  Key structures:
  ROB (Reorder Buffer):    tracks all in-flight instructions in program order
                           ensures results are committed in order
  Reservation Stations:    instructions wait here until operands are ready
  Register Renaming:       maps architectural registers (r0-r15) to physical
                           registers (many more) — eliminates false WAR/WAW hazards
  Common Data Bus (CDB):   broadcasts results to waiting instructions

  Pipeline stages in modern OoO CPU:
  Fetch → Decode → Rename → Dispatch → Issue → Execute → Complete → Retire
```

### SIMD — Single Instruction, Multiple Data

```
One instruction operates on multiple data elements simultaneously.

  Normal (scalar):  ADD r1, r2, r3   → 1 addition
  SIMD (vector):    VADDPS ymm0, ymm1, ymm2  → 8 float additions at once

x86 SIMD evolution:
  MMX:    64-bit registers  (8×8-bit or 4×16-bit)
  SSE:    128-bit (xmm)     16 bytes = 4 floats or 2 doubles
  AVX:    256-bit (ymm)     32 bytes = 8 floats or 4 doubles
  AVX-512: 512-bit (zmm)   64 bytes = 16 floats or 8 doubles

Throughput gain example (AVX, 8-wide float):
  Matrix multiply 1000×1000 float32:
  Scalar:   1 billion multiplications → ~300ms
  AVX:      8× throughput → ~37ms
  AVX + multiple cores + cache-friendly layout → ~4ms

Auto-vectorization: compiler can often generate SIMD automatically if:
  - Loop has no data dependencies between iterations
  - Data is properly aligned (use alignas(32) or __attribute__((aligned(32))))
  - No function calls inside the loop (or they're inlined)
```

### Multi-Core & Shared Memory

```
UMA (Uniform Memory Access):
  All cores share the same memory controller.
  All memory accesses take the same time from any core.
  Typical: desktop/laptop CPUs (up to ~16 cores)

NUMA (Non-Uniform Memory Access):
  Multiple CPU sockets, each with their own local memory.
  Accessing local memory: ~100 cycles
  Accessing remote socket memory: ~300 cycles (crosses interconnect)
  OS/runtime should allocate memory on the NUMA node where the thread runs.
  
  Topology (2-socket server):
    [CPU 0]──[Local RAM 0]    [CPU 1]──[Local RAM 1]
        └──────QPI/UPI──────────┘

CPU Affinity: pin threads to specific cores to avoid migration overhead and improve cache reuse.
```

### Multithreading in Hardware

```
Coarse-grained Multithreading:
  Switch thread when current thread stalls (e.g., cache miss).
  + Hides latency
  - Slow to switch

Fine-grained Multithreading:
  Switch thread every cycle.
  + Better latency hiding
  - Each thread runs slower (sharing pipeline resources)

Simultaneous Multithreading (SMT) / Hyper-Threading (Intel):
  Multiple threads share execution units WITHIN a single cycle.
  Each logical CPU has its own register file + PC + reorder buffer.
  Shared: execution units, caches, TLB.
  + Better utilization of execution units (one thread uses ALU while another waits on memory)
  - Slightly more complex; can reduce single-thread performance
  - Security: Spectre/Meltdown attacks exploit shared hardware state
```

### GPU Architecture (Overview)

```
CPU vs GPU philosophy:
  CPU: optimized for low-latency serial execution
       Few powerful cores (8–64), large caches, OoO execution, branch prediction
  GPU: optimized for high-throughput parallel execution
       Thousands of simple cores, small caches, in-order execution

NVIDIA terminology:
  SM (Streaming Multiprocessor): ~128 CUDA cores + shared memory + warp scheduler
  Warp: 32 threads executing the SAME instruction (SIMT — Single Instruction Multiple Threads)
  Thread Block: group of threads sharing shared memory, run on one SM
  Grid: collection of thread blocks

Memory hierarchy:
  Registers:       ~1 cycle (per thread, private)
  Shared Memory:   ~5 cycles (per thread block, L1-like)
  L2 Cache:        ~200 cycles
  Global Memory:   ~600 cycles (GDDR6 VRAM)

Key GPU constraint: WARP DIVERGENCE
  if (threadIdx.x % 2 == 0) { ... } else { ... }
  → Both branches execute serially; inactive threads are masked
  → Cuts throughput in half
  → Avoid conditional divergence in inner loops
```

---

## 11. Performance & Optimization

### Amdahl's Law

```
With parallelism applied to a fraction P of the program:

  Speedup = 1 / ((1 - P) + P/N)

Where N = number of processors.

P = 0.9 (90% parallelizable), N = 16 cores:
  Speedup = 1 / (0.1 + 0.9/16) = 1 / 0.156 ≈ 6.4×

P = 0.9, N = ∞:
  Max speedup = 1 / (1 - 0.9) = 10×

Lesson: the serial portion dominates at high parallelism.
        Even 5% serial code limits max speedup to 20× regardless of core count.
```

### Key Performance Metrics

```
IPC:   Instructions Per Cycle  (higher = better)
       Modern CPUs: ~3–5 IPC on real workloads

CPI:   Cycles Per Instruction = 1/IPC

CPU Time = Instruction Count × CPI × Clock Period

MIPS:  Millions of Instructions Per Second  (largely obsolete/misleading)
FLOPS: Floating Point Operations Per Second  (used for HPC)
       A100 GPU: 312 TFLOPS (FP16), 19.5 TFLOPS (FP64)

Memory bandwidth:  GB/s (often the bottleneck, not compute)
Arithmetic intensity: FLOPS per byte of memory accessed
  → Low intensity = memory-bound
  → High intensity = compute-bound
  → Roofline model plots performance against intensity
```

### The Roofline Model

```
Peak performance = min(compute_peak, bandwidth × arithmetic_intensity)

         ┌────────────────────────────────────────
Peak     │                  ┌────────────────────  ← compute bound
GFLOPS   │                 /
         │                /  ← memory bound
         │               /
         │              /
         └─────────────┴─────────────────────────
                       │
               Ridge point = compute_peak / bandwidth
               (operations per byte where you become compute-bound)
```

### Hardware Performance Counters

Modern CPUs expose counters for profiling:

```
Instructions retired        — actual instructions completed
Cycles                      — total cycles elapsed
L1/L2/L3 cache misses       — cache miss counts
Branch mispredictions       — wasted cycles from wrong guesses
TLB misses                  — page table walks needed
Instructions per cycle      — IPC = instructions/cycles
Memory bandwidth used       — bytes read/written to RAM
Stall cycles                — cycles CPU was waiting for memory

Tools:
  Linux:  perf stat ./program     (overview)
          perf record + perf report (hotspot profiling)
  Intel:  VTune
  AMD:    uProf
  macOS:  Instruments
```

### Code-Level Optimization Principles

```
1. Cache-friendly data layout
   • Prefer arrays of structs of scalars to structs of arrays when accessing fields together
   • Access arrays sequentially (spatial locality)
   • Keep frequently accessed data hot in cache (temporal locality)
   • Separate hot (frequently accessed) fields from cold (rarely accessed) fields in structs

2. Avoid branch mispredictions
   • Sort data before processing to make branches predictable
   • Use branchless arithmetic: abs, min, max without if
   • Profile-guided optimization lets the compiler optimize common paths

3. Avoid false sharing in multithreaded code
   • Pad per-thread data to 64-byte cache line boundaries

4. Reduce memory bandwidth
   • Process data in blocks that fit in cache (cache blocking / tiling)
   • Use smaller data types when precision allows (float vs double, int16 vs int32)

5. Enable SIMD
   • Use restrict keyword (C) or noalias to tell compiler pointers don't alias
   • Align data: alignas(32) float arr[N];
   • Use intrinsics or libraries (Eigen, BLAS) when auto-vectorization falls short

6. Compiler flags
   -O2/-O3:      general optimization
   -march=native: use all CPU features of the current machine
   -ffast-math:  allow reordering of float ops (may change results slightly)
   -funroll-loops: unroll loops to reduce branch overhead
```

---

## 12. Quick Reference

### CPU Pipeline Hazard Summary

| Hazard | Cause | Solution |
|--------|-------|----------|
| Structural | Resource conflict | Stall or duplicate hardware |
| RAW (data) | Read before write completes | Forwarding or stall |
| Load-use | Load followed immediately by use | 1-cycle stall (unavoidable) |
| Control (branch) | Branch changes PC | Prediction + flush, or stall |
| WAR, WAW | Out-of-order CPUs | Register renaming |

### Cache Miss Types

| Type | Cause | Fix |
|------|-------|-----|
| Compulsory | First access | Prefetch |
| Capacity | Cache too small | Bigger cache or smaller working set |
| Conflict | Set contention | Higher associativity or data padding |
| Coherency | Other core invalidated | MESI protocol |

### Memory Latency Quick Reference

| Level | Latency | Size |
|-------|---------|------|
| Register | 0 cycles | ~1 KB |
| L1 Cache | 4 cycles | 32–64 KB |
| L2 Cache | 12 cycles | 256 KB–1 MB |
| L3 Cache | 40 cycles | 4–64 MB |
| DRAM | ~100 cycles | 4–64 GB |
| NVMe SSD | ~100,000 cycles | 100 GB–4 TB |
| HDD | ~40,000,000 cycles | 1–20 TB |

### Key Formulas

```
CPU Execution Time  = Instruction_Count × CPI × Clock_Period
CPI                 = 1/IPC (ideally 1.0, typically 0.25–0.5 for OoO CPUs)
Cache size          = Sets × Ways × Line_size
Address bits        = Tag_bits + Index_bits + Offset_bits
Offset bits         = log₂(Line_size)
Index bits          = log₂(Sets)
Amdahl speedup      = 1 / ((1-P) + P/N)
Bandwidth needed    = Working_set_size × Access_frequency
Arithmetic intensity= FLOPS / bytes_transferred
```

### ISA Comparison

| Feature | x86-64 | ARM64 (AArch64) | RISC-V |
|---------|--------|-----------------|--------|
| Type | CISC (micro-ops internally) | RISC | RISC |
| Instruction width | Variable (1–15B) | Fixed (32-bit) | Fixed (32-bit, 16-bit compressed) |
| Registers | 16 GP + 16 SIMD | 31 GP + 32 SIMD | 32 GP + 32 FP |
| Memory ops | Register or memory operands | Load/store only | Load/store only |
| Endianness | Little | Bi (usually little) | Bi (usually little) |
| Used in | Desktop/server (Intel/AMD) | Mobile, Apple Silicon, servers | Embedded, research, growing |

### Common Processor Specs (2024)

| CPU | Cores | L3 Cache | Process | TDP |
|-----|-------|----------|---------|-----|
| Intel Core i9-14900K | 24 (8P+16E) | 36 MB | Intel 7 (10nm) | 125W |
| AMD Ryzen 9 7950X | 16 | 64 MB | TSMC 5nm | 170W |
| Apple M3 Max | 16 (12P+4E) | 48 MB | TSMC 3nm | ~92W |
| AMD EPYC 9654 (server) | 96 | 384 MB | TSMC 5nm | 360W |
| ARM Cortex-A78 (mobile) | 1–4 | 4–8 MB | 5nm | ~3W |

---

*For deeper study: "Computer Organization and Design" (Patterson & Hennessy) for fundamentals. "Computer Architecture: A Quantitative Approach" (Hennessy & Patterson) for advanced topics. "What Every Programmer Should Know About Memory" (Drepper) for cache/memory optimization.*