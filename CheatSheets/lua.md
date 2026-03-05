# Lua Cheat Sheet
### From Zero to Expert

---

## What is Lua?

**Lua** is a lightweight, fast, embeddable scripting language designed for simplicity and extensibility. It is written in clean ANSI C, runs on virtually every platform, and is the go-to scripting language for game engines (Roblox, LÖVE, World of Warcraft), embedded systems, and as a config/extension layer in applications like Neovim, Redis, and nginx.

```lua
-- Hello, World!
print("Hello, World!")
```

**Key traits:** dynamically typed · garbage collected · 1-indexed arrays · first-class functions · prototype-based OOP via metatables · tiny runtime (~200 KB)

---

## Running Lua

```bash
lua script.lua          # run a file
lua -e "print('hi')"   # run inline expression
lua                     # interactive REPL
luac -o out.luac in.lua # compile to bytecode
```

---

## Variables & Types

Lua has **8 types**. Variables are untyped — types belong to values, not variables.

| Type | Description | Example |
|---|---|---|
| `nil` | Absence of value | `x = nil` |
| `boolean` | `true` or `false` | `x = true` |
| `number` | 64-bit float by default (int subtype in 5.3+) | `x = 42` |
| `string` | Immutable byte sequence | `x = "hello"` |
| `table` | The only data structure | `x = {}` |
| `function` | First-class value | `x = print` |
| `userdata` | C data (host application) | *(C API)* |
| `thread` | Coroutine | `x = coroutine.create(f)` |

```lua
type(42)          --> "number"
type("hi")        --> "string"
type(nil)         --> "nil"
type({})          --> "table"
type(print)       --> "function"
type(true)        --> "boolean"
```

### Variable Scope

```lua
x = 10            -- global (accessible everywhere)
local y = 20      -- local (block-scoped — always prefer this)

do
  local z = 30    -- z only exists inside this block
  print(z)        -- 30
end
-- print(z)       -- ERROR: z is nil here

-- Rule: always use `local` unless you explicitly need a global
```

---

## Numbers

```lua
-- Integer vs float (Lua 5.3+):
local i = 42          -- integer subtype
local f = 42.0        -- float subtype
local f2 = 3.14e2     -- scientific notation → 314.0
local h = 0xFF        -- hex literal → 255
local b = 0b1010      -- binary (Lua 5.4+) → 10

math.type(1)          --> "integer"
math.type(1.0)        --> "float"

-- Arithmetic:
10 + 3   --> 13
10 - 3   --> 7
10 * 3   --> 30
10 / 3   --> 3.3333...  (always float division)
10 // 3  --> 3          (floor division)
10 % 3   --> 1          (modulo)
2 ^ 10   --> 1024.0     (power, always float)

-- Math library:
math.abs(-5)        --> 5
math.ceil(4.2)      --> 5
math.floor(4.9)     --> 4
math.sqrt(16)       --> 4.0
math.max(1, 5, 3)   --> 5
math.min(1, 5, 3)   --> 1
math.random()       --> float in [0,1)
math.random(6)      --> integer in [1,6]
math.random(3, 9)   --> integer in [3,9]
math.randomseed(os.time())  -- seed the RNG
math.pi             --> 3.1415926535898
math.huge           --> inf
math.maxinteger     --> 9223372036854775807

-- Integer / float conversions:
math.tointeger(5.0)  --> 5  (nil if not representable)
(1.5):__tointeger()  -- N/A; use math.tointeger
```

---

## Strings

Strings in Lua are **immutable** and **interned** — operations return new strings.

