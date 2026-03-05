# JavaScript Reference Guide

> A comprehensive reference for JavaScript — from foundational concepts to advanced patterns.
> ECMAScript 2024 | Compatible with all modern browsers and Node.js 18+

---

## What is JavaScript?

JavaScript is a **dynamic, interpreted, single-threaded programming language** originally designed to make web pages interactive. Today it runs across browsers, servers (Node.js), mobile apps, desktop apps, and embedded systems.

**Core characteristics:**

- **Dynamically typed** — variables can hold any type; types are resolved at runtime.
- **Interpreted** — executed line by line via a JavaScript engine (e.g., V8 in Chrome/Node.js).
- **Prototype-based** — objects inherit directly from other objects.
- **First-class functions** — functions are values; they can be stored, passed, and returned.
- **Single-threaded with an event loop** — one operation runs at a time, but asynchronous tasks are handled non-blockingly.

---

## Running JavaScript

```javascript
// In a browser: open DevTools > Console (F12 or Cmd+Option+J)

// Embedded in an HTML file:
<script src="app.js"></script>
<script>
  console.log("Hello, world!");
</script>

// In Node.js (terminal):
node app.js
node -e "console.log('Hello')"   // inline execution

// Console output methods:
console.log("Hello");             // standard output
console.error("Oops");            // error output (styled red)
console.warn("Watch out");        // warning output (styled yellow)
console.table([{ a: 1 }, { a: 2 }]); // render array/object as table
console.time("label");            // start a named timer
console.timeEnd("label");         // stop timer and print elapsed time
```

---

## Variables

```javascript
var name = "Alice";   // Legacy — function-scoped, hoisted. Avoid in modern code.
let age = 30;         // Block-scoped, reassignable.
const PI = 3.14159;   // Block-scoped, cannot be reassigned. Prefer this by default.

// Rule of thumb: use const everywhere. Switch to let only when reassignment is needed.

// Multiple declarations on one line:
let x = 1, y = 2, z = 3;

// Declaring without a value:
let score;            // value is undefined

// const with objects and arrays — the binding is constant, not the contents:
const user = { name: "Alice" };
user.name = "Bob";    // Allowed — you are mutating the object, not the binding.
user = {};            // TypeError — you cannot reassign the binding itself.
```

### Hoisting

Hoisting is JavaScript's behavior of moving declarations to the top of their scope before execution.

```javascript
// var declarations are hoisted and initialized to undefined:
console.log(x);  // undefined (no error)
var x = 5;

// let and const are hoisted but not initialized (Temporal Dead Zone):
console.log(y);  // ReferenceError
let y = 5;

// Function declarations are fully hoisted:
greet();         // Works — the entire function is available before its declaration.
function greet() { console.log("Hi"); }
```

---

## Data Types

JavaScript has **8 data types**: 7 primitives and 1 object type.

```javascript
// Primitives (immutable, compared by value):
let str   = "hello";            // String
let num   = 42;                 // Number (covers both integers and floats)
let big   = 9007199254740991n;  // BigInt (for integers beyond Number.MAX_SAFE_INTEGER)
let bool  = true;               // Boolean
let empty = null;               // Null (intentional absence of a value)
let undef = undefined;          // Undefined (variable declared but not assigned)
let sym   = Symbol("id");       // Symbol (guaranteed unique identifier)

// Object type (mutable, compared by reference):
let obj   = { key: "value" };   // Object
let arr   = [1, 2, 3];          // Array (a specialized object)
let fn    = function() {};       // Function (a specialized object)

// Checking types:
typeof "hello"       // "string"
typeof 42            // "number"
typeof true          // "boolean"
typeof undefined     // "undefined"
typeof null          // "object"  — historical bug; null is not actually an object
typeof {}            // "object"
typeof []            // "object"  — arrays are objects
typeof function(){}  // "function"

// More precise checks:
Array.isArray([])                       // true
null === null                           // true
value instanceof Date                   // true / false
Object.prototype.toString.call(value)   // "[object Array]", "[object Date]", etc.
```

---

## Strings

```javascript
let s = "Hello, World!";

// Template literals — preferred for interpolation and multi-line strings:
let name = "Alice";
let greeting = `Hello, ${name}! 2 + 2 = ${2 + 2}`;
let multiline = `Line one
Line two`;

// Common string methods:
s.length                     // 13
s.toUpperCase()              // "HELLO, WORLD!"
s.toLowerCase()              // "hello, world!"
s.trim()                     // Remove whitespace from both ends.
s.trimStart()                // Remove leading whitespace.
s.trimEnd()                  // Remove trailing whitespace.
s.includes("World")          // true
s.startsWith("Hello")        // true
s.endsWith("!")              // true
s.indexOf("o")               // 4 — first index of match, or -1
s.lastIndexOf("o")           // 8
s.slice(0, 5)                // "Hello" — end index is exclusive
s.slice(-6)                  // "World!" — negative index counts from the end
s.substring(7, 12)           // "World"
s.replace("World", "JS")     // "Hello, JS!" — replaces first match
s.replaceAll("l", "r")       // "Herro, Worrd!" — replaces all matches
s.split(", ")                // ["Hello", "World!"]
s.split("")                  // ["H", "e", "l", "l", "o", ...]
s.repeat(3)                  // "Hello, World!Hello, World!Hello, World!"
s.padStart(15, "*")          // "**Hello, World!"
s.padEnd(15, "-")            // "Hello, World!--"
s.charAt(0)                  // "H"
s[0]                         // "H" — bracket notation
s.charCodeAt(0)              // 72 — Unicode code point
String.fromCharCode(72)      // "H"

// Type conversions:
String(42)                   // "42"
(42).toString()              // "42"
(255).toString(16)           // "ff" — hexadecimal
parseInt("42px")             // 42 — parses leading number, ignores the rest
parseFloat("3.14abc")        // 3.14
Number("42")                 // 42
```

