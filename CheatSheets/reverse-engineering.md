# Reverse Engineering Cheat Sheet
### From Zero to Expert

---

## What is Reverse Engineering?

**Reverse engineering** is the process of analyzing a compiled binary, firmware, protocol, or system to understand its structure, behavior, and logic — without access to the original source code. It is the foundation of malware analysis, vulnerability research, CTF competitions, software interoperability, and exploit development.

```
Source Code  ──►  Compiler  ──►  Binary
                                   │
                              Reverse Engineer
                                   │
                                   ▼
                           Recovered Logic / Behavior
```

> **Legal context:** Reverse engineering is legal for interoperability (EU Software Directive, DMCA §1201(f)), security research, and personal education. Always obtain authorization before analyzing proprietary software in a commercial context.

---

## The Fundamental Mindset

Reverse engineering is not about recovering perfect source code. It is about answering specific questions:

- What does this function do?
- What data structure is this?
- How does this protocol work?
- Where is the vulnerability?
- How does this malware persist?

Start with **what you need to know**, not with reading every instruction. Work top-down: behavior → functions → critical paths → individual instructions.

---

## Binary Formats

Before analyzing a binary, understand its container format. The format determines how code and data are laid out in memory.

### ELF (Executable and Linkable Format) — Linux

```
ELF Header
  ├── Magic: 7f 45 4c 46 (0x7f 'E' 'L' 'F')
  ├── Class: 32/64-bit
  ├── Endianness
  ├── Entry point address
  ├── Program header table offset
  └── Section header table offset

Program Headers (for loader):
  LOAD     → segments mapped into memory
  DYNAMIC  → dynamic linking info
  INTERP   → path to dynamic linker (/lib/ld-linux.so)
  GNU_STACK → stack permissions (NX bit)

Key Sections:
  .text    → executable code
  .data    → initialized global variables
  .bss     → uninitialized globals (zero-filled)
  .rodata  → read-only data (string literals, constants)
  .plt     → Procedure Linkage Table (lazy binding stubs)
  .got     → Global Offset Table (resolved addresses)
  .dynsym  → dynamic symbol table
  .symtab  → full symbol table (stripped in release builds)
  .debug_* → DWARF debug info
```

```bash
# Inspect ELF:
readelf -h binary          # ELF header
readelf -S binary          # section headers
readelf -l binary          # program headers
readelf -d binary          # dynamic section
readelf -s binary          # symbol table
objdump -d binary          # disassemble .text
file binary                # quick format identification
```

### PE (Portable Executable) — Windows

```
DOS Header (MZ magic: 4d 5a)
  └── e_lfanew → offset to PE Header

PE Header (PE\0\0 magic: 50 45 00 00)
  ├── COFF Header (machine type, timestamp, number of sections)
  └── Optional Header
        ├── AddressOfEntryPoint
        ├── ImageBase (default load address)
        ├── SectionAlignment / FileAlignment
        └── Data Directories:
              Import Table (.idata)   → DLLs and functions used
              Export Table            → functions exposed by DLLs
              Resource Table          → icons, strings, version info
              TLS Table               → Thread Local Storage callbacks
              Debug Directory         → PDB path, debug info
              .NET Header             → managed code indicator

Key Sections:
  .text    → code
  .data    → initialized data
  .rdata   → read-only data
  .rsrc    → resources
  .reloc   → base relocations
  .tls     → thread local storage
```

```bash
# Inspect PE (Windows):
dumpbin /headers binary.exe
dumpbin /imports binary.exe
dumpbin /exports binary.dll
# On Linux:
pefile (Python library)
pe-bear (GUI tool)
```

### Mach-O — macOS / iOS

```
Magic: CE FA ED FE (32-bit LE) / CF FA ED FE (64-bit LE)

Structure:
  mach_header
  Load Commands:
    LC_SEGMENT_64    → memory segments (__TEXT, __DATA, __LINKEDIT)
    LC_MAIN          → entry point
    LC_LOAD_DYLIB    → dynamic libraries imported
    LC_CODE_SIGNATURE → code signing
    LC_ENCRYPTION_INFO → App Store encryption (iOS)

Key Segments:
  __TEXT.__text      → executable code
  __TEXT.__cstring   → C string literals
  __DATA.__data      → initialized data
  __DATA.__bss       → uninitialized data
  __DATA.__got       → non-lazy symbol pointers
  __DATA.__la_symbol_ptr → lazy symbol pointers
```

### Other Formats

| Format | Magic Bytes | Usage |
|---|---|---|
| Java .class | CA FE BA BE | JVM bytecode |
| .NET PE | MZ + CLR header | C#/VB.NET managed code |
| WebAssembly | 00 61 73 6D | Browser/server WASM |
| DEX | 64 65 78 0A | Android bytecode |
| LLVM Bitcode | BC C0 DE | LLVM IR |
| Universal Binary | CA FE BA BE (same as Java!) | Multi-arch Mach-O |

---

## x86 / x86-64 Architecture Essentials

A solid grasp of the underlying architecture is non-negotiable for RE.

### Registers (x86-64)

```
General Purpose:
  rax  → accumulator; return value of functions; syscall number
  rbx  → callee-saved (preserved across calls)
  rcx  → 4th arg (Linux); counter in loops; 1st arg (Windows)
  rdx  → 3rd arg; I/O port ops; high 64 bits of mul/div result
  rsi  → 2nd arg; source index in string ops
  rdi  → 1st arg; destination index in string ops
  rbp  → base pointer (frame pointer; optional in -fomit-frame-pointer)
  rsp  → stack pointer (always points to top of stack)
  r8–r15 → 5th–8th args + general use

Instruction Pointer:
  rip  → address of next instruction to execute

Flags Register (RFLAGS):
  ZF (Zero Flag)     → set if result == 0
  SF (Sign Flag)     → set if result is negative
  CF (Carry Flag)    → set on unsigned overflow
  OF (Overflow Flag) → set on signed overflow
  PF (Parity Flag)   → set if low byte has even number of set bits
  IF (Interrupt Flag)→ enables/disables hardware interrupts

Segment Registers (mostly legacy):
  cs, ds, es, fs, gs, ss
  fs and gs used for TLS (Thread Local Storage) on modern systems

32/16/8-bit sub-registers:
  rax → eax (32-bit) → ax (16-bit) → ah/al (8-bit high/low)
  r8  → r8d (32)     → r8w (16)    → r8b (8)
```

