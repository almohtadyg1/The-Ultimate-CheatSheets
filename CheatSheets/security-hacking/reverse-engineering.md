# Reverse Engineering: A Complete Progressive Tutorial

> **Legal context:** Reverse engineering is legal for security research, interoperability (EU Software Directive, DMCA §1201(f)), malware analysis, and personal education. Always obtain authorization before analyzing proprietary software in a commercial context. This tutorial is for defensive security research, CTF competitions, and education.

---

## 1. What & Why

Reverse engineering is the process of analyzing a compiled binary, firmware, network protocol, or running system to understand its structure and behavior — without access to the original source code. It is the foundation of malware analysis, vulnerability research, CTF competitions, interoperability work, and exploit development.

Why learn this? Because compiled binaries are everywhere — in every malware sample, every unknown network protocol, every firmware update, every closed-source application you need to integrate with. Security researchers, malware analysts, CTF competitors, and exploit developers all need to read assembly and understand how programs behave at the machine level. Even if you only write high-level Python, understanding what happens at the binary level makes you a better debugger, a better performance engineer, and a significantly harder target for attacks on your own code.

---

## 2. Mental Model

Reverse engineering works backward through the compilation process:

```
Source Code (C/C++/Rust...)
       │
       ▼ Compilation
Machine Code (raw bytes: e8 1a 00 00 00 ...)
       │
       ▼ Disassembly (bytes → assembly instructions)
Assembly (mov rax, 0x1 / call printf / ret ...)
       │
       ▼ Decompilation (assembly → pseudo-C)
Pseudo-C (function names lost, logic preserved)
       │
       ▼ Your analysis
Recovered Intent (what the program actually does)

Tools at each level:
  Binary identification: file, binwalk, strings, xxd
  Static analysis:       Ghidra (free), IDA Pro, Binary Ninja, Cutter
  Dynamic analysis:      gdb, x64dbg, strace, ltrace, frida
  Network protocols:     Wireshark, scapy

Key insight: You rarely need to understand every instruction.
You need to answer specific questions.
Work top-down: overall behavior → relevant functions → critical paths → instructions.
```

---

## 3. Progressive Examples

### Level 1: Basic Binary Analysis

```bash
# --- Step 1: What are we dealing with? ---

file target_binary
# ELF 64-bit LSB executable, x86-64, dynamically linked
# Or: PE32+ executable (MS Windows), Mach-O 64-bit x86_64

# Magic bytes (identify format without extension)
xxd target_binary | head -2
# 7f 45 4c 46  → ELF (Linux)
# 4d 5a        → MZ/PE (Windows)
# ca fe ba be  → Mach-O (macOS, big-endian)
# cf fa ed fe  → Mach-O (macOS, little-endian)
# 50 4b 03 04  → ZIP (also: .jar, .docx, .apk)

# Check for known packers/obfuscation
strings target_binary | grep -i "upx\|packed\|compress"
# UPX packed binaries: upx -d target_binary (unpack before analysis)

# --- Step 2: Extract strings ---
strings target_binary                    # all printable strings (min 4 chars)
strings -n 8 target_binary              # strings of length >= 8
strings target_binary | grep -iE "http|password|key|token|flag"  # CTF patterns

# --- Step 3: What functions/symbols are present? ---
nm target_binary 2>/dev/null | grep -v " U "   # defined symbols (stripped = empty)
nm target_binary 2>/dev/null | grep " U "       # undefined (imported) symbols
readelf -s target_binary | grep FUNC             # ELF function symbols
objdump -p target_binary | grep NEEDED           # shared library dependencies

# --- Step 4: What system calls does it make? ---
strace ./target_binary                           # trace system calls (Linux)
strace -e trace=open,read,write,connect ./binary # specific syscalls only
ltrace ./target_binary                           # trace library calls (printf, malloc, etc.)

# Example strace output revealing behavior:
# open("/etc/passwd", O_RDONLY) = 3              → reads passwd
# connect(4, {AF_INET, 192.168.1.100, 4444}) = 0 → connects to C2 server!
# read(0, ...) + write(4, ...)                   → forwarding stdin to socket = shell!
```