---

## Numbers & Math

```javascript
// Number literals:
let n   = 42;
let f   = 3.14;
let sci = 1.5e6;    // 1,500,000
let hex = 0xff;     // 255
let bin = 0b1010;   // 10 (binary)
let oct = 0o17;     // 15 (octal)

// Special values:
Infinity              // Result of 1 / 0
-Infinity             // Result of -1 / 0
NaN                   // "Not a Number" — result of invalid arithmetic (e.g. "abc" * 2)
Number.isNaN(NaN)     // true — use this instead of the global isNaN(), which coerces input
Number.isFinite(42)   // true
Number.isInteger(4.0) // true

// Arithmetic operators:
5 + 3    // 8    addition
5 - 3    // 2    subtraction
5 * 3    // 15   multiplication
5 / 2    // 2.5  division
5 % 2    // 1    modulo (remainder)
2 ** 10  // 1024 exponentiation

// Floating-point precision caveat:
0.1 + 0.2 === 0.3  // false — evaluates to 0.30000000000000004
Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON  // Correct way to compare floats

// The Math object:
Math.round(4.6)      // 5
Math.floor(4.9)      // 4
Math.ceil(4.1)       // 5
Math.trunc(4.9)      // 4 — removes the decimal without rounding
Math.abs(-7)         // 7
Math.max(1, 5, 3)    // 5
Math.min(1, 5, 3)    // 1
Math.pow(2, 8)       // 256
Math.sqrt(16)        // 4
Math.cbrt(27)        // 3
Math.PI              // 3.141592653589793
Math.E               // 2.718281828459045
Math.log(Math.E)     // 1
Math.log2(8)         // 3
Math.log10(1000)     // 3
Math.random()        // Random float: 0 (inclusive) to 1 (exclusive)

// Common random integer patterns:
Math.floor(Math.random() * 10)      // Random integer 0–9
Math.floor(Math.random() * 6) + 1   // Random integer 1–6 (simulates a die)

// Number formatting:
(1234567.89).toFixed(2)         // "1234567.89"
(0.000123).toExponential(2)     // "1.23e-7"
(1234567).toLocaleString()      // "1,234,567" (locale-dependent)
```

---

## Booleans & Comparisons

```javascript
// Comparison operators (always return a boolean):
5 === 5       // true  — strict equality: same type and value. Always use this.
5 == "5"      // true  — loose equality: performs type coercion. Avoid.
5 !== 6       // true  — strict not-equal. Always use this.
5 != "6"      // true  — loose not-equal. Avoid.
5 > 3         // true
5 >= 5        // true
5 < 10        // true
5 <= 4        // false

// Logical operators:
true && false  // false — AND: both sides must be true
true || false  // true  — OR: at least one side must be true
!true          // false — NOT: inverts the boolean

// Nullish coalescing (??) — uses the right side only if the left is null or undefined:
null ?? "default"       // "default"
0 ?? "default"          // 0    — 0 is not null/undefined, so left side is used
"" ?? "default"         // ""   — empty string is not null/undefined
undefined ?? "fallback" // "fallback"

// Optional chaining (?.) — safely access nested properties without throwing:
user?.address?.city     // Returns undefined instead of throwing TypeError
arr?.[0]                // Safe array index access
obj?.method?.()         // Safe method call

// Falsy values (evaluate to false in a boolean context):
// false, 0, -0, 0n, "", '', ``, null, undefined, NaN

// Everything else is truthy, including:
// "0", "false", [], {}, function(){}
```

---

## Control Flow

### if / else

```javascript
if (score >= 90) {
  console.log("A");
} else if (score >= 80) {
  console.log("B");
} else {
  console.log("C");
}

// Ternary operator — a concise single-expression if/else:
let label = score >= 60 ? "Pass" : "Fail";

// Short-circuit patterns:
let name    = user && user.name;       // Evaluates to user.name only if user is truthy
let display = name || "Anonymous";     // Falls back to "Anonymous" if name is falsy
let value   = input ?? "default";      // Falls back only if input is null or undefined
```

### switch

```javascript
switch (day) {
  case "Mon":
  case "Tue":
  case "Wed":
  case "Thu":
  case "Fri":
    console.log("Weekday");
    break;
  case "Sat":
  case "Sun":
    console.log("Weekend");
    break;
  default:
    console.log("Unknown");
}
// Always include break (or return) — omitting it causes fall-through to the next case.
```