### Calling Conventions

**System V AMD64 ABI (Linux / macOS):**
```
Integer/pointer args:  rdi, rsi, rdx, rcx, r8, r9  (then stack)
Floating point args:   xmm0–xmm7
Return value:          rax (rax:rdx for 128-bit)
Callee-saved:          rbx, rbp, r12–r15
Caller-saved:          rax, rcx, rdx, rsi, rdi, r8–r11
Stack alignment:       16-byte aligned at CALL instruction
```

**Microsoft x64 ABI (Windows):**
```
Integer/pointer args:  rcx, rdx, r8, r9  (then stack)
Floating point args:   xmm0–xmm3
Return value:          rax
Callee-saved:          rbx, rbp, rdi, rsi, r12–r15, xmm6–xmm15
Shadow space:          32 bytes allocated by caller above return addr
```

**32-bit x86 calling conventions:**
```
cdecl:    args pushed right→left; caller cleans stack; return in eax
stdcall:  args pushed right→left; callee cleans stack (WinAPI default)
fastcall: first 2 args in ecx, edx; rest on stack
thiscall: 'this' pointer in ecx (MSVC C++ methods)
```

### The Stack Frame

```
High address
┌──────────────────┐
│  ...caller...    │
├──────────────────┤
│  arg 7+          │  ← pushed by caller (if > 6 args)
├──────────────────┤
│  return address  │  ← pushed by CALL instruction
├──────────────────┤  ← rsp after CALL
│  saved rbp       │  ← PUSH rbp  (function prologue)
├──────────────────┤  ← rbp (new frame pointer)
│  local var 1     │  [rbp - 8]
│  local var 2     │  [rbp - 16]
│  local var 3     │  [rbp - 24]
│  ...             │
├──────────────────┤  ← rsp (current stack top)
Low address
```

**Function prologue (setup):**
```asm
push rbp           ; save caller's frame pointer
mov  rbp, rsp      ; set up new frame pointer
sub  rsp, 0x30     ; allocate space for locals
```

**Function epilogue (teardown):**
```asm
mov  rsp, rbp      ; restore stack pointer (or: leave)
pop  rbp           ; restore caller's frame pointer
ret                ; pop return address and jump to it
```

### Critical x86-64 Instructions for RE

```asm
; Data movement:
mov  dst, src      ; copy src to dst
lea  dst, [addr]   ; load effective address (often used for arithmetic)
xchg dst, src      ; exchange
push src           ; rsp -= 8; [rsp] = src
pop  dst           ; dst = [rsp]; rsp += 8
movzx dst, src     ; move with zero-extension (unsigned widen)
movsx dst, src     ; move with sign-extension (signed widen)

; Arithmetic:
add  dst, src      ; dst = dst + src
sub  dst, src      ; dst = dst - src
imul dst, src      ; signed multiply
idiv src           ; rax = rdx:rax / src; rdx = remainder
inc  dst           ; dst += 1
dec  dst           ; dst -= 1
neg  dst           ; dst = -dst

; Bitwise:
and  dst, src      ; bitwise AND
or   dst, src      ; bitwise OR
xor  dst, src      ; bitwise XOR  (xor eax,eax → fast zero)
not  dst           ; bitwise NOT
shl  dst, n        ; shift left  (multiply by 2^n)
shr  dst, n        ; shift right logical (unsigned divide by 2^n)
sar  dst, n        ; shift right arithmetic (signed divide by 2^n)
ror  dst, n        ; rotate right
rol  dst, n        ; rotate left

; Comparison (set flags, discard result):
cmp  a, b          ; computes a - b; sets flags
test a, b          ; computes a & b; sets flags
                   ; test eax,eax → check if eax is zero (ZF)
                   ; test eax,1   → check if lowest bit set (CF/PF)

; Jumps:
jmp  addr          ; unconditional jump
je / jz            ; jump if ZF=1 (equal / zero)
jne/ jnz           ; jump if ZF=0 (not equal / not zero)
jl / jnge          ; jump if SF≠OF (signed less than)
jle/ jng           ; jump if ZF=1 or SF≠OF
jg / jnle          ; jump if ZF=0 and SF=OF (signed greater than)
jge/ jnl           ; jump if SF=OF
jb / jnae          ; jump if CF=1 (unsigned below)
ja / jnbe          ; jump if CF=0 and ZF=0 (unsigned above)
js                 ; jump if SF=1 (negative)
jo                 ; jump if OF=1 (overflow)

; Function calls:
call addr          ; push rip+1; jmp addr
ret                ; pop rip (return to caller)
retn N             ; ret + add rsp,N  (stdcall cleanup)

; String operations (use with rep prefix):
movs[b/w/d/q]      ; copy [rsi] to [rdi]; advance pointers
cmps[b/w/d/q]      ; compare [rsi] with [rdi]
scas[b/w/d/q]      ; compare rax with [rdi]  (strlen pattern)
stos[b/w/d/q]      ; store rax to [rdi]  (memset pattern)

; System calls (Linux x86-64):
syscall            ; invoke kernel; number in rax; args rdi,rsi,rdx,r10,r8,r9
```

---

## ARM Architecture Essentials

ARM dominates mobile, embedded, and increasingly desktop (Apple Silicon). RE without ARM knowledge is incomplete.

### ARM64 (AArch64) Registers

