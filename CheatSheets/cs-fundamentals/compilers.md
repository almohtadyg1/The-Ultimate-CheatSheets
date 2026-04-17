# Compilers: A Complete Progressive Tutorial

---

## 1. What & Why

A compiler translates source code written in one language into another form — typically machine code or an intermediate representation — while preserving the program's exact meaning. The compiler is the engine that bridges the gap between human-readable code and CPU instructions.

Why study compilers? Even if you never write a compiler, understanding compilation helps you write better code. Why does inlining a function make it faster? Compiler optimization. Why does Python code run slower than C? Interpreted vs compiled execution models. Why does a Rust program catch errors that C misses? The type system and borrow checker in the compiler. Why can a JIT-compiled language beat native C in some benchmarks? Runtime profiling enables optimizations the compiler can't make statically.

More practically: you will likely write domain-specific languages, configuration parsers, or template processors during your career. Every one of these involves parsing text into a structured form and evaluating or transforming it — the same concepts that govern compilation.

---

## 2. Mental Model

A compiler is a pipeline of transformations, each converting one representation to another:

```
Source text
    │ Lexing (tokenization)
    ▼
Token stream: [IF, LPAREN, ID("x"), GT, NUM(0), RPAREN, LBRACE, ...]
    │ Parsing
    ▼
Abstract Syntax Tree (AST): IfStmt(cond=BinOp(x, >, 0), then=Block(...))
    │ Semantic Analysis (type checking, name resolution)
    ▼
Annotated AST: types inferred, names resolved to declarations
    │ IR Generation
    ▼
Intermediate Representation: platform-independent three-address code
    │ Optimization (constant folding, inlining, dead code elimination)
    ▼
Optimized IR
    │ Code Generation (register allocation, instruction selection)
    ▼
Assembly / Machine Code → Executable binary

Front end:  source → IR  (language-specific)
Middle end: IR → IR  (language-independent optimizations)
Back end:   IR → code (target-specific)

This separation lets compilers like LLVM support many languages (C, C++, Rust, Swift)
targeting many architectures (x86, ARM, RISC-V, WASM) by mixing and matching ends.
```

---

## 3. Progressive Examples

### Level 1: Lexing — Text to Tokens