```lua
local s = "Hello, Lua!"
local s2 = 'single quotes work too'
local s3 = [[
  multi-line
  string literal
  (no escape sequences processed)
]]

-- Concatenation (.. operator):
"Hello" .. ", " .. "World"   --> "Hello, World!"
-- Note: .. is NOT +. Using + on strings causes a runtime error.

-- Length operator:
#"hello"    --> 5
#s          --> 11

-- String library:
string.len(s)               --> 11
string.upper(s)             --> "HELLO, LUA!"
string.lower(s)             --> "hello, lua!"
string.rep("ab", 3)         --> "ababab"
string.rep("ab", 3, ",")    --> "ab,ab,ab"
string.reverse(s)           --> "!auL ,olleH"
string.sub(s, 1, 5)         --> "Hello"   (1-indexed, inclusive)
string.sub(s, -4)           --> "Lua!"    (negative = from end)
string.byte("A")            --> 65
string.char(65)             --> "A"
string.format("Pi = %.2f", math.pi)  --> "Pi = 3.14"

-- Method syntax (colon notation):
s:upper()           --> "HELLO, LUA!"
s:sub(1, 5)         --> "Hello"
s:find("Lua")       --> 8  9  (start, end positions)
s:find("xyz")       --> nil
s:match("(%a+)")    --> "Hello"   (first word)
s:gsub("l", "L")    --> "HeLLo, Lua!"  2
s:gmatch("%a+")     -- iterator over all words

-- String format specifiers:
-- %d  integer      %f  float       %s  string
-- %x  hex          %q  quoted      %g  shorter of %e/%f
-- %05d → zero-padded   %-10s → left-aligned

-- Escape sequences:
-- \n newline  \t tab  \r carriage return
-- \\ backslash  \" double quote  \' single quote
-- \97 decimal escape  \x61 hex escape  \u{1F600} Unicode (5.3+)
```

### String Pattern Matching

Lua uses its own pattern language (not full regex, but powerful):

```lua
-- Pattern classes:
-- %a  letters       %A  non-letters
-- %d  digits        %D  non-digits
-- %l  lowercase     %u  uppercase
-- %s  whitespace    %S  non-whitespace
-- %w  alphanumeric  %p  punctuation
-- %c  control chars %.  any character

-- Quantifiers:
-- *   0 or more (greedy)
-- +   1 or more (greedy)
-- -   0 or more (lazy)
-- ?   0 or 1

-- Anchors:
-- ^   start of string
-- $   end of string

-- Examples:
("hello123"):match("(%a+)(%d+)")  --> "hello"  "123"
("2025-03-06"):match("(%d+)-(%d+)-(%d+)")  --> "2025" "03" "06"

for word in ("one two three"):gmatch("%a+") do
  print(word)         -- one, two, three (separate prints)
end

("hello world"):gsub("%a+", function(w) return w:upper() end)
--> "HELLO WORLD"  2
```

---

## Booleans & Operators

```lua
-- Boolean values: only false and nil are falsy.
-- Everything else (0, "", {}, functions) is TRUTHY.

-- Logical operators (short-circuit, return one of their operands):
true  and "yes"   --> "yes"
false and "yes"   --> false
nil   and "yes"   --> nil

true  or "no"     --> true
false or "no"     --> "no"
nil   or "default"--> "default"

not true          --> false
not nil           --> true
not 0             --> false  (0 is truthy in Lua!)

-- Idiomatic patterns:
local x = value or default_value     -- provide default
local y = condition and a or b       -- ternary (only safe when a is truthy)

-- Comparison operators:
==   ~=   <   >   <=   >=
-- ~= is "not equal" (not != like other languages)
-- == never coerces types: 1 == "1" is false
```

---

## Control Flow