---

## Loops

```javascript
// for loop — most common, when you need an index:
for (let i = 0; i < 5; i++) {
  console.log(i);  // 0, 1, 2, 3, 4
}

// while loop — runs as long as the condition is true:
let i = 0;
while (i < 5) {
  console.log(i++);
}

// do...while — always executes the body at least once:
do {
  console.log(i);
  i++;
} while (i < 5);

// for...of — iterate over iterable values (arrays, strings, Sets, Maps):
for (const item of [10, 20, 30]) {
  console.log(item);  // 10, 20, 30
}

// for...in — iterate over object keys:
for (const key in { a: 1, b: 2 }) {
  console.log(key);   // "a", "b"
}
// Note: Avoid for...in on arrays — the order is not guaranteed and it may iterate
// over inherited properties. Use for...of or .forEach() instead.

// Loop control:
for (let i = 0; i < 10; i++) {
  if (i === 3) continue;  // Skip the rest of this iteration.
  if (i === 7) break;     // Exit the loop entirely.
}

// Labeled loops — for breaking out of a nested loop from the outer level:
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (j === 1) break outer;
  }
}
```

---

## Functions

```javascript
// Function declaration — hoisted, available before it is defined:
function add(a, b) {
  return a + b;
}

// Function expression — not hoisted:
const multiply = function(a, b) {
  return a * b;
};

// Arrow function — concise syntax, does not have its own `this`:
const square  = (x) => x * x;
const greet   = name => `Hello, ${name}!`;       // Single param: parentheses optional
const getObj  = () => ({ key: "value" });         // Returning an object: wrap in ()
const add2    = (a, b) => {
  const result = a + b;
  return result;   // Curly braces require an explicit return statement
};

// Default parameters:
function greet(name = "stranger", greeting = "Hello") {
  return `${greeting}, ${name}!`;
}

// Rest parameters — collect remaining arguments into an array:
function sum(...numbers) {
  return numbers.reduce((acc, n) => acc + n, 0);
}
sum(1, 2, 3, 4);  // 10

// Spread syntax — expand an array or object into individual elements:
Math.max(...[1, 5, 3]);               // 5
const arr = [...[1, 2], ...[3, 4]];  // [1, 2, 3, 4]
```

### Higher-Order Functions

Functions that accept or return other functions.

```javascript
function applyTwice(fn, value) {
  return fn(fn(value));
}
applyTwice(x => x * 2, 3);  // 12

// Returning a function:
function makeMultiplier(factor) {
  return (number) => number * factor;
}
const double = makeMultiplier(2);
const triple = makeMultiplier(3);
double(5);  // 10
triple(5);  // 15
```

### Closures

A closure is a function that retains access to variables from its outer scope, even after that scope has finished executing.

```javascript
function makeCounter() {
  let count = 0;
  return {
    increment: () => ++count,
    decrement: () => --count,
    value:     () => count,
  };
}

const counter = makeCounter();
counter.increment();  // 1
counter.increment();  // 2
counter.value();      // 2
// `count` is private — it cannot be accessed or modified from outside.
```

---

## Arrays

```javascript
// Creating arrays:
const arr     = [1, 2, 3, 4, 5];
const mixed   = [1, "two", true, null, { a: 1 }];
const filled  = Array(5).fill(0);                      // [0, 0, 0, 0, 0]
const fromStr = Array.from("hello");                   // ["h","e","l","l","o"]
const range   = Array.from({ length: 5 }, (_, i) => i + 1); // [1, 2, 3, 4, 5]

// Accessing and modifying:
arr[0]          // 1 — first element
arr.at(-1)      // 5 — last element (negative indexing)
arr.length      // 5
arr[0] = 99;    // Mutate in place

// Adding and removing elements:
arr.push(6)            // Add to end — returns new length
arr.pop()              // Remove from end — returns removed element
arr.unshift(0)         // Add to start — returns new length
arr.shift()            // Remove from start — returns removed element
arr.splice(2, 1)           // Remove 1 element at index 2
arr.splice(2, 0, "a", "b") // Insert at index 2 without removing
arr.splice(1, 2, "x")      // Replace 2 elements at index 1 with "x"

// Searching:
arr.indexOf(3)              // Index of first match, or -1
arr.lastIndexOf(3)          // Index of last match
arr.includes(3)             // true / false
arr.find(x => x > 3)        // First element that passes the test
arr.findIndex(x => x > 3)   // Index of first element that passes the test
arr.findLast(x => x > 3)    // Last element that passes the test

// Iterating:
arr.forEach((item, index) => console.log(index, item));

// Transforming — all return a new array, never mutate the original:
arr.map(x => x * 2)                // [2, 4, 6, 8, 10]
arr.filter(x => x % 2 === 0)       // [2, 4]
arr.reduce((acc, x) => acc + x, 0) // 15 — accumulate to a single value
arr.reduceRight((acc, x) => acc + x, 0) // Same, but iterates right-to-left
arr.flat()                         // Flatten one level deep
arr.flat(Infinity)                 // Flatten all levels
arr.flatMap(x => [x, x * 2])       // Map then flatten one level

// Sorting — mutates the original array:
arr.sort()                              // Lexicographic by default — not suitable for numbers!
arr.sort((a, b) => a - b)              // Ascending numeric
arr.sort((a, b) => b - a)              // Descending numeric
arr.sort((a, b) => a.name.localeCompare(b.name))  // By string property

// To sort without mutating:
const sorted = [...arr].sort((a, b) => a - b);

// Reordering:
arr.reverse()        // Reverse in place — mutates the original

// Extracting a portion:
arr.slice(1, 3)      // [2, 3] — non-destructive, end index is exclusive
arr.slice(-2)        // Last 2 elements

// Combining:
arr.concat([6, 7])   // [1, 2, 3, 4, 5, 6, 7]
[...arr, ...[6, 7]]  // Same result using spread

// Testing:
Array.isArray(arr)          // true
arr.every(x => x > 0)       // true if ALL elements pass the test
arr.some(x => x > 4)        // true if ANY element passes the test

// Converting:
arr.join(" - ")    // "1 - 2 - 3 - 4 - 5"
arr.toString()     // "1,2,3,4,5"

// Destructuring:
const [first, second, ...rest] = [1, 2, 3, 4, 5];
// first = 1, second = 2, rest = [3, 4, 5]

const [a, , b] = [1, 2, 3];  // Skip elements with an empty comma slot
// a = 1, b = 3
```