```python
# The lexer (tokenizer) converts raw source text into a stream of tokens.
# Each token has a type (KEYWORD, IDENTIFIER, NUMBER, OPERATOR, etc.) and a value.

import re
from dataclasses import dataclass
from enum import Enum, auto
from typing import Iterator

class TokenType(Enum):
    # Keywords
    IF = auto(); ELSE = auto(); WHILE = auto()
    RETURN = auto(); INT = auto(); LET = auto()
    # Literals
    INTEGER = auto(); FLOAT = auto(); STRING = auto()
    # Identifiers and operators
    IDENTIFIER = auto()
    PLUS = auto(); MINUS = auto(); STAR = auto(); SLASH = auto()
    EQUALS = auto(); EQEQ = auto(); NEQ = auto()
    LT = auto(); GT = auto(); LEQ = auto(); GEQ = auto()
    LPAREN = auto(); RPAREN = auto()
    LBRACE = auto(); RBRACE = auto()
    SEMICOLON = auto(); COMMA = auto()
    EOF = auto()

@dataclass
class Token:
    type: TokenType
    value: str
    line: int
    col: int

    def __repr__(self):
        return f"Token({self.type.name}, {self.value!r}, {self.line}:{self.col})"

class Lexer:
    KEYWORDS = {
        "if": TokenType.IF, "else": TokenType.ELSE,
        "while": TokenType.WHILE, "return": TokenType.RETURN,
        "int": TokenType.INT, "let": TokenType.LET,
    }

    TOKEN_PATTERNS = [
        (r"\d+\.\d+",          TokenType.FLOAT),
        (r"\d+",               TokenType.INTEGER),
        (r'"[^"]*"',           TokenType.STRING),
        (r"[a-zA-Z_]\w*",      TokenType.IDENTIFIER),
        (r"==",                TokenType.EQEQ),
        (r"!=",                TokenType.NEQ),
        (r"<=",                TokenType.LEQ),
        (r">=",                TokenType.GEQ),
        (r"=",                 TokenType.EQUALS),
        (r"\+",                TokenType.PLUS),
        (r"-",                 TokenType.MINUS),
        (r"\*",                TokenType.STAR),
        (r"/",                 TokenType.SLASH),
        (r"<",                 TokenType.LT),
        (r">",                 TokenType.GT),
        (r"\(",                TokenType.LPAREN),
        (r"\)",                TokenType.RPAREN),
        (r"\{",                TokenType.LBRACE),
        (r"\}",                TokenType.RBRACE),
        (r";",                 TokenType.SEMICOLON),
        (r",",                 TokenType.COMMA),
    ]

    def __init__(self, source: str):
        self.source = source
        self.pos = 0
        self.line = 1
        self.col = 1

    def tokenize(self) -> list[Token]:
        tokens = []
        while self.pos < len(self.source):
            # Skip whitespace
            if self.source[self.pos].isspace():
                if self.source[self.pos] == '\n':
                    self.line += 1; self.col = 1
                else:
                    self.col += 1
                self.pos += 1
                continue
            # Skip comments
            if self.source[self.pos:self.pos+2] == "//":
                while self.pos < len(self.source) and self.source[self.pos] != '\n':
                    self.pos += 1
                continue

            matched = False
            for pattern, tok_type in self.TOKEN_PATTERNS:
                m = re.match(pattern, self.source[self.pos:])
                if m:
                    value = m.group(0)
                    # Check if identifier is a keyword
                    if tok_type == TokenType.IDENTIFIER and value in self.KEYWORDS:
                        tok_type = self.KEYWORDS[value]
                    tokens.append(Token(tok_type, value, self.line, self.col))
                    self.col += len(value)
                    self.pos += len(value)
                    matched = True
                    break

            if not matched:
                raise SyntaxError(f"Unexpected character: {self.source[self.pos]!r} at {self.line}:{self.col}")

        tokens.append(Token(TokenType.EOF, "", self.line, self.col))
        return tokens

# Test it
source = """
let x = 10;
if (x > 5) {
    return x + 1;
}
"""
lexer = Lexer(source)
tokens = lexer.tokenize()
for tok in tokens:
    print(tok)
```

### Level 2: Parsing — Tokens to AST

