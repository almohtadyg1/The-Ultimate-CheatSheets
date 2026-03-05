# Compilers: Zero to Expert Cheat Sheet

---

## 1. What is a Compiler?

A **compiler** is a program that translates source code written in one language (source language) into another language (target language), typically machine code or an intermediate form, while preserving the program's meaning.

```
Source Code  ──►  [ COMPILER ]  ──►  Target Code
  (text)                              (binary / IR / other language)
```

### Core Properties
| Property | Description |
|---|---|
| **Correctness** | Output program must behave identically to source |
| **Efficiency** | Generated code should run fast / use little memory |
| **Diagnostics** | Useful error messages for the programmer |
| **Speed** | Compilation itself should be reasonably fast |

---

## 2. Compiler vs Interpreter vs JIT

| Mode | How it works | Examples | Tradeoff |
|---|---|---|---|
| **Compiler** | Translate entire source → machine code ahead of time | GCC, Clang, rustc | Fast runtime, slow startup |
| **Interpreter** | Execute source line-by-line at runtime | CPython, Ruby MRI | Portable, slower execution |
| **JIT Compiler** | Compile hot paths to native code at runtime | JVM HotSpot, V8, PyPy | Best of both, complex to build |
| **AOT Compiler** | Compile to native code before deployment | GraalVM native-image, Go | Fast startup, large binary |
| **Transpiler** | Source → different source language | Babel, TypeScript | Language interop |

---

## 3. The Big Picture: Compilation Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                        SOURCE PROGRAM                           │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                ┌───────────▼───────────┐
                │   LEXICAL ANALYSIS    │  → Token stream
                │   (Lexer / Scanner)   │
                └───────────┬───────────┘
                            │
                ┌───────────▼───────────┐
                │   SYNTAX ANALYSIS     │  → Parse Tree / AST
                │   (Parser)            │
                └───────────┬───────────┘
                            │
                ┌───────────▼───────────┐
                │   SEMANTIC ANALYSIS   │  → Annotated AST
                │   (Type checker etc.) │
                └───────────┬───────────┘
                            │
                ┌───────────▼───────────┐
                │   IR GENERATION       │  → Three-address code,
                │                       │    LLVM IR, bytecode…
                └───────────┬───────────┘
                            │
                ┌───────────▼───────────┐
                │   OPTIMIZATION        │  → Optimized IR
                │   (Middle end)        │
                └───────────┬───────────┘
                            │
                ┌───────────▼───────────┐
                │   CODE GENERATION     │  → Assembly / Machine code
                │   (Back end)          │
                └───────────┬───────────┘
                            │
                ┌───────────▼───────────┐
                │   LINKER / ASSEMBLER  │  → Executable binary
                └───────────────────────┘