---

## Objects

```javascript
// Creating objects:
const user = {
  name: "Alice",
  age: 30,
  "favorite color": "blue",  // Quote keys that contain spaces or special characters
};

// Accessing properties:
user.name            // "Alice" — dot notation
user["name"]         // "Alice" — bracket notation (required for dynamic keys)
user["favorite color"]

// Adding and modifying:
user.email = "alice@example.com";
user.age = 31;

// Deleting a property:
delete user.age;

// Checking for a property:
"name" in user              // true — checks own and inherited properties
user.hasOwnProperty("name") // true — checks own properties only

// Computed property names:
const prop = "score";
const obj = { [prop]: 100 };   // { score: 100 }

// Shorthand property names:
const name = "Alice";
const age  = 30;
const person = { name, age };  // Equivalent to { name: name, age: age }

// Shorthand methods:
const calculator = {
  value: 0,
  add(n)      { this.value += n; },
  subtract(n) { this.value -= n; },
};

// Destructuring:
const { name: userName, age, city = "Unknown" } = user;
// userName = "Alice", age = 30, city = "Unknown" (default value)

// Nested destructuring:
const { address: { street, zip } } = user;

// Spread and rest with objects:
const updated     = { ...user, age: 31 };        // Shallow clone with an override
const { name: _, ...withoutName } = user;        // Clone without a specific property

// Object utility methods:
Object.keys(obj)                       // Array of own enumerable keys
Object.values(obj)                     // Array of own enumerable values
Object.entries(obj)                    // Array of [key, value] pairs
Object.fromEntries([["a", 1], ["b", 2]]) // { a: 1, b: 2 }
Object.assign({}, obj)                 // Shallow clone
Object.assign(target, source)          // Copy properties from source into target
Object.freeze(obj)                     // Prevent all modifications
Object.seal(obj)                       // Prevent adding or deleting properties (modification allowed)
Object.create(proto)                   // Create an object with a specified prototype
Object.keys(obj).length                // Count enumerable properties
```

---

## Map & Set

### Map

A `Map` holds key-value pairs where keys can be of any type — unlike plain objects which only support string or symbol keys.

```javascript
const map = new Map();

map.set("name", "Alice");
map.set(42, "the answer");
map.set(true, "boolean key");

map.get("name")   // "Alice"
map.has("name")   // true
map.size          // 3
map.delete("name")
map.clear()

// Iterating:
for (const [key, value] of map) {
  console.log(key, value);
}

[...map.keys()]     // Array of all keys
[...map.values()]   // Array of all values
[...map.entries()]  // Array of [key, value] pairs

// Create from an array of entries:
const map2 = new Map([["a", 1], ["b", 2]]);
```

### Set

A `Set` stores unique values — any duplicate entries are silently ignored.

```javascript
const set = new Set([1, 2, 3, 2, 1]);  // {1, 2, 3}

set.add(4)
set.has(2)      // true
set.delete(2)
set.size        // 3
set.clear()

for (const val of set) console.log(val);
[...set]  // Convert to array

// Remove duplicates from an array:
const unique = [...new Set([1, 2, 2, 3, 3])];  // [1, 2, 3]

// Set operations:
const a = new Set([1, 2, 3]);
const b = new Set([2, 3, 4]);
const union        = new Set([...a, ...b]);                     // {1, 2, 3, 4}
const intersection = new Set([...a].filter(x => b.has(x)));    // {2, 3}
const difference   = new Set([...a].filter(x => !b.has(x)));   // {1}
```

---

## Classes