```
x0–x7    → function arguments & return values
x8       → indirect result location / syscall number
x9–x15   → caller-saved temporaries
x16–x17  → intra-procedure call scratch (IP0/IP1)
x18      → platform register (often TLS on iOS)
x19–x28  → callee-saved
x29 (fp) → frame pointer
x30 (lr) → link register (return address)
sp       → stack pointer
pc       → program counter (not directly addressable)
xzr/wzr  → zero register (reads as 0; writes discarded)
```

```asm
; ARM64 key instructions:
ldr  x0, [x1]        ; load 8 bytes from [x1] into x0
ldr  w0, [x1]        ; load 4 bytes (32-bit) from [x1]
str  x0, [x1]        ; store x0 to [x1]
ldp  x29, x30, [sp]  ; load pair (common in prologues)
stp  x29, x30, [sp, #-16]! ; store pair, pre-decrement sp

; Arithmetic:
add  x0, x1, x2      ; x0 = x1 + x2
sub  x0, x1, #4      ; x0 = x1 - 4
mul  x0, x1, x2      ; x0 = x1 * x2

; Branches:
b    label            ; unconditional branch
bl   func             ; branch with link (call: saves pc to lr)
ret                   ; return (branch to lr)
cbz  x0, label        ; branch if x0 == 0
cbnz x0, label        ; branch if x0 != 0
b.eq / b.ne / b.lt / b.gt / b.le / b.ge   ; conditional branches

; Comparison:
cmp  x0, x1           ; sets flags (like x86 cmp)
tst  x0, #0xFF        ; bitwise AND, sets flags only

; Bit manipulation:
lsl  x0, x1, #3       ; logical shift left
lsr  x0, x1, #3       ; logical shift right
asr  x0, x1, #3       ; arithmetic shift right
and  x0, x1, x2
orr  x0, x1, x2
eor  x0, x1, x2       ; XOR
```

### ARM32 Key Differences

```asm
; Thumb-2 mode (compact 16/32-bit instructions, common in embedded):
; BX LR  → return from function
; BLX    → branch with link and exchange (ARM↔Thumb mode switch)
; Conditional execution suffix: MOVEQ, ADDNE, BEQ, etc.
; LDM/STM (load/store multiple) → push/pop multiple registers at once
PUSH {r4-r7, lr}    ; save registers + link register
POP  {r4-r7, pc}    ; restore + return (load lr into pc)
```

---

## Static Analysis

Analyzing a binary **without executing it**. Safe, repeatable, and works on malware.

### Initial Triage

```bash
# Identify the binary type, architecture, linking:
file binary
# Output: ELF 64-bit LSB executable, x86-64, dynamically linked, not stripped

# Check strings (find hardcoded URLs, registry keys, error messages, commands):
strings binary
strings -n 6 binary      # minimum 6-char strings
strings -e l binary      # little-endian 16-bit (Unicode Windows strings)

# Detect packer/protector/compiler:
detect-it-easy (DIE) binary    # GUI + command-line
exeinfo binary
PE-bear binary.exe
# Look for: UPX, Themida, VMProtect, MPRESS, NSIS, packed entropy

# Check entropy (packed/encrypted sections have entropy near 8.0):
binwalk -E binary        # entropy graph
# Normal code: ~5.5–6.5; compressed/encrypted: ~7.5–8.0

# Import analysis (what APIs/libraries does it use?):
readelf -d binary        # dynamic section dependencies
objdump -p binary        # imports (Linux)
dumpbin /imports binary  # Windows

# Hash for threat intel lookups:
md5sum binary; sha256sum binary
# Search VirusTotal, MalwareBazaar, Hybrid Analysis
```

### Disassemblers

Disassembly converts raw bytes back into assembly instructions.

```
Linear sweep:   Process bytes sequentially (simple, misses data/code boundaries)
Recursive descent: Follow control flow from entry point (smarter, handles obfuscation less well)
```

**Key Disassemblers:**

| Tool | Type | Best For |
|---|---|---|
| **IDA Pro** | Commercial | Industry standard, best analysis |
| **Ghidra** | Free (NSA) | Full-featured, excellent decompiler |
| **Binary Ninja** | Commercial | Modern API, scripting-first |
| **Radare2 / Cutter** | Free | CLI power, scriptable |
| **Hopper** | Commercial | macOS/iOS focus |
| **objdump** | Free | Quick CLI disassembly |
| **ndisasm** | Free | Raw binary disassembly |

### Decompilers

Lift assembly back to pseudo-C/C++. The output is **not** original source — it is a best-effort reconstruction.

```
IDA Pro + Hex-Rays Decompiler   → gold standard (F5 in IDA)
Ghidra built-in decompiler      → free, very capable
Binary Ninja HLIL               → high-level IL view
RetDec (Avast)                  → open source, online
rec studio                      → C decompiler, good for old binaries
Snowman                         → C++ decompiler
dotPeek / dnSpy / ILSpy         → .NET / C# decompiler (recovers near-original)
jadx / apktool                  → Android APK / DEX decompiler
cfr / procyon / fernflower       → Java .class decompiler
```

### IDA Pro Essentials

```
Navigation:
  Space          → toggle graph / text view
  G              → go to address
  N              → rename symbol (function, variable, label)
  Y              → set type (function signature, variable type)
  ;              → add repeatable comment
  /              → add non-repeatable comment
  H              → convert to hex display
  A              → convert to ASCII string
  D              → convert to data (byte/word/dword)
  C              → convert to code
  P              → create function
  U              → undefine
  X              → cross-references (Xrefs)
  Ctrl+X         → Xrefs to current location
  Alt+T          → search text
  Alt+B          → search bytes
  Ctrl+L         → jump to label
  F5             → decompile to pseudocode (Hex-Rays)
  Esc            → navigate back

Key Windows:
  Imports window     → see all imported API calls (critical for malware)
  Exports window     → functions exposed by a DLL
  Strings window     → all decoded strings
  Functions window   → list of all identified functions
  Xrefs             → who calls this? where is this data used?
  Structures         → define custom data structures
  Enums              → define enum types
```

### Recognizing Common Patterns in Assembly

