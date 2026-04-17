# Lua: A Complete Progressive Tutorial

---

## 1. What & Why

Lua is a lightweight, fast, embeddable scripting language designed for simplicity and extensibility. Written in clean ANSI C, it runs on virtually every platform and compiles to a runtime of about 200KB. This tiny footprint — combined with a clean C API for embedding — makes Lua the dominant choice when you need a scripting layer inside another application.

Where you encounter Lua in practice: Roblox (game scripting), World of Warcraft addons, LÖVE 2D game engine, Neovim configuration, nginx via OpenResty, Redis scripting, and countless game engines and embedded systems. If you're adding a user-programmable scripting layer to a C or C++ application, Lua is the standard choice.

The language itself is deliberately minimal: eight types, one compound data structure (the table), first-class functions, and prototype-based object orientation built on top of those tables. This minimalism is a feature — Lua is learnable in a day and the semantics have almost no edge cases.

---

## 2. Mental Model

Lua's data model is built on two principles: everything is a value, and tables are everything.

```
Lua's 8 types:
  nil         — absence of a value (also what undefined variables contain)
  boolean     — true or false
  number      — integer or float (both are "number" — math.type() distinguishes)
  string      — immutable byte sequence, interned
  table       — Lua's ONLY compound data structure (array, dict, object, class — all tables)
  function    — first-class, closures
  userdata    — C data exposed to Lua (opaque to Lua code)
  thread      — coroutine (not OS thread)

Truth: ONLY nil and false are falsy. Everything else is truthy.
  0 is truthy. "" is truthy. {} is truthy.
  This trips up every programmer coming from other languages.

Tables are everything:
  Array:   t = {10, 20, 30}     → t[1]=10, t[2]=20, t[3]=30  (1-indexed!)
  Dict:    t = {x=1, y=2}       → t["x"]=1, t.x=1
  Object:  t = {name="Alice", greet=function(self) ... end}
  Class:   use a table as a metatable/prototype

Scope: global unless marked `local`. Always use local.
  global_var = 1    -- accessible everywhere in the file/chunk
  local local_var = 2  -- block-scoped, faster access
```

---

## 3. Progressive Examples

### Level 1: Variables, Types, and Basic Operations

```lua
-- Variables: global by default. ALWAYS use local.
local name = "Alice"
local age = 30
local pi = 3.14159
local active = true
local nothing = nil

-- Type checking
print(type(42))        -- "number"
print(type("hello"))   -- "string"
print(type(nil))       -- "nil"
print(type({}))        -- "table"
print(type(print))     -- "function"
print(type(true))      -- "boolean"

-- Numbers: Lua 5.3+ distinguishes integers and floats
local i = 42            -- integer
local f = 42.0          -- float
local h = 0xFF          -- hex: 255

print(math.type(42))    -- "integer"
print(math.type(42.0))  -- "float"

-- Arithmetic
print(10 / 3)    -- 3.3333333... (/ is ALWAYS float division)
print(10 // 3)   -- 3 (floor division)
print(10 % 3)    -- 1 (modulo)
print(2 ^ 10)    -- 1024.0 (^ is ALWAYS float — result is float)

-- Strings: immutable, double or single quotes, long brackets [[...]]
local s1 = "double quotes"
local s2 = 'single quotes'
local s3 = [[
multi-line string
no escape processing here
]]

-- String concatenation: .. (NOT +)
local greeting = "Hello" .. ", " .. name .. "!"
print(greeting)    -- "Hello, Alice!"

-- String length
print(#"hello")    -- 5
print(#greeting)   -- depends on name

-- String methods (colon syntax calls method with string as first arg)
print(("hello"):upper())          -- "HELLO"
print(("  spaces  "):match("^%s*(.-)%s*$"))  -- "spaces" (trimmed)
print(string.format("%.2f", pi))  -- "3.14"

-- String formatting
local msg = string.format("Name: %s, Age: %d, Score: %.1f", name, age, 95.7)
print(msg)   -- "Name: Alice, Age: 30, Score: 95.7"

-- Truthiness: ONLY false and nil are falsy
if 0 then print("0 is truthy") end    -- prints (unlike C!)
if "" then print("'' is truthy") end   -- prints (unlike Python!)
if {} then print("{} is truthy") end   -- prints

-- Logical operators return operands, not booleans
local x = nil
local default = x or "fallback"    -- "fallback" (x is nil/falsy)
local y = 5
local doubled = y and y * 2         -- 10 (y is truthy, evaluates second)
```

### Level 2: Tables — Lua's Only Data Structure