```javascript
class Animal {
  #sound;  // Private field — only accessible within this class

  constructor(name, sound) {
    this.name  = name;    // Public instance property
    this.#sound = sound;  // Private instance property
  }

  speak() {
    return `${this.name} says ${this.#sound}!`;
  }

  get info() {
    return `${this.name} (${this.constructor.name})`;
  }

  set nickname(value) {
    this.name = value;
  }

  static create(name, sound) {
    return new Animal(name, sound);
  }
}

// Inheritance:
class Dog extends Animal {
  constructor(name) {
    super(name, "woof");  // Must call super() before accessing this
    this.tricks = [];
  }

  learn(trick) {
    this.tricks.push(trick);
  }

  speak() {
    return super.speak() + " (tail wagging)";  // Extend the parent method
  }
}

const dog = new Dog("Rex");
dog.speak();           // "Rex says woof! (tail wagging)"
dog instanceof Dog     // true
dog instanceof Animal  // true

Animal.create("Cat", "meow").speak();  // "Cat says meow!"
```

---

## Prototypes

Every JavaScript object has an internal `[[Prototype]]` link. This chain is the mechanism behind inheritance.

```javascript
function Person(name) {
  this.name = name;
}
Person.prototype.greet = function() {
  return `Hi, I'm ${this.name}`;
};

const alice = new Person("Alice");
alice.greet();  // "Hi, I'm Alice"

// The prototype chain:
// alice -> Person.prototype -> Object.prototype -> null

// Inspecting the chain:
Object.getPrototypeOf(alice) === Person.prototype  // true

// Own property vs inherited:
alice.hasOwnProperty("name")    // true  — defined on alice itself
alice.hasOwnProperty("greet")   // false — inherited from Person.prototype

// Class syntax is syntactic sugar over this prototype-based system.
```

---

## Asynchronous JavaScript

### The Event Loop

JavaScript is single-threaded. Long-running operations (network requests, timers) are handed off to browser/Node APIs. When they complete, their callbacks are placed in a queue. The **event loop** picks up callbacks when the call stack is empty.

```
Call Stack  ->  Web APIs  ->  Callback Queue  ->  (Event Loop)  ->  Call Stack
Microtask Queue (Promises) is processed before the Callback Queue.
```

### Callbacks

The traditional pattern — can become deeply nested ("callback hell") for sequential async operations.

```javascript
setTimeout(() => console.log("1 second later"), 1000);

readFile("data.txt", (err, data) => {
  if (err) throw err;
  parseData(data, (err, parsed) => {
    if (err) throw err;
    saveResult(parsed, (err) => {
      if (err) throw err;
      // This nesting becomes hard to read and maintain.
    });
  });
});
```

### Promises

A Promise represents a future value. It starts as **pending** and settles as either **fulfilled** or **rejected**.

```javascript
// Creating a Promise:
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    if (success) resolve("Data loaded!");
    else reject(new Error("Something went wrong"));
  }, 1000);
});

// Consuming a Promise:
promise
  .then(data  => console.log(data))
  .catch(err  => console.error(err))
  .finally(()  => console.log("Done"));

// Chaining:
fetch("/api/user")
  .then(res  => res.json())
  .then(user => fetch(`/api/posts/${user.id}`))
  .then(res  => res.json())
  .then(posts => console.log(posts))
  .catch(err  => console.error(err));

// Promise combinators:
Promise.all([p1, p2, p3])        // Resolves when ALL settle. Rejects if any reject.
Promise.allSettled([p1, p2, p3]) // Resolves when ALL settle. Never rejects.
Promise.race([p1, p2, p3])       // Resolves/rejects with whichever settles first.
Promise.any([p1, p2, p3])        // Resolves with the first success. Rejects if all fail.
Promise.resolve(value)           // Wrap a value in an already-resolved Promise.
Promise.reject(reason)           // Create an already-rejected Promise.
```

### Async / Await

Syntactic sugar over Promises. Makes asynchronous code read like synchronous code.

```javascript
// An async function always returns a Promise:
async function loadUser(id) {
  try {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) throw new Error(`HTTP error: ${res.status}`);
    const user = await res.json();
    return user;
  } catch (err) {
    console.error("Failed to load user:", err);
    throw err;
  } finally {
    console.log("Request complete");
  }
}

// Running requests in parallel:
const [user, posts] = await Promise.all([
  fetchUser(id),
  fetchPosts(id),
]);

// Sequential (each awaits the previous — slower):
const a = await fetchA();
const b = await fetchB();

// Parallel (both start at the same time — faster):
const [a, b] = await Promise.all([fetchA(), fetchB()]);
```

---

## Error Handling

```javascript
try {
  JSON.parse("invalid json");
} catch (err) {
  console.error(err.name);    // "SyntaxError"
  console.error(err.message); // Human-readable description
  console.error(err.stack);   // Full stack trace
} finally {
  // Executes regardless of whether an error was thrown.
}

// Throwing built-in error types:
throw new Error("Something went wrong");
throw new TypeError("Expected a string");
throw new RangeError("Index out of bounds");
throw new ReferenceError("x is not defined");

// Custom error classes:
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name  = "ValidationError";
    this.field = field;
  }
}