### Level 2: Reading x86-64 Assembly

```asm
; Key x86-64 registers:
; rax     - function return value, accumulator
; rdi/rsi/rdx/rcx/r8/r9 - function arguments (in that order)
; rsp     - stack pointer (points to top of stack)
; rbp     - base pointer (points to current stack frame)
; rip     - instruction pointer (address of next instruction)

; Common instructions:
; mov  dst, src    - copy value from src to dst
; push reg         - decrement rsp, store reg on stack
; pop  reg         - load from stack into reg, increment rsp
; call addr        - push rip, jump to addr (function call)
; ret              - pop rip (return from function)
; add  dst, val    - dst = dst + val
; sub  dst, val    - dst = dst - val
; xor  reg, reg    - zero a register (xor rax, rax → rax = 0)
; cmp  a, b        - sets flags based on a - b (doesn't store result)
; test a, b        - sets flags based on a & b (often: test rax, rax)
; jz/je  addr      - jump if Zero flag set (last comparison was equal)
; jnz/jne addr     - jump if Zero flag NOT set
; jl/jg  addr      - jump if Less/Greater (signed)
; jb/ja  addr      - jump if Below/Above (unsigned)

; Reading a real function — password check:
;
; int check_password(char *input) {
;     if (strcmp(input, "secret123") == 0) return 1;
;     return 0;
; }
;
; In assembly (what you'd see in Ghidra/objdump):

check_password:
    push   rbp
    mov    rbp, rsp            ; set up stack frame
    sub    rsp, 0x10           ; allocate 16 bytes for locals
    mov    QWORD PTR [rbp-0x8], rdi   ; save first argument (input ptr)
    
    mov    rax, QWORD PTR [rbp-0x8]   ; load input into rax
    lea    rsi, [rip+0x2000]          ; address of string "secret123"
    mov    rdi, rax                    ; first arg to strcmp = input
    call   strcmp                      ; strcmp(input, "secret123")
    
    test   eax, eax            ; is return value 0?
    jnz    .not_equal          ; if not 0, jump to failure
    
    mov    eax, 0x1            ; return 1 (success)
    jmp    .done
.not_equal:
    mov    eax, 0x0            ; return 0 (failure)
.done:
    leave                      ; restore rbp from stack
    ret                        ; return to caller
```

```python
# Python helper: disassemble bytes for quick analysis
from capstone import Cs, CS_ARCH_X86, CS_MODE_64

def disassemble(shellcode: bytes, base_addr: int = 0x400000):
    """Quick disassembly using Capstone."""
    md = Cs(CS_ARCH_X86, CS_MODE_64)
    for inst in md.disasm(shellcode, base_addr):
        print(f"0x{inst.address:x}: {inst.mnemonic:10} {inst.op_str}")

# Example: analyze a shellcode sample
shellcode = bytes([
    0x48, 0x31, 0xc0,           # xor rax, rax
    0x48, 0x31, 0xff,           # xor rdi, rdi
    0xb0, 0x3c,                 # mov al, 0x3c (sys_exit = 60)
    0x0f, 0x05,                 # syscall
])
disassemble(shellcode)
# 0x400000: xor        rax, rax   (zero out rax)
# 0x400003: xor        rdi, rdi   (zero out rdi — exit code = 0)
# 0x400006: mov        al, 0x3c   (syscall 60 = exit)
# 0x400008: syscall               (make the syscall)
# This is: exit(0)
```

### Level 3: Static Analysis with Ghidra