```lua
-- Tables are arrays, dictionaries, objects, and classes — all in one.

-- Array (1-indexed — this is not negotiable in Lua)
local fruits = {"apple", "banana", "cherry"}
print(fruits[1])    -- "apple" (not 0!)
print(fruits[2])    -- "banana"
print(#fruits)      -- 3 (length operator)

fruits[4] = "date"        -- append
table.insert(fruits, "elderberry")   -- append with table library
table.insert(fruits, 2, "avocado")   -- insert at position 2 (shifts others right)
table.remove(fruits, 1)              -- remove at index 1

-- Iterate array
for i, fruit in ipairs(fruits) do   -- ipairs: stops at first nil gap
    print(i, fruit)
end

-- Dictionary
local user = {
    name = "Alice",
    age = 30,
    active = true,
}
user.email = "alice@example.com"   -- dot notation (syntactic sugar for user["email"])
user["score"] = 95

print(user.name)     -- "Alice"
print(user["age"])   -- 30
user.name = nil      -- delete a key (set to nil)

-- Iterate dictionary (order not guaranteed)
for key, value in pairs(user) do   -- pairs: all key-value pairs
    print(key, value)
end

-- Mixed table (array + dictionary)
local data = {
    "first",           -- data[1]
    "second",          -- data[2]
    label = "mixed",   -- data["label"]
    count = 2,         -- data["count"]
}

-- Table as a set (store values as keys, all values are true)
local seen = {}
seen["apple"] = true
seen["banana"] = true
if seen["apple"] then print("seen apple") end

-- Nested tables
local config = {
    server = {host = "localhost", port = 8080},
    database = {host = "db.local", port = 5432},
}
print(config.server.port)     -- 8080
print(config["database"]["port"])   -- 5432

-- table library
local t = {3, 1, 4, 1, 5, 9, 2, 6}
table.sort(t)                -- sort in-place (ascending)
table.sort(t, function(a, b) return a > b end)  -- sort descending
print(table.concat(t, ", "))  -- "9, 6, 5, 4, 3, 2, 1, 1"
```

### Level 3: Functions and Closures

```lua
-- Functions are first-class values in Lua

-- Basic function
local function add(a, b)
    return a + b
end
print(add(3, 4))   -- 7

-- Anonymous function (stored in variable)
local multiply = function(a, b)
    return a * b
end

-- Multiple return values (Lua speciality)
local function divmod(a, b)
    return a // b, a % b   -- returns TWO values
end

local q, r = divmod(17, 5)
print(q, r)   -- 3    2

-- Only the first value is used when needed in an expression
local q_only = divmod(17, 5)   -- q_only = 3, second value discarded

-- Variadic functions
local function sum(...)
    local args = {...}        -- pack varargs into a table
    local total = 0
    for _, v in ipairs(args) do
        total = total + v
    end
    return total
end
print(sum(1, 2, 3, 4, 5))   -- 15

-- select with varargs
local function describe(...)
    print("Count:", select('#', ...))   -- number of args
    print("From 2nd:", select(2, ...))  -- args from 2nd onward
end
describe("a", "b", "c", "d")
-- Count: 4
-- From 2nd: b   c   d

-- Closures: functions capture the enclosing scope
local function make_counter(start)
    local count = start or 0
    return {
        increment = function() count = count + 1 end,
        decrement = function() count = count - 1 end,
        value = function() return count end,
    }
end

local c = make_counter(10)
c.increment()
c.increment()
print(c.value())   -- 12

-- Higher-order functions
local function map(t, f)
    local result = {}
    for i, v in ipairs(t) do
        result[i] = f(v)
    end
    return result
end

local nums = {1, 2, 3, 4, 5}
local squares = map(nums, function(x) return x * x end)
-- {1, 4, 9, 16, 25}
```

### Level 4: Metatables and Object-Oriented Programming