```asm
; NULL check:
test  rax, rax
jz    null_handler          ; if rax == NULL, jump

; Integer comparison if/else:
cmp   eax, 5
jge   greater_or_equal
; ... else body ...
jmp   end
; ... if body ...

; For loop (for i=0; i<n; i++):
xor   ecx, ecx             ; i = 0
.loop:
  cmp   ecx, [n]
  jge   .done
  ; ... body ...
  inc   ecx
  jmp   .loop
.done:

; While loop:
.while:
  cmp   [condition], 0
  je    .end_while
  ; ... body ...
  jmp   .while

; Switch statement (jump table):
cmp   eax, max_case
ja    default
lea   rcx, [jump_table]
movsxd rax, dword [rcx + rax*4]
add   rax, rcx
jmp   rax                  ; indirect jump to case handler

; strlen (scasb pattern):
mov   rdi, str_ptr
xor   eax, eax             ; search for 0x00
mov   rcx, -1
repne scasb                 ; scan until AL found, decrement RCX
not   rcx
dec   rcx                   ; RCX now holds length

; memcpy (rep movsb):
mov   rdi, dst
mov   rsi, src
mov   rcx, count
rep movsb

; Structure member access:
; struct->member at offset 0x18:
mov   rax, [rbx + 0x18]    ; rbx = struct pointer
```

---

## Dynamic Analysis

Analyzing a binary **while it executes**. Reveals runtime behavior, network activity, file system changes, and decrypted data that static analysis cannot see.

### Sandboxing (Automated Dynamic Analysis)

Run the sample in an isolated environment and observe behavior automatically.

```
Online sandboxes:
  Any.run         → interactive sandbox, live process tree
  Hybrid Analysis → free, FireEye tech
  Joe Sandbox     → very detailed reports
  Cuckoo Sandbox  → open-source, self-hosted
  VirusTotal      → multi-AV + behavioral analysis

What sandboxes capture:
  File system:  files created/modified/deleted
  Registry:     keys read/written/deleted (Windows)
  Network:      DNS queries, TCP/UDP connections, HTTP requests
  Processes:    spawned processes, injected threads
  API calls:    all Windows API / Linux syscalls traced
```

### Debuggers

A debugger lets you step through execution instruction by instruction, inspect registers and memory, set breakpoints, and modify runtime state.

**Linux Debuggers:**
```bash
gdb binary                    # GNU Debugger
gdb -p PID                    # attach to running process
r2 -d binary                  # Radare2 in debug mode
lldb binary                   # LLDB (also on macOS)
```

**GDB with PEDA / GEF / pwndbg** (essential extensions):
```bash
# Install GEF (recommended):
pip3 install gef

# Core GDB commands:
run [args]         → start execution
run < input.txt    → run with stdin
break main         → breakpoint at function
break *0x400560    → breakpoint at address
info breakpoints   → list breakpoints
delete 1           → delete breakpoint 1
continue (c)       → resume execution
step (s)           → step into (follow calls)
next (n)           → step over (don't follow calls)
finish             → run until function returns
stepi (si)         → step one instruction
nexti (ni)         → step over one instruction

# Inspect state:
info registers     → all registers
p $rax             → print register value
p/x $rax           → print in hex
x/10wx $rsp        → examine 10 words at rsp
x/20i $rip         → disassemble 20 instructions at rip
x/s 0x601020       → examine as string
x/b 0x601020       → examine as bytes
info proc mappings → memory layout
vmmap              → (GEF) colored memory map
heap chunks        → (GEF) heap layout

# Modify state:
set $rax = 0       → change register
set {int}0x601020 = 42  → write to memory
patch byte 0x400560 0x90  → NOP a byte

# Breakpoint conditions:
break *0x400560 if $rdi == 0
commands 1         → run these commands when bp 1 hits
  p/x $rax
  continue
end
```

**Windows Debuggers:**
```
x64dbg / x32dbg    → best free Windows debugger; scriptable; plugin-rich
WinDbg / WinDbg Preview → Microsoft kernel + user-mode debugger
OllyDbg            → classic 32-bit; many old tutorials reference it
Immunity Debugger  → OllyDbg fork, Python scripting (pwndbg ancestor)

x64dbg key shortcuts:
  F2    → toggle breakpoint at cursor
  F7    → step into
  F8    → step over
  F9    → run
  F4    → run to cursor
  Ctrl+G → go to address/symbol
  Ctrl+F → find in disassembly
```

### API Monitoring

```
# Windows API tracing:
API Monitor       → log every Win32 API call with args + return value
Frida             → dynamic instrumentation; hook any function in any process
Process Monitor   → file, registry, network, process activity
Procmon + Regshot → before/after snapshot comparison

# Linux syscall tracing:
strace ./binary               → trace all syscalls
strace -e trace=network ./binary  → network syscalls only
ltrace ./binary               → trace library calls
perf trace ./binary           → modern alternative to strace
```

### Network Analysis

```
Wireshark          → full packet capture and protocol dissection
tcpdump            → CLI packet capture
mitmproxy          → intercept and modify HTTP/HTTPS
FakeDNS / INetSim  → simulate internet services in isolated lab
Fiddler            → Windows HTTP/HTTPS proxy

# For HTTPS interception:
# Install mitmproxy certificate as trusted root CA in analysis VM
mitmproxy --mode transparent
```

### Frida — Dynamic Instrumentation

Frida lets you inject JavaScript into any running process on any platform to hook functions, read memory, and alter behavior — without modifying the binary.