```python
# The parser takes the token stream and produces an Abstract Syntax Tree (AST).
# The AST represents the grammatical structure of the program.

# Grammar for our tiny language (in BNF):
# program   ::= statement*
# statement ::= let_stmt | if_stmt | return_stmt | expr_stmt
# let_stmt  ::= "let" IDENTIFIER "=" expression ";"
# if_stmt   ::= "if" "(" expression ")" "{" statement* "}"
# return_stmt ::= "return" expression ";"
# expression ::= comparison
# comparison ::= addition (("==" | "!=" | "<" | ">") addition)*
# addition   ::= primary (("+" | "-") primary)*
# primary    ::= INTEGER | FLOAT | STRING | IDENTIFIER | "(" expression ")"

from dataclasses import dataclass, field
from typing import Optional, Any

# AST Node types
@dataclass
class NumberLit:
    value: float

@dataclass
class StringLit:
    value: str

@dataclass
class Identifier:
    name: str

@dataclass
class BinOp:
    left: Any
    operator: str
    right: Any

@dataclass
class LetStmt:
    name: str
    value: Any

@dataclass
class IfStmt:
    condition: Any
    then_body: list

@dataclass
class ReturnStmt:
    value: Any

class Parser:
    def __init__(self, tokens: list[Token]):
        self.tokens = tokens
        self.pos = 0

    def current(self) -> Token:
        return self.tokens[self.pos]

    def peek(self, offset=1) -> Token:
        return self.tokens[min(self.pos + offset, len(self.tokens) - 1)]

    def consume(self, expected_type: TokenType = None) -> Token:
        tok = self.current()
        if expected_type and tok.type != expected_type:
            raise SyntaxError(f"Expected {expected_type.name}, got {tok.type.name} at {tok.line}:{tok.col}")
        self.pos += 1
        return tok

    def parse_program(self) -> list:
        stmts = []
        while self.current().type != TokenType.EOF:
            stmts.append(self.parse_statement())
        return stmts

    def parse_statement(self):
        tok = self.current()
        if tok.type == TokenType.LET:
            return self.parse_let()
        elif tok.type == TokenType.IF:
            return self.parse_if()
        elif tok.type == TokenType.RETURN:
            return self.parse_return()
        else:
            expr = self.parse_expression()
            self.consume(TokenType.SEMICOLON)
            return expr

    def parse_let(self):
        self.consume(TokenType.LET)
        name = self.consume(TokenType.IDENTIFIER).value
        self.consume(TokenType.EQUALS)
        value = self.parse_expression()
        self.consume(TokenType.SEMICOLON)
        return LetStmt(name=name, value=value)

    def parse_if(self):
        self.consume(TokenType.IF)
        self.consume(TokenType.LPAREN)
        cond = self.parse_expression()
        self.consume(TokenType.RPAREN)
        self.consume(TokenType.LBRACE)
        body = []
        while self.current().type != TokenType.RBRACE:
            body.append(self.parse_statement())
        self.consume(TokenType.RBRACE)
        return IfStmt(condition=cond, then_body=body)

    def parse_return(self):
        self.consume(TokenType.RETURN)
        value = self.parse_expression()
        self.consume(TokenType.SEMICOLON)
        return ReturnStmt(value=value)

    def parse_expression(self):
        return self.parse_comparison()

    def parse_comparison(self):
        left = self.parse_addition()
        ops = {TokenType.EQEQ: "==", TokenType.NEQ: "!=",
               TokenType.LT: "<", TokenType.GT: ">",
               TokenType.LEQ: "<=", TokenType.GEQ: ">="}
        while self.current().type in ops:
            op = ops[self.consume().type]
            right = self.parse_addition()
            left = BinOp(left=left, operator=op, right=right)
        return left

    def parse_addition(self):
        left = self.parse_primary()
        while self.current().type in (TokenType.PLUS, TokenType.MINUS):
            op = self.consume().value
            right = self.parse_primary()
            left = BinOp(left=left, operator=op, right=right)
        return left

    def parse_primary(self):
        tok = self.current()
        if tok.type == TokenType.INTEGER:
            self.consume()
            return NumberLit(value=int(tok.value))
        elif tok.type == TokenType.FLOAT:
            self.consume()
            return NumberLit(value=float(tok.value))
        elif tok.type == TokenType.STRING:
            self.consume()
            return StringLit(value=tok.value[1:-1])  # strip quotes
        elif tok.type == TokenType.IDENTIFIER:
            self.consume()
            return Identifier(name=tok.value)
        elif tok.type == TokenType.LPAREN:
            self.consume(TokenType.LPAREN)
            expr = self.parse_expression()
            self.consume(TokenType.RPAREN)
            return expr
        else:
            raise SyntaxError(f"Unexpected token: {tok}")

# Test
tokens = Lexer("let x = 10; if (x > 5) { return x + 1; }").tokenize()
parser = Parser(tokens)
ast = parser.parse_program()
for node in ast:
    print(node)
```

### Level 3: Semantic Analysis and Interpretation