```lua
-- Metatables: attach behavior to tables (operator overloading, prototype chains)

-- A "class" in Lua is a table used as a prototype
local Animal = {}
Animal.__index = Animal   -- when field not found on instance, look in Animal

function Animal.new(name, sound)
    local self = setmetatable({}, Animal)  -- create instance with Animal as metatable
    self.name = name
    self.sound = sound
    return self
end

function Animal:speak()   -- colon syntax: self is implicit first argument
    print(self.name .. " says " .. self.sound .. "!")
end

function Animal:__tostring()   -- metamethod: called by tostring()
    return "Animal(" .. self.name .. ")"
end

local cat = Animal.new("Whiskers", "meow")
cat:speak()                     -- "Whiskers says meow!"
print(tostring(cat))            -- "Animal(Whiskers)"

-- Inheritance
local Dog = setmetatable({}, {__index = Animal})  -- Dog inherits from Animal
Dog.__index = Dog

function Dog.new(name)
    local self = Animal.new(name, "woof")   -- call parent constructor
    return setmetatable(self, Dog)           -- change metatable to Dog
end

function Dog:fetch(item)   -- Dog-specific method
    print(self.name .. " fetches the " .. item .. "!")
end

local rex = Dog.new("Rex")
rex:speak()         -- inherited from Animal: "Rex says woof!"
rex:fetch("ball")   -- "Rex fetches the ball!"

-- Metamethods (operator overloading)
local Vector = {}
Vector.__index = Vector

function Vector.new(x, y)
    return setmetatable({x = x, y = y}, Vector)
end

function Vector:__add(other)    -- overload +
    return Vector.new(self.x + other.x, self.y + other.y)
end

function Vector:__tostring()
    return string.format("Vector(%g, %g)", self.x, self.y)
end

function Vector:length()
    return math.sqrt(self.x^2 + self.y^2)
end

local v1 = Vector.new(3, 4)
local v2 = Vector.new(1, 2)
local v3 = v1 + v2          -- calls Vector.__add
print(tostring(v3))          -- "Vector(4, 6)"
print(v1:length())           -- 5.0
```

### Level 5: Coroutines — Lua's Cooperative Concurrency

```lua
-- Coroutines: cooperative multitasking within a single thread
-- A coroutine can yield (pause itself) and be resumed by another coroutine

-- Basic coroutine
local co = coroutine.create(function(a, b)
    print("start:", a, b)
    local c = coroutine.yield(a + b)   -- pause, return a+b, receive c when resumed
    print("resumed with:", c)
    return "done"
end)

-- Resume the coroutine (pass initial arguments)
local ok, value = coroutine.resume(co, 10, 20)
print("yielded:", value)   -- "yielded: 30"

-- Resume again (pass value to be received by yield)
ok, value = coroutine.resume(co, "hello")
-- "resumed with: hello"
print("returned:", value)  -- "returned: done"

-- Coroutine status
print(coroutine.status(co))  -- "dead" (finished)

-- Generator pattern using coroutines
local function range(from, to, step)
    step = step or 1
    return coroutine.wrap(function()
        for i = from, to, step do
            coroutine.yield(i)
        end
    end)
end

for n in range(1, 10, 2) do
    io.write(n .. " ")   -- 1 3 5 7 9
end
print()

-- Producer-consumer with coroutines
local function producer()
    local items = {"apple", "banana", "cherry"}
    for _, item in ipairs(items) do
        coroutine.yield(item)
    end
end

local gen = coroutine.wrap(producer)
for item in gen do   -- iterates until coroutine is dead
    print("Processing:", item)
end
```

### Level 6: Modules, Error Handling, and Embedding