try {
  throw new ValidationError("Required", "email");
} catch (err) {
  if (err instanceof ValidationError) {
    console.log(`Validation failed on field: ${err.field}`);
  } else {
    throw err;  // Re-throw errors you do not know how to handle.
  }
}
```

---

## The DOM

The DOM (Document Object Model) is the browser's structured representation of an HTML document as a tree of objects.

```javascript
// Selecting elements:
document.getElementById("myId")
document.querySelector(".class")          // First match using a CSS selector
document.querySelectorAll("p.intro")      // NodeList of all matches
document.getElementsByClassName("cls")   // Live HTMLCollection
document.getElementsByTagName("div")

// Traversing the tree:
el.parentElement
el.children             // HTMLCollection of child elements
el.firstElementChild
el.lastElementChild
el.nextElementSibling
el.previousElementSibling
el.closest(".container") // Nearest ancestor matching a selector

// Reading and modifying content:
el.textContent           // Plain text only (safe)
el.innerHTML             // HTML string — use with caution, risk of XSS with user input
el.value                 // Value of form inputs
el.textContent = "New text";
el.innerHTML   = "<strong>Bold</strong>";

// Attributes:
el.getAttribute("href")
el.setAttribute("href", "https://example.com")
el.removeAttribute("disabled")
el.hasAttribute("required")
el.dataset.userId        // Reads the data-user-id attribute

// Classes:
el.classList.add("active")
el.classList.remove("active")
el.classList.toggle("active")
el.classList.contains("active")
el.classList.replace("old", "new")

// Inline styles:
el.style.color           = "red";
el.style.backgroundColor = "#fff";
getComputedStyle(el).color  // The actual computed style, including stylesheets

// Creating and inserting elements:
const div = document.createElement("div");
div.textContent = "Hello";
div.className   = "card";
parent.appendChild(div)
parent.prepend(div)
el.insertAdjacentElement("afterend", div)  // Positions: "beforebegin", "afterbegin", "beforeend", "afterend"
el.replaceWith(newEl)

// Removing elements:
el.remove()
parent.removeChild(el)

// Cloning:
const clone = el.cloneNode(true);  // true = deep clone (includes all descendants)
```

### Events

```javascript
// Adding a listener:
el.addEventListener("click", function(event) {
  console.log(event.target);    // The element that triggered the event
  console.log(event.type);      // "click"
  event.preventDefault();       // Suppress default behavior (e.g. link navigation)
  event.stopPropagation();      // Stop the event from bubbling up the DOM tree
});

// Removing a listener — requires a reference to the same function:
function handleClick(e) { /* ... */ }
el.addEventListener("click", handleClick);
el.removeEventListener("click", handleClick);

// Common event types:
// Mouse:    click, dblclick, mouseenter, mouseleave, mousemove, mousedown, mouseup
// Keyboard: keydown, keyup, keypress
// Form:     submit, change, input, focus, blur
// Page:     load, DOMContentLoaded, resize, scroll
// Touch:    touchstart, touchend, touchmove

// Event delegation — handle events for many children on a single parent:
document.querySelector("#list").addEventListener("click", (e) => {
  if (e.target.matches("li")) {
    console.log("Clicked:", e.target.textContent);
  }
});

// One-time listener:
el.addEventListener("click", handler, { once: true });
```

---

## Fetch API & HTTP

```javascript
// Basic GET request:
const res  = await fetch("https://api.example.com/data");
const data = await res.json();

// Fetch only rejects on network failure — always check res.ok for HTTP errors:
if (!res.ok) throw new Error(`HTTP ${res.status}: ${res.statusText}`);

// POST with a JSON body:
const res = await fetch("/api/users", {
  method: "POST",
  headers: {
    "Content-Type":  "application/json",
    "Authorization": "Bearer <token>",
  },
  body: JSON.stringify({ name: "Alice", age: 30 }),
});

// Other HTTP methods:
fetch(url, { method: "PUT",    ... })
fetch(url, { method: "PATCH",  ... })
fetch(url, { method: "DELETE" })

// Response parsing — call only one of these per response:
res.json()        // Parse JSON body
res.text()        // Read as plain text
res.blob()        // Read as a Blob (for file data)
res.arrayBuffer() // Read as raw binary data
res.formData()    // Read as FormData

// Response properties:
res.ok                        // true if status is 200–299
res.status                    // 200, 404, 500, etc.
res.statusText                // "OK", "Not Found", etc.
res.headers.get("Content-Type")

// Cancelling a request:
const controller = new AbortController();
fetch(url, { signal: controller.signal });
controller.abort();
```

---

## JSON

```javascript
// Serialize JavaScript to a JSON string:
JSON.stringify({ name: "Alice", age: 30 })
// '{"name":"Alice","age":30}'

// Pretty-print with indentation:
JSON.stringify(data, null, 2)

// Filter or transform values during serialization:
JSON.stringify(data, (key, value) => {
  if (typeof value === "number") return undefined;  // Exclude all numbers
  return value;
});

// Deserialize a JSON string to JavaScript:
JSON.parse('{"name":"Alice"}')  // { name: "Alice" }