```javascript
// Attach to process and hook a function:
Java.perform(() => {               // Android Java
  const MyClass = Java.use('com.example.MainActivity');
  MyClass.checkLicense.implementation = function(key) {
    console.log('[*] checkLicense called with: ' + key);
    return true;                   // always return true → bypass license check
  };
});

// Hook native function by address:
Interceptor.attach(ptr('0x7f1234560000').add(0x1234), {
  onEnter(args) {
    console.log('arg0:', args[0].readUtf8String());
    console.log('arg1:', args[1].toInt32());
  },
  onLeave(retval) {
    console.log('return:', retval.toInt32());
    retval.replace(ptr(1));        // override return value
  }
});

// Read/write memory:
Memory.readByteArray(ptr('0x401000'), 32);
Memory.writeByteArray(ptr('0x401000'), [0x90, 0x90, 0x90]);

// Enumerate modules:
Process.enumerateModules().forEach(m => console.log(m.name, m.base));

// Find function by export name:
const addr = Module.findExportByName('libc.so.6', 'strcmp');
```

```bash
# Frida CLI usage:
frida -U -f com.example.app -l hook.js   # spawn + inject on Android
frida -p 1234 -l hook.js                 # attach to PID
frida-trace -p 1234 -i "strcmp"          # auto-trace all strcmp calls
frida-ps -U                              # list processes on USB device
```

---

## Anti-Analysis Techniques and Bypasses

Malware and protected software actively resist analysis. Knowing these techniques is as important as knowing how to analyze.

### Anti-Debugging Techniques

```c
// Windows — IsDebuggerPresent:
if (IsDebuggerPresent()) { exit(1); }
// Bypass: patch NOP the check, or set PEB.BeingDebugged = 0

// Windows — CheckRemoteDebuggerPresent:
BOOL dbg; CheckRemoteDebuggerPresent(GetCurrentProcess(), &dbg);

// PEB direct check:
mov eax, fs:[30h]          ; PEB pointer (32-bit TEB)
mov eax, [eax+2]           ; PEB.BeingDebugged byte
test eax, eax
jnz  detected

// NtQueryInformationProcess (ProcessDebugPort):
// Returns non-zero if debugged
// Bypass: hook NtQueryInformationProcess, always return 0

// Timing checks:
RDTSC before and after code block
if (delta > threshold) → debugger slowing execution → exit
// Bypass: patch the comparison, or use a fast system

// INT 3 / 0xCC scanning:
// Malware checks its own code for breakpoint bytes
// Bypass: use hardware breakpoints (don't modify memory)

// SEH-based detection (32-bit):
// Set up exception handler; if exception handled normally → no debugger
// If debugger catches first → detected

// GetTickCount / timeGetTime delta
// OutputDebugString trick (causes NtSetLastError behavior change)
// FindWindow ("x64dbg", "OllyDbg", "Immunity Debugger")
// Process name / module name checks (CreateToolhelp32Snapshot)
// Parent process check (debugger is parent process)
```

### Anti-Disassembly Techniques

```asm
; Junk byte insertion:
jmp  real_code           ; disassembler assumes next byte is instruction
db   0xe8                ; this byte confuses linear sweep disassemblers
real_code:
  ; actual code here

; Opaque predicates:
; A condition that always evaluates the same way, but is not obvious statically
xor eax, eax
jz  real_target          ; always jumps (ZF always 1 after xor eax,eax)
db  0xFF, 0x15           ; garbage bytes the disassembler tries to decode

; Return address modification:
call next_instruction
next_instruction:
  pop rax                ; rax = address of next instruction
  add rax, 5             ; modify return address
  push rax
  ret                    ; "return" to modified address (indirect jump)

; Self-modifying code:
; Code writes new bytes over itself before executing them
; Cannot be analyzed statically — must run to see final code
```

### Packers and Protectors

```
Packers: compress/encrypt the original binary; stub decrypts at runtime
  UPX      → most common, easily unpacked with upx -d
  MPRESS   → .NET focused
  PECompact, ASPack, Petite

Protectors: virtualization, obfuscation, anti-tamper
  Themida / WinLicense → code virtualization (custom VM bytecode)
  VMProtect            → selective virtualization of critical functions
  Obsidium, Armadillo  → older but still seen
  .NET Reactor          → .NET obfuscation + native wrapper
  
Identifying packers:
  High entropy sections (>7.5)
  Few imports (just LoadLibrary, GetProcAddress, VirtualAlloc)
  Single section or non-standard section names
  Entry point in last section (not .text)
  
Generic unpacking approach:
  1. Load in debugger
  2. Set breakpoint on VirtualAlloc / VirtualProtect (regions being written then executed)
  3. Run until original entry point (OEP) is jumped to
  4. Dump process memory (Scylla, PE-sieve, procdump)
  5. Fix import table (Scylla IAT fix)
  6. Analyze the dumped binary
```

### Code Virtualization (VM Obfuscation)

The most advanced protection. Original instructions replaced by custom bytecode for a proprietary virtual machine embedded in the binary.

```
Architecture:
  Handler table    → array of function pointers, one per opcode
  Dispatcher       → fetch next bytecode, index into handler table, jump
  vCPU registers   → virtual registers mapped to real registers or memory

Analysis approach:
  1. Find the dispatcher loop (small loop with indirect jump)
  2. Identify the handler table
  3. Trace execution and log each handler invoked
  4. Reverse each handler to understand its semantics
  5. Reconstruct the original algorithm from the handler sequence

Tools: VMAttack (IDA plugin), TITAN (academic), manual analysis
```

---

## Vulnerability Research

Applying RE skills to find exploitable security flaws.

### Buffer Overflow

**Stack-based overflow:**
```c
// Vulnerable code:
void vuln(char *input) {
    char buf[64];
    strcpy(buf, input);     // no bounds check
}

// Stack layout before overflow:
// [buf: 64 bytes][saved rbp: 8 bytes][return address: 8 bytes]

// Overflow steps:
// 1. Fill buf (64 bytes)
// 2. Overwrite saved rbp (8 bytes)
// 3. Overwrite return address → redirect execution

// Finding offset:
cyclic 200                   # generate De Bruijn pattern (pwntools)
run < pattern.txt
# Check value in rip after crash
cyclic -l 0x6161616c         # find offset where crash occurred

// Basic exploit structure (pwntools):
from pwn import *
p = process('./vuln')
offset = 72                  # offset to return address
ret    = 0x401234            # ROP gadget or shellcode address
payload = b'A' * offset + p64(ret)
p.sendline(payload)
p.interactive()
```