```lua
-- if / elseif / else:
local score = 85

if score >= 90 then
  print("A")
elseif score >= 80 then
  print("B")
elseif score >= 70 then
  print("C")
else
  print("F")
end

-- while loop:
local i = 1
while i <= 5 do
  print(i)
  i = i + 1
end

-- repeat...until (condition checked AFTER body — runs at least once):
local n = 0
repeat
  n = n + 1
until n >= 5

-- Numeric for loop:
for i = 1, 10 do        -- 1 to 10 inclusive, step 1
  print(i)
end

for i = 10, 1, -2 do    -- 10, 8, 6, 4, 2 (step -2)
  print(i)
end

-- Generic for loop (iterators):
local fruits = {"apple", "banana", "cherry"}

for i, v in ipairs(fruits) do   -- ordered, stops at first nil
  print(i, v)                   -- 1 apple, 2 banana, 3 cherry
end

local person = {name = "Alice", age = 30}

for k, v in pairs(person) do    -- all key-value pairs (unordered)
  print(k, v)
end

-- break (exit loop), goto (Lua 5.2+):
for i = 1, 100 do
  if i > 5 then break end
  print(i)
end

-- goto (useful to skip to end of block):
for i = 1, 10 do
  if i % 2 == 0 then goto continue end
  print(i)                       -- prints odd numbers only
  ::continue::
end

-- Lua has NO switch/case, NO continue keyword (use goto ::continue::)
-- Lua has NO ++ or += operators (use i = i + 1)
```

---

## Functions

Functions are **first-class values** in Lua — they can be stored in variables, passed as arguments, and returned from other functions.

```lua
-- Basic definition:
local function greet(name)
  return "Hello, " .. name
end

-- Equivalent (function stored in variable):
local greet = function(name)
  return "Hello, " .. name
end

print(greet("Lua"))   --> Hello, Lua

-- Multiple return values:
local function minmax(a, b)
  return math.min(a, b), math.max(a, b)
end

local lo, hi = minmax(3, 7)   -- lo=3, hi=7
local lo2     = minmax(3, 7)  -- lo2=3 (extra returns discarded)

-- Variadic functions (...):
local function sum(...)
  local args = {...}           -- pack into table
  local total = 0
  for _, v in ipairs(args) do
    total = total + v
  end
  return total
end

sum(1, 2, 3, 4)   --> 10

-- select with varargs:
local function info(...)
  print(select('#', ...))     -- number of args
  print(select(2, ...))       -- args from index 2 onward
end

-- Named arguments via table:
local function createWindow(opts)
  opts = opts or {}
  local w = opts.width  or 800
  local h = opts.height or 600
  local t = opts.title  or "Window"
  print(w, h, t)
end

createWindow({ width = 1024, title = "My App" })

-- Closures (function captures its enclosing scope):
local function makeCounter(start)
  local count = start or 0
  return function()
    count = count + 1
    return count
  end
end

local counter = makeCounter(10)
counter()   --> 11
counter()   --> 12
counter()   --> 13

-- Tail calls (Lua optimizes these — no stack growth):
local function fact(n, acc)
  acc = acc or 1
  if n <= 1 then return acc end
  return fact(n - 1, n * acc)   -- tail call
end
```

---

## Tables

Tables are Lua's **only** compound data structure. They serve as arrays, dictionaries, objects, namespaces, and modules — all at once.

```lua
-- Empty table:
local t = {}

-- Array-style (1-indexed by convention):
local arr = {10, 20, 30, 40, 50}
arr[1]          --> 10
arr[#arr]       --> 50   (last element)
arr[6] = 60     -- append

-- Dictionary-style:
local person = {
  name = "Alice",
  age  = 30,
  ["favorite-color"] = "blue",   -- bracket syntax for non-identifier keys
}
person.name              --> "Alice"
person["favorite-color"] --> "blue"
person.job = "Engineer"  -- add new key
person.age = nil         -- remove key (set to nil)

-- Mixed (array + dict):
local mixed = {
  "first",           -- [1]
  "second",          -- [2]
  label = "hello",   -- string key
}

-- Table manipulation:
table.insert(arr, 99)        -- append 99
table.insert(arr, 2, 15)     -- insert 15 at index 2
table.remove(arr, 2)         -- remove element at index 2
table.remove(arr)            -- remove last element
table.sort(arr)              -- sort in-place (ascending)
table.sort(arr, function(a, b) return a > b end)  -- custom sort (descending)
table.concat(arr, ", ")      -- join array elements: "10, 20, 30"
table.move(src, 1, #src, 1, dst)  -- copy src into dst (Lua 5.3+)

-- Unpacking:
local a, b, c = table.unpack({10, 20, 30})

-- Iterating:
-- ipairs → array portion (1..n, stops at first nil, ordered)
for i, v in ipairs(arr) do print(i, v) end

-- pairs → all keys (including string keys, unordered)
for k, v in pairs(person) do print(k, v) end

-- next() → low-level iteration:
local k, v = next(t)         -- first key-value pair (or nil if empty)

-- Table as set:
local set = {}
set["apple"]  = true
set["banana"] = true
if set["apple"] then print("in set") end

-- Nested tables:
local matrix = {
  {1, 2, 3},
  {4, 5, 6},
  {7, 8, 9},
}
matrix[2][3]   --> 6
```