```

> **Front end** = Lexer + Parser + Semantic Analysis  
> **Middle end** = IR + Optimization  
> **Back end** = Code Generation + Register Allocation + Assembly  

---

## 4. Phase 1 — Lexical Analysis (Scanning)

The **lexer** reads raw characters and groups them into **tokens** (the smallest meaningful units).

### What is a Token?
```
<TOKEN_TYPE,  VALUE>
<ID,         "velocity">
<ASSIGN,     "=">
<NUM,        42>
<SEMICOLON,  ";">
```

### Token Categories
| Category | Examples |
|---|---|
| Keywords | `if`, `while`, `return`, `int` |
| Identifiers | variable/function names |
| Literals | `42`, `3.14`, `"hello"`, `true` |
| Operators | `+`, `-`, `*`, `==`, `&&` |
| Punctuation | `{`, `}`, `(`, `)`, `;` |
| Whitespace / Comments | usually discarded |

### Regular Expressions → DFA

Tokens are defined using **regular expressions**, then converted:

```
Regex  ──►  NFA (Thompson's construction)  ──►  DFA (subset construction)  ──►  Minimized DFA
```

**Identifier regex:** `[a-zA-Z_][a-zA-Z0-9_]*`  
**Integer regex:** `[0-9]+`  
**Float regex:** `[0-9]+\.[0-9]*([eE][+-]?[0-9]+)?`

### Maximal Munch Rule
Always consume the **longest possible** token.  
`<=` is one token, not `<` then `=`.

### Handling Conflicts
If two patterns match the same string, priority is given by **order of definition** (keywords before identifiers).

### Tools
- **flex** / **lex** — generates C lexers from regex specs
- **ANTLR** — generates lexers + parsers in multiple languages
- **Alex** — Haskell lexer generator
- Hand-rolled: use a state machine with a switch or jump table

---

## 5. Phase 2 — Syntax Analysis (Parsing)

The **parser** takes a token stream and builds a **parse tree** or **Abstract Syntax Tree (AST)** according to the language grammar.

### Context-Free Grammars (CFGs)
A CFG is a 4-tuple **G = (V, Σ, R, S)** where:
- **V** = non-terminal symbols
- **Σ** = terminal symbols (tokens)
- **R** = production rules
- **S** = start symbol

```
// Arithmetic expression grammar
expr   → expr '+' term | expr '-' term | term
term   → term '*' factor | term '/' factor | factor
factor → '(' expr ')' | NUMBER | ID
```

### Parse Tree vs AST
```
Parse Tree (concrete):          AST (abstract):
      expr                           +
     / | \                          / \
  expr  +  term                   3    *
   |       / | \                      / \
  term   term * factor               4   5
   |      |       |
  factor factor   5
   |      |
   3      4
```
The AST **discards** redundant nodes (parentheses, separators) while keeping semantic structure.

### Ambiguity
A grammar is **ambiguous** if a string has more than one parse tree.  
Fix with **precedence rules** or grammar rewriting.

```
// Ambiguous:
expr → expr '+' expr | expr '*' expr | NUM

// Unambiguous (encodes precedence):
expr → expr '+' term | term
term → term '*' factor | factor
```

### AST Node Types (example)
```
BinaryOp  { op: '+', left: Expr, right: Expr }
UnaryOp   { op: '-', operand: Expr }
IfStmt    { cond: Expr, then: Stmt, else: Stmt? }
FuncDecl  { name: String, params: [Param], body: Block }
VarDecl   { name: String, type: Type, init: Expr? }
```

---

## 6. Phase 3 — Semantic Analysis

Checks that the program is **meaningful**, not just syntactically correct.

### Key Tasks

| Task | Description |
|---|---|
| **Scope resolution** | Each identifier resolves to exactly one declaration |
| **Type checking** | Operations applied to compatible types |
| **Flow analysis** | Variables initialized before use |
| **Overload resolution** | Correct function variant selected |
| **Coercions** | Implicit type conversions inserted |

### Scope Rules
```
// Lexical (static) scoping — most languages
{
  int x = 1;
  {
    int x = 2;  // shadows outer x
    print(x);   // prints 2
  }
  print(x);     // prints 1
}
```

### Type Checking
- **Static** — checked at compile time (C, Java, Rust, Haskell)
- **Dynamic** — checked at runtime (Python, JavaScript, Ruby)
- **Gradual** — optional static types (TypeScript, mypy)

#### Type Rules (notation)
```
Γ ⊢ e : T          "In environment Γ, expression e has type T"

Γ ⊢ e1 : Int    Γ ⊢ e2 : Int
─────────────────────────────
      Γ ⊢ e1 + e2 : Int
```

### Type Inference (Hindley-Milner)
Algorithm **W** infers types without annotations:
1. Assign fresh type variables to unknowns
2. Generate **constraints** from usage
3. Solve via **unification**

```
let f x = x + 1
// x must be Int (used with +1), so f : Int → Int
```

---

## 7. Phase 4 — Intermediate Representation (IR)

IR sits between the high-level AST and low-level machine code. A good IR is:
- Easy to generate from AST
- Easy to optimize
- Easy to lower to multiple targets

### Three-Address Code (TAC)
Every instruction has at most **3 addresses** (2 operands, 1 result):

```
t1 = a * b
t2 = t1 + c
if t2 > 0 goto L1
x = -t2
goto L2
L1: x = t2
L2: ...
```

### LLVM IR (real-world example)
```llvm
define i32 @add(i32 %a, i32 %b) {
entry:
  %sum = add i32 %a, %b
  ret i32 %sum
}
```
LLVM IR is **typed**, **SSA form**, and **infinite registers**.

### Common IR Variants
| IR | Used In | Notes |
|---|---|---|
| Three-address code | Textbook | Quadruples / triples |
| SSA | GCC, LLVM, V8 | Each var assigned once |
| CPS | Functional langs | Continuation-passing style |
| Bytecode | JVM, .NET CLR | Stack-based virtual machine |
| ANF | MLton, Haskell | A-normal form |
| RTL | GCC backend | Register transfer language |

### Basic Blocks & Control Flow Graph (CFG)
A **basic block** is a maximal sequence of instructions with:
- One entry point (only first instruction is a branch target)
- One exit point (only last instruction is a branch)

```
       [Entry]
          |
     [Block 1: x = a + b]
          |
    [Condition: x > 0]
       /        \
[Block 2]    [Block 3]
  x = x*2      x = -x
       \        /
     [Block 4: return x]
```

---

## 8. Phase 5 — Optimization

Transformations that improve performance (speed, code size, memory) **without changing semantics**.

### Classification
| Level | Examples |
|---|---|
| **Peephole** | Replace `x * 2` → `x << 1`, remove redundant loads |
| **Local** | Within a basic block |
| **Global** | Across basic blocks in a function |
| **Interprocedural** | Across function boundaries |
| **Loop** | Transformations targeting loops |

### Essential Optimizations

#### Constant Folding
```
x = 2 + 3  →  x = 5          (at compile time)
```

#### Constant Propagation
```
x = 5
y = x + 3  →  y = 8
```

#### Dead Code Elimination (DCE)
```
x = expensive_calc()   // x never used → remove
```

#### Common Subexpression Elimination (CSE)
```
// Before:
a = b * c + 1
d = b * c - 1

// After (reuse b*c):
t = b * c
a = t + 1
d = t - 1
```

#### Inlining
Replace a function call with the function body. Removes call overhead; enables further optimization.

#### Loop Optimizations
| Optimization | Description |
|---|---|
| **Loop-invariant code motion (LICM)** | Move unchanged computations outside the loop |
| **Loop unrolling** | Replicate loop body N times, reduce branch overhead |
| **Loop fusion** | Combine two adjacent loops into one |
| **Loop fission** | Split one loop into multiple (improve cache behavior) |
| **Strength reduction** | Replace expensive ops (`*`) with cheaper ones (`+`) |
| **Vectorization (SIMD)** | Execute multiple iterations in parallel using vector instructions |

#### Tail Call Optimization (TCO)
```
// Recursive call in tail position:
int fact(int n, int acc) {
  if (n == 0) return acc;
  return fact(n-1, n*acc);  // ← tail call
}
// Compiler converts to a loop — no stack growth!
```

#### Alias Analysis
Determines if two pointers can point to the same memory. Enables more aggressive optimization.

#### Escape Analysis
Determines if an object's reference escapes a function. If not, it can be stack-allocated (no GC pressure).

### Optimization Passes (LLVM example pipeline)
```
mem2reg → instcombine → gvn → licm → loop-unroll → dce → inliner
```

---

## 9. Phase 6 — Code Generation

Translate optimized IR into **target machine instructions**.

### Key Steps
1. **Instruction selection** — Map IR ops to machine instructions
2. **Instruction scheduling** — Order instructions to hide latency
3. **Register allocation** — Assign variables to physical registers

### Instruction Selection
**Tree pattern matching** (ISEL):
```
IR:  t1 = load(base + offset * 4)

x86: mov eax, [rbx + rcx*4]    ← one instruction covers the tree!
```
Algorithms: **BURS** (Bottom-Up Rewriting System), **DAG tiling**.

### Instruction Scheduling
Modern CPUs have pipelines — reorder instructions to avoid stalls:
```
// Naive (data hazard stall):
load r1, [addr]
add  r2, r1, r3   // must wait for r1 to load

// Scheduled (insert independent work):
load r1, [addr]
add  r4, r5, r6   // independent — fills the slot
add  r2, r1, r3
```

### Calling Conventions
Defines how functions pass arguments and return values.

| Convention | Platform | Args in | Return |
|---|---|---|---|
| **System V AMD64 ABI** | Linux/macOS x86-64 | rdi, rsi, rdx, rcx, r8, r9 | rax |
| **Windows x64** | Windows x86-64 | rcx, rdx, r8, r9 | rax |
| **ARM64 (AAPCS64)** | ARM Linux/macOS | x0–x7 | x0 |

**Callee-saved registers** (preserved across calls): rbx, rbp, r12–r15  
**Caller-saved registers** (may be clobbered): rax, rcx, rdx, rsi, rdi, r8–r11

---

## 10. Symbol Tables

A **symbol table** maps identifiers to their attributes (type, scope, location).

### Structure
```
SymbolEntry {
  name:    String
  kind:    Variable | Function | Type | Label
  type:    TypeInfo
  scope:   ScopeLevel
  address: MemLocation (stack offset, global addr, register)
}
```

### Scope Stack
```
Global scope:   { printf, main }
  Function main: { argc, argv, x, y }
    Block:       { temp, i }
```
Implemented as a **stack of hash tables**. On entering a scope, push a new table. On exit, pop.

### Hashing
- Open addressing or chaining hash maps
- Must handle shadowing: inner scope can redefine outer names
- Lookup: search from innermost outward

---

## 11. Type Systems

### Fundamental Concepts

| Concept | Meaning |
|---|---|
| **Type** | A set of values + operations valid on them |
| **Subtyping** | S is a subtype of T if an S can be used wherever T is expected |
| **Polymorphism** | One function works for multiple types |
| **Covariance** | If A <: B then `F<A>` <: `F<B>` |
| **Contravariance** | If A <: B then `F<B>` <: `F<A>` |

### Polymorphism Varieties
| Kind | Example |
|---|---|
| **Parametric** | `List<T>`, generics, `∀T. T → T` |
| **Ad-hoc (overloading)** | `+(int,int)` and `+(float,float)` |
| **Subtype** | OOP inheritance, `Animal` ref holds `Dog` |
| **Row polymorphism** | Structural typing, TypeScript interfaces |

### Type Inference: Unification
```
// Constraint:  T1 = Int → T2,  T1 = T3 → Bool
// Unify:       Int = T3  →  T3 := Int
//              T2 = Bool →  T2 := Bool
// Result:      T1 = Int → Bool
```

### Linear / Affine Types (Rust)
Each value is **used exactly once** (linear) or **at most once** (affine).  
Enforces memory safety without GC: the **borrow checker** tracks ownership.

```rust
let s = String::from("hello");
let t = s;          // ownership moved
// println!("{}", s); // compile error: s moved
```

---

## 12. Memory Layout & Runtime

### Stack Frame Layout (x86-64)
```
High address
┌─────────────────┐
│  Caller's frame │
├─────────────────┤  ← old rbp
│  Saved rbp      │
├─────────────────┤  ← rbp (frame pointer)
│  Local var 1    │
│  Local var 2    │
│  ...            │
├─────────────────┤
│  Spilled regs   │
├─────────────────┤  ← rsp (stack pointer)
Low address
```

### Memory Segments
```
┌─────────────┐  High address
│   Stack     │  Grows ↓  (local vars, frames)
│      ↓      │
│             │
│      ↑      │
│    Heap     │  Grows ↑  (malloc/new)
├─────────────┤
│    BSS      │  Uninitialized globals
├─────────────┤
│    Data     │  Initialized globals/statics
├─────────────┤
│    Text     │  Executable code (read-only)
└─────────────┘  Low address (0x400000 typically)
```

### Garbage Collection Algorithms
| Algorithm | How | Pro | Con |
|---|---|---|---|
| **Reference counting** | Count pointers to each object | Simple, immediate | Cycles leaked (need cycle detector) |
| **Mark & sweep** | Mark live from roots, sweep dead | Handles cycles | Stop-the-world pauses |
| **Copying / Cheney** | Copy live objects to new space | Compacts heap | Wastes half memory |
| **Generational** | Separate young/old generations | Fast minor GCs | Tuning required |
| **Tri-color incremental** | Interleave GC with mutator | Reduced pauses | Write barrier overhead |
| **G1 / ZGC / Shenandoah** | Region-based concurrent GC | Sub-ms pauses | Complex implementation |

---

## 13. Grammars Deep Dive

### Chomsky Hierarchy
| Class | Grammar | Machine | Example |
|---|---|---|---|
| Type 0 | Unrestricted | Turing Machine | Any computable language |
| Type 1 | Context-sensitive | LBA | `aⁿbⁿcⁿ` |
| Type 2 | Context-free | Pushdown Automaton | Most PLs (simplified) |
| Type 3 | Regular | Finite Automaton | Tokens / lexing |

### FIRST and FOLLOW Sets
Used to build predictive parsers.

**FIRST(α)** = set of terminals that can begin strings derived from α  
**FOLLOW(A)** = set of terminals that can appear after A in a sentential form

```
expr  → term expr'
expr' → '+' term expr' | ε
term  → factor term'
term' → '*' factor term' | ε
factor → '(' expr ')' | ID | NUM

FIRST(expr)   = { '(', ID, NUM }
FOLLOW(expr') = { ')', $ }
```

### LL(1) Grammar Conditions
A grammar is **LL(1)** if for every production A → α | β:
1. FIRST(α) ∩ FIRST(β) = ∅
2. If ε ∈ FIRST(α), then FIRST(β) ∩ FOLLOW(A) = ∅

### Left Recursion Elimination
```
// Left recursive (LL parsers can't handle):
A → A α | β

// Transformed:
A  → β A'
A' → α A' | ε
```

### Left Factoring
```
// Ambiguous lookahead:
stmt → if expr then stmt
     | if expr then stmt else stmt

// Factored:
stmt    → if expr then stmt stmt'
stmt'   → else stmt | ε
```

---

## 14. Parsing Algorithms

### Top-Down Parsing

#### Recursive Descent
One function per non-terminal. Reads left-to-right, constructs tree top-down.

```python
def parse_expr():
    left = parse_term()
    while current_token() in ('+', '-'):
        op = consume()
        right = parse_term()
        left = BinaryOp(op, left, right)
    return left
```

#### LL(k) Table-Driven
Build a **parsing table** M[A, a] — given non-terminal A and lookahead a, which production to use.

```
       |  '+'  |  '*'  |  '('  |  ID   |  ')'  |  $
───────┼───────┼───────┼───────┼───────┼───────┼──────
expr   │       │       │  1    │  1    │       │
expr'  │  2    │       │       │       │  3    │  3
term   │       │       │  4    │  4    │       │
term'  │  6    │  5    │       │       │  6    │  6
factor │       │       │  7    │  8    │       │
```

### Bottom-Up Parsing

#### Shift-Reduce Parsing
Maintains a **stack**. Two actions:
- **Shift**: push next token onto stack
- **Reduce**: pop RHS of a production, push LHS

```
Stack           Input           Action
─────           ─────           ──────
$               id + id $       shift
$ id            + id $          reduce (factor → id)
$ factor        + id $          reduce (term → factor)
$ term          + id $          shift
$ term +        id $            shift
$ term + id     $               reduce (factor → id)
$ term + factor $               reduce (term → factor)
$ term + term   $               reduce (expr → expr + term)
$ expr          $               accept
```

#### LR(0), SLR(1), LALR(1), LR(1)
| Variant | Power | States | Use |
|---|---|---|---|
| LR(0) | Weakest | Few | Rarely sufficient |
| SLR(1) | Moderate | Few | Simple grammars |
| **LALR(1)** | Strong | Moderate | **yacc/bison — most practical** |
| LR(1) | Strongest | Many | Full power, large tables |

#### Earley Parser
- Handles **all** CFGs (including ambiguous)
- O(n³) general, O(n²) unambiguous, O(n) for LR grammars
- Used in NLP and some compilers (GHC's old parser)

#### Packrat / PEG Parsing
- **Parsing Expression Grammars** — ordered choice, no ambiguity
- Memoization gives O(n) time
- Tools: **PEG.js**, **pest** (Rust), **parsec** (Haskell)

---

## 15. SSA Form (Static Single Assignment)

Each variable is **defined exactly once**. Transforming to SSA:

```
// Original:
x = 1
x = x + 2
y = x

// SSA:
x₁ = 1
x₂ = x₁ + 2
y₁ = x₂
```

### φ (Phi) Functions
At join points (after branches), use φ to select the correct version:

```
if cond:
    x₁ = 5       x₂ = 10
         \       /
          x₃ = φ(x₁, x₂)   ← select based on which branch was taken
```

### Why SSA?
- Makes **def-use chains** explicit
- Simplifies many optimizations (DCE, CSE, GVN, PRE)
- Enables **sparse** analysis (track individual definitions)

### SSA Construction: Dominance Frontiers
1. Build the CFG
2. Compute **dominators** (node A dominates B if all paths to B go through A)
3. Compute **dominance frontier** DF(A) = {nodes where A's dominance ends}
4. Insert φ-functions at dominance frontiers
5. Rename variables

---

## 16. Register Allocation

Assign **virtual registers** (IR) to **physical registers** (finite hardware registers). When more variables are live than registers exist, some must be **spilled** to the stack.

### Liveness Analysis
A variable is **live** at a point if it will be used before being redefined.

```
// Liveness equations (backward dataflow):
liveIn[B]  = use[B] ∪ (liveOut[B] − def[B])
liveOut[B] = ∪ { liveIn[S] for each successor S of B }
```

### Interference Graph
Two variables **interfere** if they are simultaneously live.
Build a graph: nodes = variables, edges = interference.

### Graph Coloring (Chaitin's Algorithm)
Coloring the interference graph with **k colors** (k = # physical registers):
1. **Simplify**: Remove nodes with degree < k (can always color these)
2. **Spill**: If stuck, choose a node to spill (mark it)
3. **Select**: Pop nodes, assign colors avoiding neighbors
4. **Spill code**: Insert load/store for spilled variables

### Linear Scan (LLVM, JIT)
Faster alternative to graph coloring:
- Sort live intervals by start point
- Greedily assign registers
- Expire intervals that end before the current start
- Spill the interval with the furthest end point when no registers available

---

## 17. Linking & Loading

### Object Files
Compiled source files produce **object files** (.o / .obj) containing:
- **Text section**: machine code
- **Data section**: initialized globals
- **Symbol table**: exported/imported symbols
- **Relocation table**: patch addresses after linking

### Static Linking
The **linker** combines object files into one executable:
1. **Symbol resolution**: match each reference to a definition
2. **Relocation**: patch addresses to their final values

```
main.o  ─┐
util.o  ─┼──► linker ──► a.out (executable)
libfoo.a ─┘
```

### Dynamic Linking
Shared libraries (`.so` / `.dll` / `.dylib`) loaded at runtime:
- Smaller executables, shared across processes
- **PLT** (Procedure Linkage Table) + **GOT** (Global Offset Table) enable position-independent code

### ELF Format (Linux)
```
ELF Header
Program Headers    ← for loader (segments)
.text              ← executable code
.rodata            ← read-only data (string literals)
.data              ← initialized globals
.bss               ← uninitialized globals (zero-filled)
.symtab            ← symbol table
.strtab            ← string table
.rel.text          ← relocation entries
Section Headers    ← for linker
```

### Position-Independent Code (PIC)
Code that runs correctly regardless of load address. Required for shared libraries.
Uses PC-relative addressing; GOT stores absolute addresses filled in by dynamic linker.

---

## 18. Advanced Topics

### Compiler Correctness & Verification
- **CompCert**: formally verified C compiler (Coq proof)
- **Alive2**: formally verifies LLVM optimization passes
- **Translation validation**: verify each compilation result

### Supercompilation / Partial Evaluation
- **Partial evaluation**: specialize a program for known inputs → residual program
- **Supercompilation**: drive execution symbolically, fold repeated states

### Profile-Guided Optimization (PGO)
1. Compile with instrumentation
2. Run with representative workload, collect profile
3. Recompile using profile data → branch hints, inlining decisions, layout

### Link-Time Optimization (LTO)
Optimization across translation units at link time. Enables whole-program inlining, DCE, and IPO.

### Polyhedral Model
Represent loop nests as polyhedra; use affine transformations for optimal loop nesting, tiling, and parallelization. Tools: **isl**, **Pluto**, **MLIR's affine dialect**.

### MLIR (Multi-Level IR)
Framework for building compiler IRs with reusable infrastructure:
- **Dialects**: domain-specific IR layers (affine, LLVM, linalg, GPU)
- **Passes**: transformations between dialects
- Powers TensorFlow XLA, Torch-MLIR, Flang (Fortran)

### WebAssembly (Wasm)
- Stack-based virtual ISA; runs in browsers & servers
- Compiler target for C, Rust, Go, and others
- **WASI** extends to system interfaces

### Auto-Vectorization
Compiler automatically generates SIMD instructions (SSE, AVX, NEON):
```c
// Scalar loop:
for (int i = 0; i < n; i++) c[i] = a[i] + b[i];

// Vectorized (AVX2, 8 floats at once):
for (int i = 0; i < n; i+=8) _mm256_add_ps(a+i, b+i) → c+i
```

### Compiler-Based Security
| Technique | Description |
|---|---|
| **Stack canaries** | Detect buffer overflows |
| **ASLR** | Randomize address space layout |
| **CFI** (Control Flow Integrity) | Restrict valid branch targets |
| **ShadowStack** | Protect return addresses |
| **FORTIFY_SOURCE** | Safer libc wrappers |

---

## 19. Real-World Compilers

### GCC (GNU Compiler Collection)
- Languages: C, C++, Fortran, Ada, Go
- IR: GIMPLE → RTL
- Back ends: x86, ARM, RISC-V, MIPS, PowerPC, and many more
- Passes: ~200 optimization passes

### LLVM / Clang
- Modular, reusable compiler infrastructure
- IR: **LLVM IR** (SSA, typed, infinite registers)
- Used by: Clang (C/C++), Rust (rustc), Swift, Julia, Kotlin/Native
- **MLIR** is LLVM's next-generation IR framework

### GHC (Glasgow Haskell Compiler)
- Multiple IRs: Core (System F) → STG → Cmm → native/LLVM
- Features: lazy evaluation, type inference, deforestation

### V8 (JavaScript — Google)
- Ignition: bytecode interpreter
- TurboFan: optimizing JIT compiler
- Maglev: mid-tier JIT (new)
- Sparkplug: fast baseline JIT

### HotSpot JVM
- **C1**: fast, lightly optimizing JIT
- **C2**: heavily optimizing JIT (sea-of-nodes IR)
- Tiered compilation: interpreted → C1 → C2

### rustc
- Front end: Rust AST → HIR (High-level IR)
- Type checking / borrow checking on HIR/MIR
- **MIR** (Mid-level IR): control-flow graph, enables borrow checker
- Back end: MIR → LLVM IR → machine code

---

## 20. Quick Reference Glossary

| Term | Definition |
|---|---|
| **AST** | Abstract Syntax Tree — tree representing parsed program structure |
| **Attribute grammar** | Grammar extended with semantic rules (synthesized / inherited attributes) |
| **Basic block** | Maximal straight-line code sequence (single entry, single exit) |
| **CFG** | Control Flow Graph — graph of basic blocks and their control edges |
| **Closure** | Function + its captured environment |
| **CPS** | Continuation-Passing Style — every function takes a "what to do next" argument |
| **DAG** | Directed Acyclic Graph — used to represent expressions without duplication |
| **Dataflow analysis** | Framework for computing properties of programs over CFG edges |
| **Dominator** | Node A dominates B if every path to B goes through A |
| **DFA** | Deterministic Finite Automaton — used in lexing |
| **Epsilon (ε)** | Empty string — a production that derives nothing |
| **Fixed point** | Iterative analysis terminates when values stop changing |
| **GVN** | Global Value Numbering — assigns value numbers to detect equivalent expressions |
| **Inline cache** | Cached type info to speed up dynamic dispatch (V8, JVM) |
| **Lattice** | Partially ordered set with least upper bound; used in dataflow analysis |
| **LICM** | Loop-Invariant Code Motion |
| **NFA** | Nondeterministic Finite Automaton — intermediate step in regex compilation |
| **Peephole** | Local optimization on a small instruction window |
| **PRE** | Partial Redundancy Elimination — generalization of CSE |
| **φ-function** | SSA merge point: selects value based on which predecessor was taken |
| **Reduction** | Replace a pattern with a cheaper equivalent |
| **Spilling** | Saving a register to memory when running out of physical registers |
| **SSA** | Static Single Assignment — each variable defined exactly once |
| **Strength reduction** | Replace expensive ops with equivalent cheaper ones |
| **Symbolic execution** | Execute program with symbolic instead of concrete values |
| **Tokens** | Lexer output: categorized chunks of source text |
| **Unification** | Algorithm to find substitution making two types equal |
| **Use-def chain** | Links each use of a variable to its definition(s) |
| **Virtual register** | Compiler's unlimited registers before hardware register allocation |

---

## Further Learning Path

```
Beginner       →  "Crafting Interpreters" (R. Nystrom) — free online
Intermediate   →  "Compilers: Principles, Techniques, and Tools" (Dragon Book)
Advanced       →  "Engineering a Compiler" (Cooper & Torczon)
LLVM           →  llvm.org/docs + "Getting Started with LLVM Core Libraries"
Research       →  PLDI, POPL, CGO conference papers
Practice       →  Build: a Lisp interpreter → a bytecode VM → an LLVM pass
```

---

*Last updated: 2026 | Covers: lexing, parsing, semantic analysis, IR, optimization, code generation, linking, and advanced compiler topics.*