**Heap overflow:** Overflows in heap-allocated buffers can corrupt heap metadata or adjacent objects, enabling arbitrary write primitives.

### Format String Vulnerabilities

```c
// Vulnerable:
printf(user_input);          // user controls format string

// Read arbitrary memory (%n$x reads nth stack argument):
printf("%x %x %x %x")       // leak stack values
printf("%7$x")               // read 7th argument directly
printf("%7$s")               // interpret 7th arg as string pointer → dereference

// Find offset to your buffer:
python3 -c "print('AAAA' + '.%x'*20)" | ./vuln
# Count until 41414141 appears → that's your offset N

// Write to arbitrary address (%n writes byte count to pointed address):
printf("%100c%7$n")          // write value 100 to address at 7th arg

// Full arbitrary write (4-byte write with %n):
payload = p32(target_addr) + b"%N$n"   # N = format arg position
```

### Use-After-Free (UAF)

```c
// Pattern:
char *ptr = malloc(64);
free(ptr);              // freed — but ptr still points to the chunk
// ... time passes, chunk reallocated for different purpose ...
ptr->vtable->method();  // use after free → if vtable overwritten → code exec

// Analysis approach in RE:
// 1. Find all free() calls
// 2. Trace all uses of the pointer after free
// 3. Determine if allocations between free and use can be controlled
```

### Integer Overflows

```c
// Signed overflow:
int size = user_controlled;
if (size > 0 && size < 1000)      // passes check
    char *buf = malloc(size);
    memcpy(buf, data, size * 4);   // size=0x40000000 → size*4 overflows to 0 → 0-byte malloc → heap overflow

// Unsigned wraparound:
unsigned short len = user_len;   // truncated to 16-bit
if (len < 256) { ... }           // passes if len = 0x10100 → stored as 0x0100 = 256? No, truncated to 0x100 = 256
```

### Return-Oriented Programming (ROP)

Modern exploit technique that bypasses NX/DEP by chaining existing code snippets ("gadgets") that each end in `RET`.

```python
# Finding gadgets:
ROPgadget --binary binary --rop        # find all ROP gadgets
ropper -f binary                        # alternative gadget finder
objdump -d binary | grep -E 'ret|pop'  # manual search

# Common gadgets needed:
# pop rdi; ret           → set first argument
# pop rsi; ret           → set second argument
# pop rdx; ret           → set third argument
# pop rax; ret           → set rax (syscall number)
# syscall; ret           → trigger syscall
# ret                    → stack alignment gadget

# Building a ROP chain (execve("/bin/sh", NULL, NULL)):
from pwn import *
elf = ELF('./binary')
rop = ROP(elf)

rop.raw(rop.find_gadget(['pop rdi', 'ret'])[0])
rop.raw(next(elf.search(b'/bin/sh')))   # address of "/bin/sh" string
rop.raw(rop.find_gadget(['pop rsi', 'ret'])[0])
rop.raw(0)
rop.raw(rop.find_gadget(['pop rdx', 'ret'])[0])
rop.raw(0)
rop.raw(elf.plt['execve'])              # or syscall gadget

payload = b'A' * offset + rop.chain()
```

### ASLR, NX, PIE, Stack Canaries — Bypasses

```
Protection      → Bypass Technique
────────────────────────────────────────────────────────────
Stack canary    → leak canary via format string; brute-force in fork()
NX/DEP          → ROP chains (code already in memory); JIT spraying
ASLR            → leak address via info disclosure; brute-force (32-bit); relative offsets with PIE
PIE             → leak base address via format string or use-after-free info leak
RELRO (full)    → can't overwrite GOT; target other writable regions
```

---

## Malware Analysis

A specialized RE discipline focused on understanding malicious software.

### Analysis Environment Setup

```
NEVER analyze malware on your host machine.
Use an isolated VM with:
  - No network access (or FakeDNS/INetSim for simulated internet)
  - Shared folders disabled or write-protected
  - Snapshots before every analysis session
  - Dedicated analysis tools pre-installed

Recommended lab setup:
  FlareVM (Windows) → comprehensive malware analysis distro
  REMnux (Linux)    → Linux-based malware analysis distro
  SANS SIFT         → forensics-focused
```

### Malware Classification

| Type | Behavior | Key Indicators |
|---|---|---|
| **Trojan** | Disguised as legitimate software | Drops/executes payload on run |
| **Ransomware** | Encrypts files, demands payment | CryptAPI / OpenSSL calls, file enumeration, C2 contact |
| **RAT** | Remote access / control | Reverse shell, keylogger, screenshot, C2 comms |
| **Rootkit** | Hides its presence | Hook SSDT/IDT, DKOM, filter drivers |
| **Bootkit** | Persists in MBR/UEFI | MBR write, UEFI variable modification |
| **Worm** | Self-propagating | Network scanning, exploitation, lateral movement |
| **Info stealer** | Harvests credentials/data | Browser DB access, keylogging, clipboard hook |
| **Dropper/Loader** | Delivers payload | Downloads/unpacks additional stage |
| **Backdoor** | Persistent access | Scheduled tasks, registry run keys, service install |

### Persistence Mechanisms (Windows)

```
Registry autorun keys:
  HKCU\Software\Microsoft\Windows\CurrentVersion\Run
  HKLM\Software\Microsoft\Windows\CurrentVersion\Run
  HKLM\SYSTEM\CurrentControlSet\Services\  → services
  HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon (Userinit, Shell)

Scheduled tasks:
  schtasks /create ...
  C:\Windows\System32\Tasks\
  C:\Windows\SysWOW64\Tasks\

Services:
  sc create; OpenSCManager / CreateService API calls

Startup folders:
  %APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup
  C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup

DLL hijacking:
  Place malicious DLL in app directory (loaded before system DLL)
  
COM hijacking:
  HKCU\Software\Classes\CLSID\ (user writable)
  
WMI event subscriptions:
  wmic /namespace:\\root\subscription ...

AppInit DLLs (legacy):
  HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows\AppInit_DLLs
```