---

## Metatables & Metamethods

Metatables give tables (and userdata) **operator overloading** and custom behavior. They are the foundation of OOP in Lua.

```lua
local mt = {}

-- Arithmetic metamethods:
mt.__add = function(a, b) ... end   -- a + b
mt.__sub = function(a, b) ... end   -- a - b
mt.__mul = function(a, b) ... end   -- a * b
mt.__div = function(a, b) ... end   -- a / b
mt.__mod = function(a, b) ... end   -- a % b
mt.__pow = function(a, b) ... end   -- a ^ b
mt.__unm = function(a)    ... end   -- -a  (unary minus)
mt.__idiv= function(a, b) ... end   -- a // b

-- Comparison metamethods:
mt.__eq = function(a, b) ... end    -- a == b
mt.__lt = function(a, b) ... end    -- a < b
mt.__le = function(a, b) ... end    -- a <= b

-- String metamethod:
mt.__tostring = function(t) return "MyTable" end   -- tostring(t)

-- Length metamethod:
mt.__len = function(t) return 42 end   -- #t

-- Concatenation:
mt.__concat = function(a, b) ... end   -- a .. b

-- Call metamethod (make table callable):
mt.__call = function(t, ...) ... end   -- t(args)

-- Index metamethods (most important for OOP):
mt.__index = function(t, k) ... end    -- called when t[k] is nil
mt.__newindex = function(t, k, v) ... end  -- called on t[k] = v

-- Attach metatable:
local obj = setmetatable({}, mt)
getmetatable(obj)   --> mt

-- __index shortcut — inherit from another table:
mt.__index = ParentTable   -- if key not in obj, look in ParentTable
```

### Practical Vector Example

```lua
local Vector = {}
Vector.__index = Vector

function Vector.new(x, y)
  return setmetatable({x = x, y = y}, Vector)
end

function Vector:__tostring()
  return string.format("(%g, %g)", self.x, self.y)
end

function Vector.__add(a, b)
  return Vector.new(a.x + b.x, a.y + b.y)
end

function Vector:length()
  return math.sqrt(self.x^2 + self.y^2)
end

local v1 = Vector.new(3, 4)
local v2 = Vector.new(1, 2)
local v3 = v1 + v2

print(tostring(v1))    --> (3, 4)
print(v1:length())     --> 5.0
print(tostring(v3))    --> (4, 6)
```

---

## Object-Oriented Programming

Lua has no built-in class system. OOP is implemented via **tables + metatables**.

```lua
-- Class definition pattern:
local Animal = {}
Animal.__index = Animal

-- Constructor:
function Animal.new(name, sound)
  local self = setmetatable({}, Animal)
  self.name  = name
  self.sound = sound
  return self
end

-- Methods (colon syntax sugar: self is implicit first arg):
function Animal:speak()
  print(self.name .. " says " .. self.sound)
end

function Animal:getName()
  return self.name
end

-- Inheritance:
local Dog = setmetatable({}, {__index = Animal})
Dog.__index = Dog

function Dog.new(name)
  local self = Animal.new(name, "Woof")
  return setmetatable(self, Dog)
end

function Dog:fetch(item)
  print(self.name .. " fetches the " .. item)
end

-- Usage:
local cat = Animal.new("Cat", "Meow")
cat:speak()              --> Cat says Meow

local dog = Dog.new("Rex")
dog:speak()              --> Rex says Woof  (inherited)
dog:fetch("ball")        --> Rex fetches the ball

-- instanceof check:
local function instanceof(obj, class)
  local mt = getmetatable(obj)
  while mt do
    if mt == class then return true end
    mt = getmetatable(mt)
  end
  return false
end
```