```
GHIDRA WORKFLOW for analyzing an unknown binary:

1. CREATE PROJECT → Import File → auto-analyze
   Ghidra's auto-analysis: identifies functions, strings, cross-references, data types

2. CODE BROWSER → Symbol Tree → Functions
   Look at function names if not stripped.
   Start with: main, _start, or suspicious names

3. DECOMPILER WINDOW
   Ghidra's decompiler produces readable pseudo-C.
   It's not perfect but dramatically accelerates analysis.
   
   Real decompiler output:
   
   int FUN_004012a0(char *param_1) {
       int iVar1;
       iVar1 = strcmp(param_1, "secret123");
       return (int)(iVar1 == 0);
   }
   
   You can rename: FUN_004012a0 → check_password
                   param_1      → user_input

4. CROSS-REFERENCES (xrefs)
   Right-click any function/variable → References → Find All References
   See every place check_password is called from
   
5. SEARCH → For Strings
   Find hardcoded values: keys, passwords, C2 server addresses, flag patterns

6. USEFUL SHORTCUTS
   L        → rename symbol
   T        → retype variable
   Ctrl+F   → search for strings/instructions
   G        → go to address
   ;        → add comment at cursor
   B        → toggle bookmark

COMMON PATTERNS TO RECOGNIZE:

Stack cookie (stack canary):
   mov rax, QWORD PTR fs:0x28    ; load stack canary from TLS
   mov [rbp-0x8], rax            ; store on stack
   ... function body ...
   mov rdx, [rbp-0x8]            ; load back
   xor rdx, QWORD PTR fs:0x28    ; compare with original
   jne stack_smash_detected       ; if changed: stack overflow!

String comparison loop:
   xor eax, eax
   .loop:
   movsx ecx, BYTE PTR [rdi+rax]  ; load char from string 1
   movsx edx, BYTE PTR [rsi+rax]  ; load char from string 2  
   cmp ecx, edx                   ; compare
   jne .not_equal
   test ecx, ecx                  ; null terminator?
   jz .equal
   add rax, 0x1                   ; advance index
   jmp .loop

malloc/free pattern (heap allocation):
   mov edi, <size>
   call malloc                    ; allocate
   mov [rbp-0x8], rax             ; save pointer
   ... use allocated memory ...
   mov rdi, [rbp-0x8]
   call free                      ; deallocate
```

### Level 4: Dynamic Analysis with GDB

```bash
# GDB: GNU Debugger — step through code, inspect memory and registers

# Launch
gdb ./target_binary
gdb -p 1234              # attach to running process

# Essential commands
(gdb) info functions     # list all functions (if not stripped)
(gdb) disassemble main   # disassemble the main function
(gdb) break main         # set breakpoint at main
(gdb) break *0x400abc    # breakpoint at specific address
(gdb) break check_password  # breakpoint at function name

(gdb) run arg1 arg2      # start execution with arguments
(gdb) continue           # continue until next breakpoint
(gdb) next               # step over (don't enter function calls)
(gdb) step               # step into (enter function calls)
(gdb) finish             # run until current function returns
(gdb) nexti              # next machine instruction (not source line)

# Inspect state at a breakpoint
(gdb) info registers     # all registers
(gdb) p $rax             # print register value
(gdb) p $rdi             # first function argument
(gdb) p (char*)$rdi      # first arg interpreted as string
(gdb) x/20xb $rsp        # examine 20 bytes at stack pointer (hex bytes)
(gdb) x/s $rsi           # examine memory as string
(gdb) x/5i $rip          # examine 5 instructions at current position

# Modify execution (useful for bypassing checks)
(gdb) set $rax = 0       # change register value
(gdb) set {int}0x601234 = 42   # change memory value
(gdb) jump *0x400abc     # jump to address (skip code)

# PEDA/pwndbg: enhanced GDB plugins for security research
# pip install pwndbg  OR  git clone https://github.com/longld/peda
```

```python
# pwntools: Python library for binary exploitation and automation
from pwn import *

# Set up binary context
elf = ELF("./target_binary")
context.arch = "amd64"
context.log_level = "debug"

# Start a local process
p = process("./target_binary")

# Or connect to a remote CTF challenge
# p = remote("challenge.ctf.com", 1337)

# Send input
p.sendline(b"hello")              # send + newline
p.send(b"\x41" * 64)              # send raw bytes
p.sendlineafter(b"Password:", b"secret123")  # wait for prompt, then send

# Receive output
response = p.recv(1024)           # receive up to 1024 bytes
line = p.recvline()               # receive one line
p.recvuntil(b"flag{")             # receive until pattern
flag_start = p.recv(50)           # then receive flag content

# Build a simple exploit
offset_to_return = 72             # how many bytes until we overwrite return address
target_function = elf.symbols["win_function"]  # address to redirect to

payload = b"A" * offset_to_return + p64(target_function)  # pack as 64-bit little-endian
p.sendline(payload)
p.interactive()                   # hand control to user (for shell interaction)
```