### Process Injection Techniques

```c
// Classic DLL injection:
VirtualAllocEx(target)                // allocate memory in target
WriteProcessMemory(target, dll_path)  // write DLL path
CreateRemoteThread(target, LoadLibraryA, dll_path)  // load DLL

// Shellcode injection:
VirtualAllocEx(target, PAGE_EXECUTE_READWRITE)
WriteProcessMemory(target, shellcode)
CreateRemoteThread(target, shellcode_addr)

// Process hollowing:
CreateProcess(legit.exe, SUSPENDED)  // create suspended
NtUnmapViewOfSection(legit.exe)      // unmap original code
VirtualAllocEx + WriteProcessMemory  // write malicious code
SetThreadContext                     // redirect entry point
ResumeThread                         // resume execution

// APC injection:
OpenThread(target_thread)
QueueUserAPC(shellcode, target_thread)  // executes when thread enters alertable wait

// Reflective DLL injection:
// DLL contains its own loader; injected as shellcode; maps itself without LoadLibrary

// ETW patching (bypass logging):
// Patch EtwEventWrite in ntdll.dll to return immediately → evade ETW-based EDR
```

### C2 Communication Patterns

```
// HTTP/HTTPS beaconing:
GET /update?id=abc123&version=2.1    // check-in with unique identifier
POST /data                            // exfiltrate data

// DNS C2:
query: encoded_data.c2domain.com     // data encoded in subdomain
response TXT record: commands        // receive commands via DNS response

// Common evasion:
Domain Generation Algorithm (DGA): malware generates hundreds of domains daily; C2 registers one
Fast flux: C2 IP address changes frequently
Domain fronting: route traffic through CDN (Cloudflare, Azure Front Door)

// Identifying C2 in analysis:
Periodic HTTP requests (beaconing interval)
Custom User-Agent strings
RC4/XOR encrypted POST bodies
DNS requests to algorithmically-generated domains
SSL cert with self-signed or suspicious issuer
```

### Ransomware Analysis

```
// Key behaviors to identify:
1. Shadow copy deletion:
   vssadmin delete shadows /all /quiet
   wmic shadowcopy delete
   bcdedit /set {default} recoveryenabled No

2. File enumeration:
   FindFirstFile / FindNextFile recursive walk
   Target extensions: .docx, .xlsx, .pdf, .jpg, .db, etc.

3. Encryption scheme:
   Symmetric key (AES) per file
   Asymmetric key (RSA) wraps the AES key
   Encrypted AES key stored alongside file or in ransom note
   
4. Key storage:
   Key generated locally → must find key material in memory before reboot
   Key sent to C2 → no local recovery without paying
   Key derived from hostname/volume serial → sometimes recoverable

5. Ransom note drop:
   README.txt / HOW_TO_DECRYPT.html created in each directory
```

---

## Protocol Reverse Engineering

Analyzing undocumented binary protocols used by applications.

```
Approach:
  1. Capture traffic (Wireshark / tcpdump)
  2. Identify message boundaries (length prefixes, delimiters, fixed sizes)
  3. Identify message types (first byte/short often a type field)
  4. Correlate network traffic with application actions
  5. Cross-reference with API monitoring to find encode/decode functions
  6. Define the protocol grammar

Common patterns in binary protocols:
  Length-prefixed:  [4-byte length][payload]
  TLV (Type-Length-Value): [1-byte type][2-byte length][N-byte value]
  Magic header: DE AD BE EF (protocol identifier)
  Sequence numbers: for ordering and replay detection
  Checksums/CRC: last 4 bytes often CRC32

Tools:
  Wireshark custom dissectors (Lua scripts)
  010 Editor (binary templates for protocol structures)
  Scapy (Python protocol modeling)
  Canape (protocol fuzzing)
```

---

## CTF Reverse Engineering

Common challenge patterns and techniques in Capture The Flag competitions.

### Common Challenge Types

```
CrackMe:      Find the correct input/password/key
Flag check:   Binary validates input == flag; reverse the validation
Packer:       Unpack compressed/encrypted binary first
VM/bytecode:  Reverse a custom virtual machine
Algorithm:    Reverse a transformation and compute its inverse
Anti-debug:   Bypass protections to reach the flag check
```

### Quick Triage Methodology

```bash
# 1. Identify binary:
file chall && checksec chall

# 2. Find flag format reference:
strings chall | grep -i flag
strings chall | grep -E 'CTF|correct|wrong|password|key'

# 3. Trace execution:
ltrace ./chall                 # library calls
strace ./chall                 # syscalls
./chall <<< "AAAA"             # try wrong input; observe output

# 4. Find the comparison:
objdump -d chall | grep -A5 'strcmp\|memcmp\|strncmp'
# Set breakpoint on strcmp/memcmp → inspect arguments → one is likely the flag

# 5. Pin/angr symbolic execution for complex checks:
python3 solve.py               # automated constraint solving
```

### angr — Symbolic Execution

```python
import angr, claripy

proj = angr.Project('./chall', auto_load_libs=False)

# Create symbolic input:
flag = claripy.BVS('flag', 8 * 32)   # 32-byte symbolic bitvector

# Set up initial state with symbolic stdin:
state = proj.factory.full_init_state(
    stdin=flag,
    add_options={angr.options.LAZY_SOLVES}
)

# Add constraints (printable ASCII):
for byte in flag.chop(8):
    state.solver.add(byte >= 0x20)
    state.solver.add(byte <= 0x7e)

# Find path to "Correct" output, avoid "Wrong":
simgr = proj.factory.simulation_manager(state)
simgr.explore(
    find=lambda s: b'Correct' in s.posix.dumps(1),
    avoid=lambda s: b'Wrong' in s.posix.dumps(1)
)

if simgr.found:
    solution = simgr.found[0]
    print(solution.solver.eval(flag, cast_to=bytes))
```