---

## Modules

The standard way to organize Lua code into reusable units.

```lua
-- mymodule.lua
local M = {}   -- module table (only this is returned/exported)

local private_var = "secret"   -- not exported

function M.hello(name)
  return "Hello, " .. name
end

function M.add(a, b)
  return a + b
end

return M   -- MUST return the module table

-- ─────────────────────────────────────

-- main.lua
local mymod = require("mymodule")   -- searches package.path

mymod.hello("Lua")    --> "Hello, Lua"
mymod.add(2, 3)       --> 5

-- require caches results — calling require("mymodule") again
-- returns the SAME table (not re-executed)

-- package.path controls where require searches:
print(package.path)
-- add a custom path:
package.path = package.path .. ";./libs/?.lua"

-- package.loaded — the require cache:
package.loaded["mymodule"] = nil   -- force re-load next require
```

---

## Error Handling

```lua
-- error() raises an error:
error("something went wrong")          -- message
error("oops", 2)                       -- level 2 = blame caller
error({code = 404, msg = "Not found"}) -- table as error object

-- pcall (protected call) — catches errors:
local ok, result = pcall(function()
  return 1 / 0    -- no error (Lua returns inf)
end)
-- ok = true, result = inf

local ok, err = pcall(function()
  error("boom")
end)
-- ok = false, err = "input:2: boom"

-- xpcall — like pcall but with a message handler:
local function handler(err)
  return debug.traceback(err, 2)   -- add stack trace
end

local ok, result = xpcall(riskyFunction, handler)

-- Error handling pattern:
local function safeDivide(a, b)
  if b == 0 then
    return nil, "division by zero"   -- return nil + error message
  end
  return a / b
end

local result, err = safeDivide(10, 0)
if not result then
  print("Error:", err)
end

-- assert — shorthand for error-if-nil:
local f = assert(io.open("file.txt", "r"), "Cannot open file")
-- if io.open returns nil, assert raises the error message
```

---

## Coroutines

Lua coroutines are **cooperative multitasking** — they yield control back to the caller rather than running concurrently. They are the building block for async patterns, generators, and state machines.

```lua
-- Create a coroutine:
local co = coroutine.create(function(a, b)
  print("start:", a, b)
  local c = coroutine.yield(a + b)   -- pause, return a+b to caller
  print("resumed with:", c)
  return "done"
end)

-- Resume (start or continue execution):
local ok, val = coroutine.resume(co, 10, 20)
-- prints "start: 10 20"
-- ok=true, val=30 (the yielded value)

local ok, val = coroutine.resume(co, 99)
-- prints "resumed with: 99"
-- ok=true, val="done" (the return value)

local ok, val = coroutine.resume(co)
-- ok=false, val="cannot resume dead coroutine"

-- Coroutine status:
coroutine.status(co)   --> "dead" / "suspended" / "running" / "normal"

-- coroutine.wrap — simpler interface (raises on error instead of returning false):
local gen = coroutine.wrap(function()
  for i = 1, 3 do
    coroutine.yield(i)
  end
end)

gen()   --> 1
gen()   --> 2
gen()   --> 3
gen()   --> error: cannot resume dead coroutine

-- Generator pattern:
local function range(from, to, step)
  step = step or 1
  return coroutine.wrap(function()
    for i = from, to, step do
      coroutine.yield(i)
    end
  end)
end

for v in range(1, 5) do
  print(v)   -- 1, 2, 3, 4, 5
end

-- Producer-consumer pattern:
local function producer()
  local items = {"a", "b", "c"}
  for _, item in ipairs(items) do
    coroutine.yield(item)
  end
end

local co = coroutine.create(producer)
while true do
  local ok, item = coroutine.resume(co)
  if not ok or item == nil then break end
  print("consumed:", item)
end
```