```lua
-- Modules: each file is a module returning a table

-- mymodule.lua
local M = {}   -- private to this module

local function private_helper(x)
    return x * 2
end

function M.public_function(x)
    return private_helper(x) + 1
end

M.VERSION = "1.0"

return M   -- the module table is what require() returns

-- In another file:
-- local mod = require("mymodule")
-- print(mod.public_function(5))   -- 11
-- print(mod.VERSION)              -- "1.0"

-- Error handling
-- pcall: protected call, catches errors
local ok, result = pcall(function()
    error("something went wrong")
end)
print(ok, result)   -- false   "input:2: something went wrong"

local ok, result = pcall(function()
    return 42
end)
print(ok, result)   -- true    42

-- error() with a table (structured errors)
local function connect(host, port)
    if not host then
        error({code = "MISSING_HOST", message = "host is required"}, 2)
        -- second arg = level: 1=here, 2=caller, 0=no location info
    end
    -- ... connection logic
end

local ok, err = pcall(connect, nil, 8080)
if not ok then
    if type(err) == "table" then
        print("Error:", err.code, err.message)
    end
end

-- xpcall: like pcall but with a message handler (for stack traces)
local function traceback(err)
    return debug.traceback(err, 2)
end

local ok, result = xpcall(function()
    error("detailed error")
end, traceback)

-- Standard library modules
-- io: file I/O
local f = io.open("file.txt", "r")
if f then
    local content = f:read("*a")   -- read all
    f:close()
end

-- os: system operations
print(os.time())               -- Unix timestamp
print(os.date("%Y-%m-%d"))     -- formatted date
os.execute("ls -la")           -- run shell command

-- JSON-like serialization (simple, for debugging)
local function serialize(val, indent)
    indent = indent or ""
    local t = type(val)
    if t == "table" then
        local parts = {}
        for k, v in pairs(val) do
            local key = type(k) == "string" and k or tostring(k)
            parts[#parts+1] = indent .. "  " .. key .. " = " .. serialize(v, indent .. "  ")
        end
        return "{\n" .. table.concat(parts, ",\n") .. "\n" .. indent .. "}"
    elseif t == "string" then
        return string.format("%q", val)
    else
        return tostring(val)
    end
end
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Lua arrays are 1-indexed**

```lua
local t = {"a", "b", "c"}
print(t[0])   -- nil  (not "a"!)
print(t[1])   -- "a"
print(#t)     -- 3 (indices 1, 2, 3)

-- This trips up every programmer from C/Python/JavaScript
-- The # operator counts from 1. ipairs() starts at 1.
```

**Mistake 2: Forgetting `local` — accidental globals**

```lua
-- WRONG: i is a global — persists after the loop, may interfere
for i = 1, 10 do
    total = total + i   -- total is also global!
end

-- CORRECT: explicitly local
local total = 0
for i = 1, 10 do   -- loop variable i is implicitly local in numeric for
    total = total + i
end
```

**Mistake 3: `0` and `""` are truthy**

```lua
-- WRONG: expecting C-style truthiness
local count = 0
if count then   -- this IS true in Lua!
    print("count is truthy")   -- prints!
end

-- To check for zero: compare explicitly
if count == 0 then
    print("count is zero")
end

-- Only nil and false are falsy
if nil then ... end    -- false — never executes
if false then ... end  -- false — never executes
if 0 then ... end      -- TRUE — always executes!
```

**Mistake 4: `~=` not `!=` for inequality**

```lua
if x != y then   -- SyntaxError! This is not valid Lua
if x ~= y then   -- Correct: Lua uses ~= for "not equal"
if x == y then   -- equality is == (same as other languages)
```

**Mistake 5: Concatenating nil crashes with no clear message**

```lua
local name = nil
print("Hello " .. name)   -- error: attempt to concatenate a nil value

-- CORRECT: always ensure string before concatenating
print("Hello " .. (name or "guest"))   -- "Hello guest"
print("Hello " .. tostring(name))      -- "Hello nil"
```

---

## 5. The "Why Does This Work" Layer

### Why Tables Are Lua's Only Data Structure

The design decision to have a single compound data structure is intentional. Tables implement arrays (integer keys 1..n), dictionaries (string keys), and objects (tables with methods and metatables) in a single, orthogonal primitive. This simplicity makes the language easier to embed — the C API only needs to understand one type — and makes Lua's implementation small.

When you use a table as an array, Lua's implementation stores integer keys in a contiguous array part for O(1) indexed access. Non-integer keys go into a hash part. The # operator returns the boundary of the array part (last integer key before a nil gap), which is why tables with holes have undefined # behavior.

### How Metatables Enable Object-Oriented Programming

A metatable is a table attached to another table that controls how certain operations on that table behave. The `__index` metamethod is the most important: when you access `obj.field` and `field` is not in `obj`, Lua checks if `obj` has a metatable with `__index`. If `__index` is a table, Lua searches that table for `field`. If it's a function, Lua calls it.

Setting `MyClass.__index = MyClass` means: "when looking up a field on an instance of MyClass, search MyClass's own table." This is prototype inheritance — instances delegate unknown field lookups to their class table.

---

## 6. Quick Reference

### Types and Operations

| Type | Literal | `type()` | Falsy? |
|------|---------|---------|--------|
| nil | `nil` | "nil" | Yes |
| boolean | `true`, `false` | "boolean" | false only |
| number | `42`, `3.14`, `0xFF` | "number" | No (0 is truthy!) |
| string | `"hi"`, `'hi'`, `[[hi]]` | "string" | No |
| table | `{}` | "table" | No |
| function | `function() end` | "function" | No |

### String Patterns

```lua
%a = letter    %A = non-letter
%d = digit     %D = non-digit
%l = lowercase %u = uppercase
%s = space     %S = non-space
%w = alphanum  %p = punctuation
%. = any char  %% = literal %

Quantifiers: * (0+) + (1+) - (lazy 0+) ? (0 or 1)
Anchors: ^ (start) $ (end)
Captures: (pattern)

-- Examples
("abc123"):match("(%a+)(%d+)")   -- "abc", "123"
("2024-01-15"):match("(%d+)-(%d+)-(%d+)")  -- "2024", "01", "15"
```

### Metatables Quick Reference

| Metamethod | Triggered By |
|-----------|-------------|
| `__index` | `t.key` when key not in t |
| `__newindex` | `t.key = v` when key not in t |
| `__add` | `+` operator |
| `__sub` | `-` operator |
| `__mul` | `*` operator |
| `__div` | `/` operator |
| `__eq` | `==` operator |
| `__lt` | `<` operator |
| `__len` | `#` operator |
| `__tostring` | `tostring()` |
| `__call` | Calling the table as a function |