### Z3 — SMT Solver for Constraint Problems

```python
from z3 import *

# Example: reverse a simple hash/transformation
x = [BitVec(f'x_{i}', 8) for i in range(8)]
s = Solver()

# Constraints extracted from binary:
s.add(x[0] + x[1] == 0xAB)
s.add(x[2] ^ x[3] == 0x12)
s.add(x[0] * x[2] == 0x1234)
# ... add all constraints from binary analysis ...

if s.check() == sat:
    m = s.model()
    flag = bytes([m[x[i]].as_long() for i in range(8)])
    print(flag)
```

---

## Firmware Reverse Engineering

Analyzing embedded device firmware (routers, IoT, medical devices).

```bash
# Extract firmware:
binwalk -e firmware.bin                # extract embedded filesystems, archives
binwalk -Me firmware.bin               # recursive extraction
dd if=firmware.bin bs=1 skip=<offset> of=rootfs.img   # manual extraction

# Identify architecture:
binwalk -A firmware.bin                # CPU architecture scan
file extracted/*                       # check extracted files
readelf -h busybox                     # if ELF binary found

# Common embedded architectures:
MIPS (big/little endian) → routers (Broadcom, Atheros)
ARM (32/64-bit)           → most modern IoT
PowerPC                   → older networking gear
x86                       → industrial systems, modern NAS

# Emulate firmware with QEMU:
qemu-mipsel-static ./busybox sh        # MIPS LE
qemu-arm-static ./binary               # ARM

# Full system emulation (Firmadyne, QEMU):
# Mount filesystem, set up networking, boot the OS

# Find attack surface:
find . -name "*.cgi" -o -name "*.sh"   # web interface scripts
grep -r "system(" . --include="*.c"    # dangerous function usage
grep -r "eval(" . --include="*.sh"
strings httpd | grep strcpy            # unsafe functions in web server

# Hardcoded credentials:
grep -r "password" . 2>/dev/null
grep -r "admin" . 2>/dev/null
strings shadow passwd                   # password hashes
```

---

## Essential Tools Reference

| Category | Tool | Platform | Purpose |
|---|---|---|---|
| **Disassembler/Decompiler** | IDA Pro | Win/Lin/Mac | Industry standard |
| | Ghidra | All | Free, NSA, excellent |
| | Binary Ninja | All | Modern, scriptable |
| | Radare2 / Cutter | All | Free, CLI power |
| **Debugger** | x64dbg | Windows | Best free Windows debugger |
| | WinDbg | Windows | Kernel debugging |
| | GDB + GEF | Linux | Standard Linux debugger |
| | LLDB | macOS/Linux | macOS, iOS debugging |
| **Dynamic Instrumentation** | Frida | All | Script-based hooking |
| | DynamoRIO | All | Dynamic binary instrumentation |
| | Pin (Intel) | All | Instrumentation framework |
| **Binary Analysis** | pwntools | Linux | CTF/exploit development |
| | angr | All | Symbolic execution |
| | Z3 | All | SMT constraint solving |
| | Triton | All | Concolic execution |
| **Malware Analysis** | PE-bear | Windows | PE format analysis |
| | CFF Explorer | Windows | PE editor |
| | x64dbg | Windows | Dynamic analysis |
| | PEiD / DIE | Windows | Packer detection |
| | Scylla | Windows | Unpacking / IAT fix |
| **Network** | Wireshark | All | Packet capture |
| | mitmproxy | All | HTTP/S interception |
| | Fiddler | Windows | HTTP proxy |
| **Firmware** | binwalk | Linux | Firmware extraction |
| | Firmadyne | Linux | Firmware emulation |
| | QEMU | All | System emulation |
| **String Analysis** | FLOSS | All | Extract obfuscated strings |
| | strings | All | Basic string extraction |
| **Hex Editors** | 010 Editor | All | Binary templates |
| | HxD | Windows | Fast, free |
| | xxd / hexdump | Linux | CLI hex inspection |

---

## Quick Reference Card

```
FIRST STEPS ON ANY BINARY
  file binary              → format, arch, linking
  strings binary           → hardcoded data, URLs, commands
  binwalk binary           → embedded content, entropy
  readelf -a / dumpbin     → sections, imports, exports
  detect-it-easy           → packer/compiler detection
  sha256sum → VirusTotal   → known malware check

KEY REGISTERS (x86-64)
  rax  return value / syscall #     rip  instruction pointer
  rdi  arg1    rsi  arg2            rsp  stack pointer
  rdx  arg3    rcx  arg4            rbp  frame pointer
  r8   arg5    r9   arg6            rflags  condition codes

COMMON ASSEMBLY PATTERNS
  xor eax,eax          → eax = 0 (fast zero)
  test eax,eax + jz    → if (eax == NULL)
  lea rax,[rbp-0x20]   → rax = &local_variable
  mov eax,[rbx+0x18]   → eax = object->field_at_0x18
  rep movsb            → memcpy
  repne scasb          → strlen
  call + pop           → get current RIP (PIC / position-independent)

GDB QUICK COMMANDS
  b *0xADDR    break     si/ni  step instruction
  r            run       c      continue
  x/20i $rip   disasm    x/20wx $rsp  stack
  p/x $rax     register  info proc mappings

ANTI-DEBUG BYPASSES
  IsDebuggerPresent   → patch to return 0
  Timing check        → use hardware breakpoints
  INT 3 scan          → use HW BP (no 0xCC in memory)
  Process name check  → rename debugger executable

ROP BUILDING BLOCKS
  pop rdi; ret   → set arg1
  pop rsi; ret   → set arg2
  pop rdx; ret   → set arg3
  syscall; ret   → trigger syscall
  ret            → alignment gadget

CTF QUICK WIN
  ltrace/strace  → see strcmp arguments live
  angr           → symbolic execution for complex checks
  Z3             → solve math constraints
  Frida          → hook return value → bypass at runtime
```