---

## The Standard Library

### io — File I/O

```lua
-- Write to file:
local f = io.open("output.txt", "w")
f:write("Hello\n")
f:write(string.format("Pi = %.4f\n", math.pi))
f:close()

-- Read entire file:
local f = io.open("output.txt", "r")
local content = f:read("*a")   -- "*a" or "a" = all
f:close()

-- Read line by line:
for line in io.lines("output.txt") do
  print(line)
end

-- File modes: "r" read  "w" write  "a" append  "rb" binary read
-- f:read("*l") or "l" → next line (default)
-- f:read("*n") or "n" → next number
-- f:read("*a") or "a" → entire file
-- f:seek("set", 0)    → seek to beginning

-- Standard streams:
io.write("no newline")
io.read()             -- read line from stdin
print(...)            -- writes to stdout with newline
```

### os — Operating System

```lua
os.time()                   -- Unix timestamp (seconds)
os.clock()                  -- CPU time used by program
os.date("%Y-%m-%d %H:%M:%S") -- formatted date string
os.date("*t")               -- table: {year,month,day,hour,min,sec,wday,yday}
os.difftime(t2, t1)         -- difference in seconds
os.exit(0)                  -- exit with code
os.exit(true)               -- exit with success (code 0)
os.getenv("HOME")           -- environment variable
os.execute("ls -la")        -- run shell command (returns true/nil,reason,code)
os.rename("old.txt","new.txt")
os.remove("file.txt")
os.tmpname()                -- temporary filename
```

### table — Table Utilities

```lua
table.insert(t, val)        -- append
table.insert(t, pos, val)   -- insert at position
table.remove(t, pos)        -- remove at position (returns removed value)
table.remove(t)             -- remove last
table.sort(t)               -- sort in-place
table.sort(t, comparator)   -- custom sort
table.concat(t, sep)        -- join to string
table.concat(t, sep, i, j)  -- join slice [i,j]
table.unpack(t)             -- unpack to multiple values
table.unpack(t, i, j)       -- unpack slice
table.move(a1, f, e, t, a2) -- move elements (Lua 5.3+)
```

### math — Mathematics

```lua
math.pi         math.huge       math.maxinteger  math.mininteger
math.abs(x)     math.ceil(x)    math.floor(x)    math.sqrt(x)
math.sin(x)     math.cos(x)     math.tan(x)
math.asin(x)    math.acos(x)    math.atan(y, x)  -- atan2 equivalent
math.exp(x)     math.log(x)     math.log(x, base)
math.max(...)   math.min(...)
math.fmod(x,y)  math.modf(x)    -- modf returns int and frac parts
math.random()   math.random(n)  math.random(m, n)
math.randomseed(seed)
math.tointeger(x)               -- float → int (or nil)
math.type(x)                    -- "integer", "float", or fail
```

### string — String Utilities

```lua
string.format(fmt, ...)
string.len(s)          string.sub(s, i, j)
string.upper(s)        string.lower(s)
string.rep(s, n, sep)  string.reverse(s)
string.byte(s, i)      string.char(...)
string.find(s, pat, init, plain)
string.match(s, pat, init)
string.gmatch(s, pat)
string.gsub(s, pat, repl, n)
string.dump(func)      -- serialize function to binary string
```

---

## Iterators

Custom iterators integrate cleanly with the generic `for` loop.