```python
# Semantic analysis checks meaning: types, scopes, undefined variables.
# After building the AST, we can either interpret it directly or compile it.

class Environment:
    """Symbol table with lexical scoping."""
    def __init__(self, parent=None):
        self.vars = {}
        self.parent = parent

    def get(self, name: str):
        if name in self.vars:
            return self.vars[name]
        if self.parent:
            return self.parent.get(name)
        raise NameError(f"Undefined variable: {name}")

    def set(self, name: str, value):
        self.vars[name] = value

class Interpreter:
    """Tree-walking interpreter for our tiny language."""

    def __init__(self):
        self.global_env = Environment()

    def eval(self, node, env: Environment = None):
        if env is None:
            env = self.global_env

        if isinstance(node, list):
            result = None
            for stmt in node:
                result = self.eval(stmt, env)
                if isinstance(result, ReturnValue):
                    return result
            return result

        elif isinstance(node, NumberLit):
            return node.value

        elif isinstance(node, StringLit):
            return node.value

        elif isinstance(node, Identifier):
            return env.get(node.name)

        elif isinstance(node, BinOp):
            left = self.eval(node.left, env)
            right = self.eval(node.right, env)
            ops = {"+": lambda a,b: a+b, "-": lambda a,b: a-b,
                   "*": lambda a,b: a*b, "/": lambda a,b: a/b,
                   "==": lambda a,b: a==b, "!=": lambda a,b: a!=b,
                   "<": lambda a,b: a<b, ">": lambda a,b: a>b}
            return ops[node.operator](left, right)

        elif isinstance(node, LetStmt):
            value = self.eval(node.value, env)
            env.set(node.name, value)
            return value

        elif isinstance(node, IfStmt):
            if self.eval(node.condition, env):
                inner_env = Environment(parent=env)
                return self.eval(node.then_body, inner_env)

        elif isinstance(node, ReturnStmt):
            return ReturnValue(self.eval(node.value, env))

class ReturnValue:
    def __init__(self, value):
        self.value = value

# Test our interpreter
source = "let x = 10; if (x > 5) { return x + 1; }"
tokens = Lexer(source).tokenize()
ast = Parser(tokens).parse_program()
interp = Interpreter()
result = interp.eval(ast)
print(f"Result: {result.value if isinstance(result, ReturnValue) else result}")   # 11
```

### Level 4: Compiler Optimizations

```python
# Optimizations transform the AST/IR to make programs faster or smaller.
# They must be semantics-preserving: the program must behave the same.

class ConstantFolding:
    """
    Constant folding: evaluate constant expressions at compile time.
    2 + 3 → 5, "hello" + " world" → "hello world"
    This is one of the most impactful and simplest optimizations.
    """

    def optimize(self, node):
        if isinstance(node, list):
            return [self.optimize(n) for n in node]

        elif isinstance(node, BinOp):
            left = self.optimize(node.left)
            right = self.optimize(node.right)

            # If both sides are constants, evaluate now
            if isinstance(left, NumberLit) and isinstance(right, NumberLit):
                ops = {"+": left.value + right.value,
                       "-": left.value - right.value,
                       "*": left.value * right.value,
                       "/": left.value / right.value if right.value != 0 else None}
                if node.operator in ops and ops[node.operator] is not None:
                    return NumberLit(ops[node.operator])

            return BinOp(left=left, operator=node.operator, right=right)

        elif isinstance(node, LetStmt):
            return LetStmt(name=node.name, value=self.optimize(node.value))

        elif isinstance(node, IfStmt):
            cond = self.optimize(node.condition)
            # Dead code elimination: if condition is always true/false
            if isinstance(cond, NumberLit):
                if cond.value:  # always true
                    return self.optimize(node.then_body)   # keep body
                else:
                    return []   # eliminate entire if — dead code!
            return IfStmt(condition=cond, then_body=self.optimize(node.then_body))

        return node

# Example: before folding
source = "let result = 2 + 3 * 4; if (1) { return result; }"
tokens = Lexer(source).tokenize()
ast = Parser(tokens).parse_program()
print("Before:", ast)

# After constant folding
optimizer = ConstantFolding()
optimized = optimizer.optimize(ast)
print("After:", optimized)
# let result = 14; return result;  (if(1) eliminated, 2+3*4=14 folded)
```

### Level 5: Code Generation