### Level 5: Malware Analysis Methodology

```
MALWARE ANALYSIS METHODOLOGY:

STATIC ANALYSIS (no execution needed):
1. file, strings, binwalk — identify format and extract strings
2. AV scan: VirusTotal (virustotal.com) — check hash and upload (if not sensitive)
3. PE analysis: pestudio, CFF Explorer — imports, exports, resources
4. Strings analysis: hardcoded IPs, URLs, registry keys, mutexes, file paths
5. Disassembly: Ghidra/IDA — understand code logic
6. YARA rules: write signatures to detect this malware family

DYNAMIC ANALYSIS (controlled execution in sandbox):
1. Snapshot your VM before execution
2. Enable process monitoring: procmon (Windows), sysdig (Linux)
3. Enable network capture: Wireshark
4. Run the sample in the sandbox
5. Observe: new files created, registry changes, network connections, new processes
6. Sandbox automation: Cuckoo Sandbox, Any.run, Joe Sandbox

INDICATORS OF COMPROMISE (IOCs) to extract:
  Network: IP addresses, domains, URLs, SSL certificate fingerprints
  Host:    File paths, registry keys, mutex names, service names
  Behavioral: Process injection, credential dumping, lateral movement

COMMON TECHNIQUES TO IDENTIFY:
  Process injection: OpenProcess + WriteProcessMemory + CreateRemoteThread
  Persistence: HKCU\Software\Microsoft\Windows\CurrentVersion\Run
  Privilege escalation: impersonation tokens, UAC bypass
  C2 communication: HTTP/S beaconing, DNS tunneling, ICMP tunneling
  Anti-analysis: IsDebuggerPresent, timing checks, VM detection
  Packing/obfuscation: compressed sections, XOR-decoded strings at runtime
```

```python
# YARA rule for detecting a malware family
YARA_RULE = """
rule SuspiciousShellcodeLoader {
    meta:
        description = "Detects shellcode loader with common anti-analysis tricks"
        author = "Security Researcher"
        date = "2024-01-15"

    strings:
        $mz = { 4D 5A }                                   // MZ header
        $anti_debug = "IsDebuggerPresent"
        $inject_1 = "VirtualAllocEx"
        $inject_2 = "WriteProcessMemory"
        $inject_3 = "CreateRemoteThread"
        $url = /https?:\/\/[0-9]{1,3}\.[0-9]{1,3}/ wide   // IP-based URL

    condition:
        $mz at 0 and
        $anti_debug and
        2 of ($inject_*) and
        $url
}
"""
# Apply with: yara rule.yar suspicious_file.exe
```

### Level 6: CTF Binary Exploitation Basics