```lua
-- Stateful iterator (closure-based):
local function values(t)
  local i = 0
  return function()
    i = i + 1
    return t[i]
  end
end

for v in values({"a", "b", "c"}) do
  print(v)    -- a, b, c
end

-- Stateless iterator (ipairs style):
local function myipairs_iter(t, i)
  i = i + 1
  local v = t[i]
  if v ~= nil then return i, v end
end

local function myipairs(t)
  return myipairs_iter, t, 0
end

-- Filtered iterator:
local function filter(pred, iter, state, init)
  return function(s, c)
    local k, v
    repeat k, v = iter(s, c); c = k until k == nil or pred(v)
    return k, v
  end, state, init
end
```

---

## Advanced Patterns

### Memoization

```lua
local function memoize(f)
  local cache = {}
  return function(n)
    if cache[n] == nil then
      cache[n] = f(n)
    end
    return cache[n]
  end
end

local fib = memoize(function(n)
  if n < 2 then return n end
  return fib(n-1) + fib(n-2)   -- fib here is the memoized version
end)
-- Note: for mutual recursion, define fib as local first, then assign.
```

### Functional Patterns

```lua
-- map:
local function map(t, f)
  local result = {}
  for i, v in ipairs(t) do result[i] = f(v) end
  return result
end

-- filter:
local function filter(t, pred)
  local result = {}
  for _, v in ipairs(t) do
    if pred(v) then result[#result+1] = v end
  end
  return result
end

-- reduce:
local function reduce(t, f, init)
  local acc = init
  for _, v in ipairs(t) do acc = f(acc, v) end
  return acc
end

local nums = {1, 2, 3, 4, 5}
map(nums, function(x) return x * 2 end)        -- {2,4,6,8,10}
filter(nums, function(x) return x % 2 == 0 end)-- {2,4}
reduce(nums, function(a,b) return a+b end, 0)  -- 15
```

### Sandboxing

```lua
-- Run untrusted code in a restricted environment:
local sandbox_env = {
  print = print,
  math  = math,
  -- only expose safe functions
}

local code = [[
  print(math.sqrt(16))
]]

local fn, err = load(code, "sandbox", "t", sandbox_env)
if fn then
  fn()    --> 4.0
else
  print("Syntax error:", err)
end
```

### Weak Tables (Garbage Collection Hints)

```lua
-- Weak keys: entry removed when key is GC'd
local weak_keys = setmetatable({}, {__mode = "k"})

-- Weak values: entry removed when value is GC'd
local weak_vals = setmetatable({}, {__mode = "v"})

-- Both weak:
local cache = setmetatable({}, {__mode = "kv"})

-- Use case: object caches, canonicalization tables
-- where you don't want the cache to prevent GC
```

---

## The `debug` Library

```lua
-- Use sparingly — bypasses Lua's safety guarantees
debug.traceback()           -- current stack trace as string
debug.traceback(msg, level) -- with message and level
debug.getinfo(f)            -- info about function f
debug.getinfo(1)            -- info about current function
debug.getlocal(level, idx)  -- get local variable
debug.setlocal(level, idx, val) -- set local variable
debug.getupvalue(f, idx)    -- get upvalue of closure
debug.setupvalue(f, idx, v) -- set upvalue
debug.sethook(hook, mask, count) -- set debug hook
-- mask: "c" call, "r" return, "l" line
```

---

## Lua 5.4 Key Additions

```lua
-- Integer subtype (was already in 5.3, consolidated in 5.4)
-- Generalized for loop with to-be-closed variables:
for i, f <close> in stateful_iter() do ... end
-- f:close() called automatically on loop exit (like RAII)

-- <close> attribute (to-be-closed variables):
local f <close> = io.open("file.txt")
-- f:close() called automatically when f goes out of scope

-- <const> attribute:
local x <const> = 42   -- x cannot be reassigned (compile-time constant)

-- New warn() function:
warn("something suspicious happened")

-- Updated math library (math.tointeger improvements)
-- Bitwise operators (since 5.3):
a & b    -- AND
a | b    -- OR
a ~ b    -- XOR (also unary bitwise NOT: ~a)
a << n   -- left shift
a >> n   -- right shift
```