```python
# Code generation converts the optimized AST/IR into target code.
# We'll generate simple stack-machine bytecode.

from enum import Enum

class Opcode(Enum):
    PUSH = "PUSH"      # push constant onto stack
    LOAD = "LOAD"      # load variable onto stack
    STORE = "STORE"    # pop stack, store in variable
    ADD = "ADD"        # pop two, push sum
    SUB = "SUB"        # pop two, push difference
    MUL = "MUL"        # pop two, push product
    CMP_GT = "CMP_GT"  # pop two, push 1 if first > second else 0
    JMP_F = "JMP_F"    # jump to address if top of stack is 0 (false)
    JMP = "JMP"        # unconditional jump
    RETURN = "RETURN"  # return top of stack

@dataclass
class Instruction:
    opcode: Opcode
    operand: Any = None

    def __repr__(self):
        if self.operand is not None:
            return f"{self.opcode.value} {self.operand}"
        return self.opcode.value

class CodeGenerator:
    def __init__(self):
        self.instructions = []
        self.label_counter = 0

    def emit(self, opcode: Opcode, operand=None):
        self.instructions.append(Instruction(opcode, operand))
        return len(self.instructions) - 1   # return instruction index

    def patch(self, idx: int, operand):
        self.instructions[idx].operand = operand

    def new_label(self):
        self.label_counter += 1
        return self.label_counter

    def generate(self, node):
        if isinstance(node, list):
            for stmt in node:
                self.generate(stmt)

        elif isinstance(node, NumberLit):
            self.emit(Opcode.PUSH, node.value)

        elif isinstance(node, Identifier):
            self.emit(Opcode.LOAD, node.name)

        elif isinstance(node, BinOp):
            self.generate(node.left)
            self.generate(node.right)
            ops = {"+": Opcode.ADD, "-": Opcode.SUB, "*": Opcode.MUL}
            if node.operator in ops:
                self.emit(ops[node.operator])
            elif node.operator == ">":
                self.emit(Opcode.CMP_GT)

        elif isinstance(node, LetStmt):
            self.generate(node.value)
            self.emit(Opcode.STORE, node.name)

        elif isinstance(node, IfStmt):
            self.generate(node.condition)
            jump_if_false = self.emit(Opcode.JMP_F, None)   # placeholder
            self.generate(node.then_body)
            # Patch the jump to point past the body
            self.patch(jump_if_false, len(self.instructions))

        elif isinstance(node, ReturnStmt):
            self.generate(node.value)
            self.emit(Opcode.RETURN)

    def get_code(self):
        return self.instructions

# Test code generation
source = "let x = 10; if (x > 5) { return x + 1; }"
tokens = Lexer(source).tokenize()
ast = Parser(tokens).parse_program()
gen = CodeGenerator()
gen.generate(ast)
for i, instr in enumerate(gen.get_code()):
    print(f"{i:3}: {instr}")
```

### Level 6: LLVM and Modern Compiler Infrastructure

```python
# LLVM is the dominant compiler infrastructure.
# Using llvmlite (Python bindings for LLVM IR generation)
# pip install llvmlite

from llvmlite import ir, binding

# LLVM IR: strongly typed, SSA form, platform-independent
def generate_llvm_add_function():
    # Create a module (compilation unit)
    module = ir.Module(name="example")
    module.triple = binding.get_default_triple()

    # Define function type: int add(int a, int b)
    i32 = ir.IntType(32)
    func_type = ir.FunctionType(i32, [i32, i32])
    func = ir.Function(module, func_type, name="add")

    # Create entry basic block
    block = func.append_basic_block(name="entry")
    builder = ir.IRBuilder(block)

    # Add the parameters
    a, b = func.args
    a.name = "a"
    b.name = "b"

    result = builder.add(a, b, name="result")
    builder.ret(result)

    return str(module)

llvm_ir = generate_llvm_add_function()
print(llvm_ir)
# Output:
# define i32 @add(i32 %a, i32 %b) {
# entry:
#   %result = add i32 %a, %b
#   ret i32 %result
# }

# LLVM can then optimize this IR and generate native code for any target.
# Clang compiles C/C++, rustc compiles Rust, Swift compiles Swift — all emit LLVM IR.
# LLVM handles optimization passes and machine code generation.
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Parsing with regular expressions for complex grammars**

```python
# WRONG: trying to parse nested structures with regex
import re
# Can you parse nested parentheses with a single regex? NO.
# Regular expressions cannot handle recursive/nested structures.
# They can only recognize "regular" languages (no nesting).

# For balanced parentheses, you need a context-free grammar and a real parser.