```python
from pwn import *

# BUFFER OVERFLOW: the classic
# If a function copies user input into a fixed-size buffer without bounds checking,
# writing past the buffer overwrites the return address on the stack.

# Finding the offset:
# 1. Generate a cyclic pattern
pattern = cyclic(200)   # "aaaabaaacaaadaaa..."
# 2. Run the program with the pattern, note what value was in the return address at crash
# 3. Find the offset
offset = cyclic_find(0x61616166)   # the value you saw in the crash → offset = 20 (example)
print(f"Offset to return address: {offset}")

# Simple ret2win exploit (jump to a win() function)
elf = ELF("./vuln")
win_addr = elf.symbols["win"]  # or: elf.sym["win"]

payload = flat(
    b"A" * offset,      # padding to fill the buffer
    p64(win_addr),      # overwrite return address with win()'s address
)

p = process("./vuln")
p.sendline(payload)
print(p.recvall())

# Common CTF vulnerability patterns:
# gets(), strcpy(), sprintf() without bounds → buffer overflow
# printf(user_input) → format string vulnerability (read/write arbitrary memory)
# Use-after-free → heap exploitation
# Integer overflow → unexpected size calculation
# Off-by-one → one extra byte overwrites adjacent memory

# Finding stack offset with GDB:
# (gdb) run $(python3 -c "import sys; sys.stdout.buffer.write(b'A'*200)")
# Crash → examine $rsp at crash → look for 'AAAA...' to find return address
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Trying to understand every instruction before starting**

Reverse engineering is top-down, not bottom-up. You don't read assembly like reading a book. You start with a question ("what does this function check?"), identify the relevant code paths, and read only what answers your question. Trying to understand every function in a large binary before starting to answer questions leads to analysis paralysis.

**Mistake 2: Ignoring strings analysis**

`strings` is often the highest-value, lowest-effort step. A binary that is a password manager will have strings like `"incorrect password"`, `"vault unlocked"`, and possibly the expected password itself. Malware often has hardcoded C2 server addresses, mutex names, and registry keys. Always run `strings` and grep for interesting patterns before diving into assembly.

**Mistake 3: Mistaking Ghidra's decompiler output for actual source code**

Decompiler output is pseudo-code — an approximation. Variable names are invented (`param_1`, `local_8`). Types are inferred, not known. Control flow is reconstructed and sometimes wrong. Use the decompiler as a starting point for understanding, not as ground truth. Cross-reference with the assembly view for anything important.

**Mistake 4: Running malware samples on your host machine**

Dynamic analysis requires a controlled environment: a VM with snapshots, no real credentials, no shared folders with the host, and ideally a separate physical network or VLAN. Malware that detects it's in a VM may behave differently (sandbox evasion), but running it on your actual machine is far worse.

---

## 5. The "Why Does This Work" Layer

### Why Buffer Overflows Work

C does not check array bounds. `gets(buf)` copies stdin into `buf` until a newline, regardless of `buf`'s declared size. The C calling convention places the return address on the stack, above the local variables (lower address than the buffer on x86). Writing past the buffer into the return address slot means when the function executes `ret`, it loads the attacker's address rather than the legitimate return address.

Modern mitigations: stack canaries (random value placed before the return address, checked before `ret`), ASLR (randomize load addresses), NX/DEP (mark stack non-executable). Each requires a separate technique to bypass in CTF challenges.

### How the PLT/GOT Works (Dynamic Linking)

When your program calls `printf`, the compiler doesn't know at compile time which address `printf` will have at runtime (ASLR moves the libc). Instead, it generates a call to `printf@plt` — a stub in the Procedure Linkage Table. The first time this is called, the PLT stub calls the dynamic linker, which resolves the real address and stores it in the Global Offset Table (GOT). Subsequent calls go: PLT stub → GOT entry → real function address. Understanding this is essential for GOT overwrite and ret2libc exploits.

---

## 6. Quick Reference

### Static Analysis Commands

```bash
file binary            # format identification
strings -n 8 binary    # strings of length >= 8
nm -D binary           # dynamic symbols (exports/imports)
ldd binary             # shared library dependencies
readelf -s binary      # ELF symbol table
objdump -d binary      # disassemble code sections
readelf -l binary      # program headers
xxd binary | head      # hex dump of first bytes
```

### GDB Quick Reference

```
break main      → breakpoint at function
break *0x1234   → breakpoint at address
run             → start execution
continue        → continue to next breakpoint
next/nexti      → step over (C line / instruction)
step/stepi      → step into (C line / instruction)
info registers  → show all registers
p $rax          → print register
x/s $rdi        → examine memory as string
x/20xb $rsp     → 20 hex bytes at stack pointer
set $rax = 0    → modify register
```

### x86-64 Register Usage

| Register | Typical Use |
|----------|------------|
| `rax` | Return value, accumulator |
| `rdi` | 1st function argument |
| `rsi` | 2nd function argument |
| `rdx` | 3rd function argument |
| `rcx` | 4th function argument |
| `r8`, `r9` | 5th, 6th function arguments |
| `rsp` | Stack pointer |
| `rbp` | Base pointer (stack frame) |
| `rip` | Instruction pointer |