// Simple deep clone (objects only — does not support functions, undefined, circular refs):
const clone = JSON.parse(JSON.stringify(original));

// Supported JSON types: string, number, boolean, null, array, object
// Not supported:        undefined, functions, Symbol, Date (serialized as string), RegExp
```

---

## Web Storage

Web Storage is available in browsers only. Both APIs share the same interface.

```javascript
// localStorage — persists until explicitly cleared:
localStorage.setItem("key", "value")
localStorage.getItem("key")    // "value" or null
localStorage.removeItem("key")
localStorage.clear()
localStorage.length

// Storing objects (must serialize to a string):
localStorage.setItem("user", JSON.stringify({ name: "Alice" }))
const user = JSON.parse(localStorage.getItem("user"))

// sessionStorage — cleared when the browser tab is closed:
sessionStorage.setItem("key", "value")
sessionStorage.getItem("key")
```

---

## Modules

ES Modules allow you to split code across files with explicit imports and exports.

```javascript
// math.js — named exports:
export const PI = 3.14159;
export function add(a, b)      { return a + b; }
export function subtract(a, b) { return a - b; }

// user.js — default export:
export default class User {
  constructor(name) { this.name = name; }
}

// app.js — importing:
import { add, subtract }    from "./math.js";
import { add as sum }       from "./math.js";  // Rename on import
import * as math            from "./math.js";  // Import all as a namespace object
import User                 from "./user.js";  // Import the default export
import User, { add }        from "./user.js";  // Import default and named together

// Dynamic import — load a module lazily at runtime:
const module = await import("./math.js");
module.add(1, 2);

// In HTML, use type="module" to enable ES module behavior:
<script type="module" src="app.js"></script>
```

---

## Built-in Utilities

### Date

```javascript
const now      = new Date();
const specific = new Date("2024-01-15");
const fromTs   = new Date(1705276800000);  // From a Unix timestamp (ms)

now.getFullYear()    // 2024
now.getMonth()       // 0–11 (0 = January)
now.getDate()        // 1–31 (day of the month)
now.getDay()         // 0–6  (0 = Sunday)
now.getHours()       // 0–23
now.getMinutes()     // 0–59
now.getSeconds()     // 0–59
now.getTime()        // Milliseconds since Unix epoch
Date.now()           // Current timestamp in milliseconds

// Formatting:
now.toISOString()            // "2024-01-15T00:00:00.000Z"
now.toLocaleDateString()     // "1/15/2024" (locale-specific)
now.toLocaleString("en-US", {
  weekday: "long",
  year:    "numeric",
  month:   "long",
  day:     "numeric",
})  // "Monday, January 15, 2024"

// Measuring code execution time:
const start   = performance.now();
// ... code ...
const elapsed = performance.now() - start;
```

### Regular Expressions

```javascript
const regex = /hello/i;              // Literal syntax (i = case-insensitive)
const regex2 = new RegExp("hello", "i");

// Flags: i (case-insensitive), g (global/all matches), m (multiline), s (dotAll)

// Testing a pattern:
/hello/i.test("Hello World")  // true

// Matching:
"Hello World".match(/\w+/g)           // ["Hello", "World"]
"Hello World".matchAll(/(\w+)/g)      // Iterator of all matches with capture groups

// Replacing:
"Hello World".replace(/world/i, "JS") // "Hello JS"
"a1b2c3".replace(/\d/g, "#")          // "a#b#c#"

// Extracting capture groups:
const m = "2024-01-15".match(/(\d{4})-(\d{2})-(\d{2})/);
// m[1] = "2024", m[2] = "01", m[3] = "15"