# CORRECT: use a proper parser (hand-written recursive descent, or a parser generator)
# Python: PLY, lark, parso, antlr4
# JavaScript: PEG.js, nearley
```

**Mistake 2: Not building the AST, evaluating during parsing instead**

```python
# WRONG: evaluate as you parse — no way to optimize, transform, or analyze first
def parse_and_eval(tokens):
    if tokens[0] == "+":
        return parse_and_eval(tokens[1:]) + parse_and_eval(tokens[2:])

# CORRECT: build AST first, then analyze/optimize/evaluate separately
# This separation allows: constant folding, type checking, multiple backends
```

**Mistake 3: Forgetting that compiler errors need location information**

Users expect error messages like `error at line 42, column 5: undefined variable 'x'`. If you don't track line/column numbers in your tokens during lexing, you cannot produce useful error messages. Always attach source location to tokens and AST nodes.

**Mistake 4: Not handling left recursion in recursive descent parsers**

```python
# WRONG: left-recursive grammar causes infinite recursion
# expression ::= expression "+" term | term
def parse_expression():
    return parse_expression() + "+" + parse_term()  # infinite recursion!

# CORRECT: rewrite as right-recursive or use iteration
# expression ::= term ("+" term)*
def parse_expression():
    left = parse_term()
    while current_token == "+":
        consume("+")
        right = parse_term()
        left = BinOp(left, "+", right)
    return left
```

---

## 5. The "Why Does This Work" Layer

### Why SSA (Static Single Assignment) Enables Optimizations

Modern compilers convert their IR to SSA form: each variable is assigned exactly once. If `x` is assigned in two branches of an if-else, they become `x1` (in the then-branch) and `x2` (in the else-branch), joined by a special `phi` instruction at the merge point.

SSA makes many optimizations trivial: if `x1 = 5` and `y = x1 + 2`, constant propagation can replace `y` with `7` immediately. Without SSA, tracking which value of `x` is in effect at each point requires expensive data-flow analysis.

### Why Inlining Is Often the Most Powerful Optimization

When a function is inlined, its body replaces the call site. This enables: the caller and callee to be optimized together (cross-function constant propagation, type specialization), elimination of function call overhead (argument passing, stack frame setup), and better branch prediction (the inlined code runs in context with surrounding code).

This is why performance-critical languages (C++, Rust) aggressively inline small functions and why interpreted languages (Python) have much more overhead per function call.

---

## 6. Quick Reference

### Compilation Phases

| Phase | Input | Output | Examples of Analysis |
|-------|-------|--------|---------------------|
| Lexing | Source text | Tokens | `if`, `x`, `123`, `+` |
| Parsing | Tokens | AST | Grammar rules, syntax errors |
| Semantic analysis | AST | Annotated AST | Types, scope, undefined names |
| IR generation | Annotated AST | IR | Three-address code, SSA |
| Optimization | IR | Optimized IR | Constant folding, inlining, DCE |
| Code generation | IR | Assembly | Register allocation, instruction selection |

### Common Compiler Optimizations

| Optimization | Description | Example |
|-------------|-------------|---------|
| Constant folding | Evaluate constants at compile time | `2 + 3` → `5` |
| Dead code elimination | Remove unreachable code | `if (false) {...}` → removed |
| Inlining | Replace call with function body | `f(x)` → body of `f` |
| Loop invariant code motion | Move loop-independent code out | `c = a*b` moved before loop |
| Common subexpression elimination | Avoid recomputing the same value | `a*b + a*b` → `t=a*b; t+t` |
| Strength reduction | Replace expensive op with cheap | `x * 2` → `x << 1` |
| Loop unrolling | Expand loop body to reduce overhead | 4 iterations per loop cycle |

### Tools

| Tool | Purpose |
|------|---------|
| Lex/Flex | C lexer generator |
| Yacc/Bison | C parser generator (LALR) |
| ANTLR | Multi-language parser generator |
| PLY/lark | Python parser libraries |
| LLVM/Clang | Modern compiler infrastructure |
| GCC | GNU Compiler Collection |
| `clang -emit-llvm` | Emit LLVM IR for inspection |
| `gcc -S` | Emit assembly output |