---

## Bitwise Operations (Lua 5.3+)

```lua
-- Only work on integers:
0xFF & 0x0F     --> 15     (AND)
0xF0 | 0x0F     --> 255    (OR)
0xFF ~ 0x0F     --> 240    (XOR)
~0              --> -1     (bitwise NOT, all bits set)
1 << 4          --> 16     (left shift)
256 >> 4        --> 16     (right shift)

-- Common idioms:
n & 1           -- check if n is odd
n & (n-1)       -- clear lowest set bit
n | (1 << k)    -- set bit k
n & ~(1 << k)   -- clear bit k
n ~ (1 << k)    -- toggle bit k
(n >> k) & 1    -- test bit k
```

---

## Performance Tips

```lua
-- 1. Always use local variables — local access is faster than global
local sin = math.sin        -- cache frequently used functions locally
local floor = math.floor

-- 2. Pre-allocate tables when size is known:
local t = {}
for i = 1, 1000 do t[i] = i end   -- faster than table.insert in loop

-- 3. Use table.concat for string building (never .. in a loop):
-- SLOW:
local s = ""
for i = 1, 1000 do s = s .. tostring(i) end

-- FAST:
local parts = {}
for i = 1, 1000 do parts[i] = tostring(i) end
local s = table.concat(parts)

-- 4. Avoid global lookups in tight loops:
-- SLOW:
for i = 1, 1e6 do math.sin(i) end

-- FAST:
local sin = math.sin
for i = 1, 1e6 do sin(i) end

-- 5. Reuse tables instead of creating new ones in hot paths

-- 6. Use LuaJIT for 10–100× speedup on numeric/loop-heavy code

-- 7. Prefer ipairs over pairs for array iteration (faster)

-- 8. Avoid unnecessary function calls in inner loops
```

---

## Quick Reference Card

```
TYPES           nil  boolean  number  string  table  function  userdata  thread
FALSY VALUES    nil  false   (everything else is truthy, including 0 and "")

OPERATORS
  Arithmetic    +  -  *  /  //  %  ^  (no ++ or +=)
  Comparison    ==  ~=  <  >  <=  >=
  Logical       and  or  not   (short-circuit; return operand values)
  String        ..  (concat)   #  (length)
  Bitwise       &  |  ~  <<  >>  (integers only, Lua 5.3+)

SCOPE           local → block-scoped   global → _G table (avoid)

TABLES
  Array         t = {10,20,30}          t[1] → 10 (1-indexed!)
  Dict          t = {key="val"}         t.key / t["key"]
  Length        #t → array length (unreliable with holes)
  Iteration     ipairs → ordered array   pairs → all keys

FUNCTIONS
  Multireturn   return a, b, c
  Varargs       function f(...) local t={...} end
  Methods       obj:method() == obj.method(obj)
  Closures      capture upvalues from enclosing scope

OOP PATTERN
  local Class = {}; Class.__index = Class
  function Class.new(...) return setmetatable({}, Class) end
  function Class:method() ... end   -- self is implicit

ERROR HANDLING
  error(msg)            raise error
  pcall(f, ...)         protected call → ok, result
  xpcall(f, handler)    protected call with traceback handler
  assert(v, msg)        raise if v is falsy

COROUTINES
  coroutine.create(f)   coroutine.resume(co, ...)
  coroutine.yield(...)  coroutine.wrap(f)
  coroutine.status(co)  → "suspended" "running" "dead" "normal"

COMMON GOTCHAS
  Indexing starts at 1, not 0
  0 and "" are TRUTHY (only nil and false are falsy)
  ~= is not-equal (not != or !==)
  No ++ / += / -= operators
  .. is concat, not +
  pairs order is undefined; use ipairs for ordered iteration
  # is unreliable on tables with nil holes
  require caches — mutate package.loaded to reload
```