// Named capture groups:
const { year, month, day } = "2024-01-15"
  .match(/(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/).groups;

// Common patterns:
/^\d+$/             // Only digits
/^[a-zA-Z]+$/       // Only letters
/\S+@\S+\.\S+/      // Basic email
/^\+?[\d\s\-()]+$/  // Phone number
/^https?:\/\//      // URL beginning with http or https
```

---

## Advanced Concepts

### The `this` Keyword

The value of `this` depends on how a function is called, not where it is defined.

```javascript
// Inside a method: `this` is the object the method belongs to:
const obj = {
  name: "Alice",
  greet() { return `Hi, I'm ${this.name}`; }
};

// Inside a regular function: `this` is undefined in strict mode, or the global object:
function whoAmI() { console.log(this); }

// Arrow functions do not have their own `this` — they inherit from the surrounding scope:
const obj2 = {
  name: "Bob",
  greet: () => `Hi, I'm ${this.name}`,  // `this` here is NOT obj2
};

// Explicit binding:
function sayHello(greeting) { return `${greeting}, ${this.name}`; }
sayHello.call({ name: "Alice" }, "Hi");        // "Hi, Alice"
sayHello.apply({ name: "Bob" }, ["Hey"]);      // "Hey, Bob"
const greet = sayHello.bind({ name: "Carol" }); // Returns a new bound function
greet("Hello");                                // "Hello, Carol"
```

### Generators

Generator functions can pause execution and yield multiple values on demand.

```javascript
function* range(start, end, step = 1) {
  for (let i = start; i < end; i += step) {
    yield i;
  }
}

for (const n of range(0, 10, 2)) {
  console.log(n);  // 0, 2, 4, 6, 8
}

[...range(1, 6)]   // [1, 2, 3, 4, 5]

// Infinite generator:
function* naturals() {
  let n = 1;
  while (true) yield n++;
}
const gen = naturals();
gen.next().value  // 1
gen.next().value  // 2
```

### Symbols

Symbols are guaranteed to be unique. They are primarily used to define non-enumerable, non-clashing object keys.

```javascript
const id  = Symbol("id");
const id2 = Symbol("id");
id === id2  // false — always unique, even with the same description

const user = {
  [id]: 123,
  name: "Alice",
};
user[id]          // 123
Object.keys(user)  // ["name"] — symbol-keyed properties are hidden

// Well-known symbols allow customization of built-in behavior:
class MyIterable {
  [Symbol.iterator]() {
    // Implement to make the object compatible with for...of
  }
}
```

### Proxy & Reflect

`Proxy` wraps an object and lets you intercept and redefine fundamental operations like property access and assignment.

```javascript
const handler = {
  get(target, key) {
    return key in target ? target[key] : `Property '${key}' not found`;
  },
  set(target, key, value) {
    if (typeof value !== "number") throw new TypeError("Only numbers are allowed.");
    target[key] = value;
    return true;
  }
};

const data = new Proxy({}, handler);
data.x = 42;      // Allowed
data.x             // 42
data.y = "hello";  // TypeError
data.z             // "Property 'z' not found"
```

---

## Performance Patterns

### Debounce

Delays a function call until after a specified pause. Ideal for search inputs or resize handlers.

```javascript
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}
const search = debounce(handleSearch, 300);
```

### Throttle

Ensures a function executes at most once per interval. Ideal for scroll or mousemove events.

```javascript
function throttle(fn, limit) {
  let inThrottle;
  return (...args) => {
    if (!inThrottle) {
      fn(...args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}
window.addEventListener("scroll", throttle(handleScroll, 100));
```

### Memoization

Caches the results of expensive function calls to avoid redundant computation.

```javascript
function memoize(fn) {
  const cache = new Map();
  return (...args) => {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}
```

### Animation and Background Tasks

```javascript
// requestAnimationFrame — schedule updates in sync with the browser's repaint cycle:
function animate() {
  // Update visual state here
  requestAnimationFrame(animate);
}
requestAnimationFrame(animate);

// Web Workers — run CPU-intensive code in a background thread:
const worker = new Worker("worker.js");
worker.postMessage({ data: bigArray });
worker.onmessage = (e) => console.log("Result:", e.data);
```

---

## Quick Reference

| Concept | Syntax |
|---|---|
| Variable (constant) | `const x = 1` |
| Variable (reassignable) | `let y = 2` |
| Arrow function | `const fn = (x) => x * 2` |
| Template literal | `` `Hello ${name}` `` |
| Destructure array | `const [a, b] = arr` |
| Destructure object | `const { x, y } = obj` |
| Spread | `[...a, ...b]` / `{ ...a, ...b }` |
| Rest parameters | `function f(...args) {}` |
| Optional chaining | `obj?.prop?.sub` |
| Nullish coalescing | `value ?? "default"` |
| Ternary | `condition ? a : b` |
| Short-circuit AND | `a && b` |
| Short-circuit OR | `a \|\| b` |
| Async / Await | `const data = await fetch(url)` |
| Array map | `arr.map(x => x * 2)` |
| Array filter | `arr.filter(x => x > 0)` |
| Array reduce | `arr.reduce((acc, x) => acc + x, 0)` |
| Object entries | `Object.entries(obj)` |
| Type check | `typeof x` / `x instanceof Y` |
| Array check | `Array.isArray(x)` |
| Deep clone | `structuredClone(obj)` |
| Import module | `import { fn } from "./mod.js"` |
| Error handling | `try { } catch (e) { }` |
| Class with inheritance | `class Foo extends Bar { }` |
| Promise chain | `.then().catch().finally()` |

---

## Best Practices

- **Always use `===`**, never `==`. Loose equality has non-obvious coercion rules that lead to bugs.
- **Default to `const`**. Use `let` only when reassignment is required. Never use `var`.
- **Prefer `async/await`** over raw `.then()` chains — it is easier to read, reason about, and debug.
- **Do not mutate function arguments** — it produces subtle, hard-to-trace side effects.
- **Use `structuredClone(obj)`** for deep cloning rather than `JSON.parse(JSON.stringify(...))`, which has limitations.
- **Guard against `NaN`** with `Number.isNaN()`, not the global `isNaN()`, which coerces its argument first.
- **Favor early returns** to reduce nesting. Fail fast and make the happy path obvious.
- **Re-throw unknown errors** in catch blocks — only handle errors you know how to recover from.
- **Learn the event loop** — it explains the majority of confusing async behavior in JavaScript.
- **Use `console.trace()`** when debugging async flows — it reveals the full call stack, not just the current frame.
