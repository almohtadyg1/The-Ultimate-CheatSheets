# 🧠 The Ultimate JavaScript Cheat Sheet
> From zero to expert — everything you need to understand and master JavaScript.

---

## 📖 What is JavaScript?

JavaScript (JS) is a **dynamic, interpreted, single-threaded programming language** originally built to make web pages interactive. Today it runs everywhere: browsers, servers (Node.js), mobile apps, desktop apps, and even embedded devices.

**Key traits:**
- **Dynamically typed** — variables can hold any type of value; types are checked at runtime
- **Interpreted** — runs line by line (via a JS engine like V8 in Chrome/Node)
- **Prototype-based** — objects inherit from other objects
- **First-class functions** — functions are values; they can be stored, passed, and returned
- **Single-threaded + event loop** — one thing runs at a time, but async ops are handled non-blockingly

---

## ⚙️ Running JavaScript

```javascript
// In a browser: open DevTools → Console (F12 or Cmd+Option+J)

// In an HTML file:
<script src="app.js"></script>
<script>
  console.log("Hello, world!");
</script>

// In Node.js (terminal):
node app.js
node -e "console.log('Hello')"   // run inline code

// Output to console:
console.log("Hello");          // standard output
console.error("Oops");         // error (red)
console.warn("Watch out");     // warning (yellow)
console.table([{a: 1}, {a: 2}]); // display array/object as table
console.time("label");         // start timer
console.timeEnd("label");      // end timer, print elapsed
```

---

## 📦 Variables

```javascript
// Three ways to declare variables:

var name = "Alice";    // OLD — function-scoped, hoisted, avoid in modern JS
let age = 30;          // block-scoped, can be reassigned
const PI = 3.14159;    // block-scoped, cannot be reassigned (prefer this by default)

// Always prefer const. Use let when you need to reassign. Avoid var.

// Multiple declarations
let x = 1, y = 2, z = 3;

// Uninitialized variable
let score;             // value is undefined

// const with objects/arrays — the binding is constant, not the contents:
const user = { name: "Alice" };
user.name = "Bob";     // ✅ allowed — mutating the object
user = {};             // ❌ TypeError — can't reassign the binding
```

### Hoisting

```javascript
// var declarations are "hoisted" to the top of their scope:
console.log(x);  // undefined (not an error!)
var x = 5;

// let and const are hoisted but NOT initialized (Temporal Dead Zone):
console.log(y);  // ReferenceError
let y = 5;

// Function declarations are fully hoisted:
greet();         // works!
function greet() { console.log("Hi"); }
```

---

## 🔢 Data Types

JavaScript has **8 data types**: 7 primitives + Object.

```javascript
// Primitives (immutable, compared by value):
let str    = "hello";           // String
let num    = 42;                // Number (all numbers, integers and floats)
let big    = 9007199254740991n; // BigInt (for very large integers)
let bool   = true;              // Boolean
let empty  = null;              // Null (intentional absence of value)
let undef  = undefined;         // Undefined (unassigned variable)
let sym    = Symbol("id");      // Symbol (unique identifier)

// Object (mutable, compared by reference):
let obj    = { key: "value" };  // Object
let arr    = [1, 2, 3];         // Array (a special kind of Object)
let fn     = function() {};     // Function (a special kind of Object)

// Check the type:
typeof "hello"      // "string"
typeof 42           // "number"
typeof true         // "boolean"
typeof undefined    // "undefined"
typeof null         // "object"  ← famous JS bug, null is NOT an object
typeof {}           // "object"
typeof []           // "object"  ← arrays are objects too
typeof function(){} // "function"

// Better checks:
Array.isArray([])               // true
null === null                   // true
value instanceof Date           // true/false
Object.prototype.toString.call(value) // "[object Array]" etc.
```

---

## 🔤 Strings

```javascript
let s = "Hello, World!";

// Template literals (backticks) — preferred for interpolation
let name = "Alice";
let greeting = `Hello, ${name}! 2 + 2 = ${2 + 2}`;
let multiline = `Line one
Line two`;

// Common string methods:
s.length                    // 13
s.toUpperCase()             // "HELLO, WORLD!"
s.toLowerCase()             // "hello, world!"
s.trim()                    // remove whitespace from both ends
s.trimStart()               // left trim
s.trimEnd()                 // right trim
s.includes("World")         // true
s.startsWith("Hello")       // true
s.endsWith("!")             // true
s.indexOf("o")              // 4 (first occurrence, -1 if not found)
s.lastIndexOf("o")          // 8
s.slice(0, 5)               // "Hello" (start, end — end is exclusive)
s.slice(-6)                 // "World!" (negative = from end)
s.substring(7, 12)          // "World"
s.replace("World", "JS")    // "Hello, JS!"
s.replaceAll("l", "r")      // "Herro, Worrd!"
s.split(", ")               // ["Hello", "World!"]
s.split("")                 // ["H","e","l","l","o",...]
s.repeat(3)                 // "Hello, World!Hello, World!Hello, World!"
s.padStart(15, "*")         // "**Hello, World!"
s.padEnd(15, "-")           // "Hello, World!--"
s.charAt(0)                 // "H"
s[0]                        // "H" (bracket notation)
s.charCodeAt(0)             // 72 (Unicode code point)
String.fromCharCode(72)     // "H"

// Convert to/from string:
String(42)                  // "42"
(42).toString()             // "42"
(255).toString(16)          // "ff" (hex)
parseInt("42px")            // 42
parseFloat("3.14abc")       // 3.14
Number("42")                // 42
```

---

## 🔢 Numbers & Math

```javascript
// Number basics
let n = 42;
let f = 3.14;
let neg = -7;
let sci = 1.5e6;     // 1,500,000
let hex = 0xff;      // 255
let bin = 0b1010;    // 10 (binary)
let oct = 0o17;      // 15 (octal)

// Special values
Infinity              // 1 / 0
-Infinity             // -1 / 0
NaN                   // Not a Number (e.g. "abc" * 2)
Number.isNaN(NaN)     // true (use this, not global isNaN())
Number.isFinite(42)   // true
Number.isInteger(4.0) // true

// Arithmetic operators
5 + 3    // 8       addition
5 - 3    // 2       subtraction
5 * 3    // 15      multiplication
5 / 2    // 2.5     division
5 % 2    // 1       modulo (remainder)
2 ** 10  // 1024    exponentiation

// Floating point gotcha:
0.1 + 0.2 === 0.3   // false! (0.30000000000000004)
Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON  // ✅ correct comparison

// Math object
Math.round(4.6)      // 5
Math.floor(4.9)      // 4
Math.ceil(4.1)       // 5
Math.trunc(4.9)      // 4 (just removes decimal)
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
Math.random()        // random float: 0 (inclusive) to 1 (exclusive)
Math.floor(Math.random() * 10)      // random int 0–9
Math.floor(Math.random() * 6) + 1   // random int 1–6 (dice)

// Number formatting
(1234567.89).toFixed(2)         // "1234567.89"
(0.000123).toExponential(2)     // "1.23e-7"
(1234567).toLocaleString()      // "1,234,567"
```

---

## ✅ Booleans & Comparisons

```javascript
// Comparison operators (always return boolean)
5 === 5        // true   — strict equality (same type + value) ✅ use this
5 == "5"       // true   — loose equality (type coercion) ⚠️ avoid
5 !== 6        // true   — strict not-equal ✅ use this
5 != "6"       // true   — loose not-equal ⚠️ avoid
5 > 3          // true
5 >= 5         // true
5 < 10         // true
5 <= 4         // false

// Logical operators
true && false  // false  — AND
true || false  // true   — OR
!true          // false  — NOT

// Nullish coalescing — returns right side only if left is null or undefined
null ?? "default"        // "default"
0 ?? "default"           // 0      ← different from ||
"" ?? "default"          // ""     ← different from ||
undefined ?? "fallback"  // "fallback"

// Optional chaining — safely access nested properties
user?.address?.city      // undefined instead of TypeError
arr?.[0]                 // safe array access
obj?.method?.()          // safe method call

// Falsy values (evaluate to false in boolean context):
false, 0, -0, 0n, "", '', ``, null, undefined, NaN

// Everything else is truthy, including:
"0", "false", [], {}, function(){}
```

---

## 🔀 Control Flow

### if / else

```javascript
if (score >= 90) {
  console.log("A");
} else if (score >= 80) {
  console.log("B");
} else {
  console.log("C");
}

// Ternary operator (one-liner if/else)
let label = score >= 60 ? "Pass" : "Fail";

// Short-circuit evaluation
let name = user && user.name;     // name if user exists
let display = name || "Anonymous"; // fallback
let value = input ?? "default";   // only if input is null/undefined
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
// Always include break (or return) — otherwise it "falls through"!
```

---

## 🔁 Loops

```javascript
// for loop
for (let i = 0; i < 5; i++) {
  console.log(i);  // 0, 1, 2, 3, 4
}

// while loop
let i = 0;
while (i < 5) {
  console.log(i++);
}

// do...while (executes at least once)
do {
  console.log(i);
  i++;
} while (i < 5);

// for...of — iterate over iterable values (arrays, strings, Sets, Maps)
for (const item of [10, 20, 30]) {
  console.log(item);     // 10, 20, 30
}

// for...in — iterate over object keys
for (const key in { a: 1, b: 2 }) {
  console.log(key);      // "a", "b"
}
// ⚠️ Avoid for...in on arrays — use for...of or .forEach()

// Loop control
for (let i = 0; i < 10; i++) {
  if (i === 3) continue;  // skip this iteration
  if (i === 7) break;     // exit the loop
}

// Labeled loops (for breaking out of nested loops)
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (j === 1) break outer;
  }
}
```

---

## 🧩 Functions

```javascript
// Function declaration (hoisted)
function add(a, b) {
  return a + b;
}

// Function expression (not hoisted)
const multiply = function(a, b) {
  return a * b;
};

// Arrow function (concise, no own 'this')
const square = (x) => x * x;
const greet = name => `Hello, ${name}!`;     // single param, no parens needed
const getObj = () => ({ key: "value" });      // wrap object in () to avoid ambiguity
const add2 = (a, b) => {
  const result = a + b;
  return result;     // explicit return needed with curly braces
};

// Default parameters
function greet(name = "stranger", greeting = "Hello") {
  return `${greeting}, ${name}!`;
}

// Rest parameters (collect remaining args into array)
function sum(...numbers) {
  return numbers.reduce((acc, n) => acc + n, 0);
}
sum(1, 2, 3, 4);   // 10

// Spread syntax (expand array/object into individual elements)
Math.max(...[1, 5, 3]);          // 5
const arr = [...[1, 2], ...[3, 4]];  // [1, 2, 3, 4]

// Arguments object (old way, avoid — doesn't work in arrow functions)
function oldStyle() {
  console.log(arguments);  // array-like object of all args
}
```

### Higher-Order Functions

Functions that take or return other functions.

```javascript
function applyTwice(fn, value) {
  return fn(fn(value));
}
applyTwice(x => x * 2, 3);   // 12

// Returning a function (closure)
function makeMultiplier(factor) {
  return (number) => number * factor;
}
const double = makeMultiplier(2);
const triple = makeMultiplier(3);
double(5);   // 10
triple(5);   // 15
```

### Closures

A closure is a function that **remembers the variables from its outer scope** even after that scope has finished.

```javascript
function makeCounter() {
  let count = 0;
  return {
    increment: () => ++count,
    decrement: () => --count,
    value: () => count,
  };
}
const counter = makeCounter();
counter.increment();  // 1
counter.increment();  // 2
counter.value();      // 2
// count is private — not accessible from outside
```

---

## 📚 Arrays

```javascript
// Creating arrays
const arr = [1, 2, 3, 4, 5];
const mixed = [1, "two", true, null, { a: 1 }];
const empty = new Array(5);    // [empty × 5]
const filled = Array(5).fill(0);  // [0, 0, 0, 0, 0]
const fromStr = Array.from("hello");  // ["h","e","l","l","o"]
const range = Array.from({ length: 5 }, (_, i) => i + 1);  // [1, 2, 3, 4, 5]

// Accessing & modifying
arr[0]             // 1 (first)
arr.at(-1)         // 5 (last — negative indexing)
arr.length         // 5
arr[0] = 99;       // mutate

// Adding & removing
arr.push(6)        // add to end → returns new length
arr.pop()          // remove from end → returns removed item
arr.unshift(0)     // add to start → returns new length
arr.shift()        // remove from start → returns removed item
arr.splice(2, 1)         // remove 1 element at index 2
arr.splice(2, 0, "a","b") // insert at index 2, remove 0
arr.splice(1, 2, "x")    // replace 2 elements at index 1 with "x"

// Finding
arr.indexOf(3)          // 2 (index, or -1)
arr.lastIndexOf(3)      // last index of 3
arr.includes(3)         // true/false
arr.find(x => x > 3)    // 4 (first matching element)
arr.findIndex(x => x > 3) // 3 (first matching index)
arr.findLast(x => x > 3)  // 5 (last matching)

// Iterating
arr.forEach((item, index) => console.log(index, item));

// Transforming (all return NEW arrays)
arr.map(x => x * 2)              // [2, 4, 6, 8, 10]
arr.filter(x => x % 2 === 0)     // [2, 4]
arr.reduce((acc, x) => acc + x, 0) // 15 (sum)
arr.reduceRight((acc, x) => acc + x, 0) // same but right-to-left
arr.flat()                        // flatten one level deep
arr.flat(Infinity)                // flatten all levels
arr.flatMap(x => [x, x * 2])     // map then flatten one level

// Sorting (mutates the array!)
arr.sort()                        // lexicographic sort by default!
arr.sort((a, b) => a - b)         // ascending numeric
arr.sort((a, b) => b - a)         // descending numeric
arr.sort((a, b) => a.name.localeCompare(b.name))  // by string property

// Reordering
arr.reverse()               // reverses in place (mutates)
const sorted = [...arr].sort() // sort without mutating

// Extracting
arr.slice(1, 3)             // [2, 3] (non-mutating, end exclusive)
arr.slice(-2)               // last 2 elements

// Combining
arr.concat([6, 7])          // [1,2,3,4,5,6,7]
[...arr, ...[6, 7]]         // same with spread

// Checking
Array.isArray(arr)          // true
arr.every(x => x > 0)       // true if ALL pass the test
arr.some(x => x > 4)        // true if ANY pass the test

// Converting
arr.join(" - ")             // "1 - 2 - 3 - 4 - 5"
arr.toString()              // "1,2,3,4,5"

// Destructuring
const [first, second, ...rest] = [1, 2, 3, 4, 5];
// first = 1, second = 2, rest = [3, 4, 5]

const [a, , b] = [1, 2, 3];  // skip elements with empty comma
// a = 1, b = 3
```

---

## 🗂️ Objects

```javascript
// Creating objects
const user = {
  name: "Alice",
  age: 30,
  "favorite color": "blue",  // quoted key for spaces/special chars
};

// Accessing properties
user.name          // "Alice" (dot notation)
user["name"]       // "Alice" (bracket notation — required for dynamic keys)
user["favorite color"]

// Adding & modifying
user.email = "alice@example.com";
user.age = 31;

// Deleting
delete user.age;

// Checking
"name" in user           // true
user.hasOwnProperty("name")  // true

// Computed property names
const prop = "score";
const obj = { [prop]: 100 };   // { score: 100 }

// Shorthand property names
const name = "Alice";
const age = 30;
const person = { name, age };  // same as { name: name, age: age }

// Shorthand methods
const calculator = {
  value: 0,
  add(n) { this.value += n; },   // instead of add: function(n) {...}
  subtract(n) { this.value -= n; }
};

// Destructuring
const { name: userName, age, city = "Unknown" } = user;
// userName = "Alice", age = 30, city = "Unknown" (default)

// Nested destructuring
const { address: { street, zip } } = user;

// Spread & rest with objects
const updated = { ...user, age: 31 };        // shallow clone with override
const { name: _, ...withoutName } = user;    // remove a property

// Object methods
Object.keys(obj)           // ["name", "age"] — own enumerable keys
Object.values(obj)         // ["Alice", 30] — own enumerable values
Object.entries(obj)        // [["name","Alice"], ["age",30]]
Object.fromEntries([["a", 1], ["b", 2]])  // { a: 1, b: 2 }
Object.assign({}, obj)     // shallow clone
Object.assign(target, src) // copy properties from src into target
Object.freeze(obj)         // prevent all modifications
Object.seal(obj)           // prevent add/delete, allow modify
Object.create(proto)       // create object with specific prototype
Object.keys(obj).length    // count properties
```

---

## 🗺️ Map & Set

### Map (key-value pairs, any key type)

```javascript
const map = new Map();

map.set("name", "Alice");
map.set(42, "the answer");
map.set(true, "boolean key");

map.get("name")        // "Alice"
map.has("name")        // true
map.size               // 3
map.delete("name")
map.clear()

// Iterating
for (const [key, value] of map) {
  console.log(key, value);
}
map.forEach((value, key) => console.log(key, value));

[...map.keys()]        // array of keys
[...map.values()]      // array of values
[...map.entries()]     // array of [key, value] pairs

// Create from entries
const map2 = new Map([["a", 1], ["b", 2]]);
```

### Set (unique values)

```javascript
const set = new Set([1, 2, 3, 2, 1]);  // {1, 2, 3} — duplicates removed

set.add(4)
set.has(2)       // true
set.delete(2)
set.size         // 3
set.clear()

for (const val of set) console.log(val);
[...set]         // convert to array

// Remove duplicates from array:
const unique = [...new Set([1, 2, 2, 3, 3])];  // [1, 2, 3]

// Set operations
const a = new Set([1, 2, 3]);
const b = new Set([2, 3, 4]);
const union        = new Set([...a, ...b]);           // {1,2,3,4}
const intersection = new Set([...a].filter(x => b.has(x))); // {2,3}
const difference   = new Set([...a].filter(x => !b.has(x))); // {1}
```

---

## 🏗️ Classes

```javascript
class Animal {
  // Private field (modern JS)
  #sound;

  constructor(name, sound) {
    this.name = name;      // public property
    this.#sound = sound;   // private property
  }

  // Instance method
  speak() {
    return `${this.name} says ${this.#sound}!`;
  }

  // Getter
  get info() {
    return `${this.name} (${this.constructor.name})`;
  }

  // Setter
  set nickname(value) {
    this.name = value;
  }

  // Static method (called on the class, not an instance)
  static create(name, sound) {
    return new Animal(name, sound);
  }
}

// Inheritance
class Dog extends Animal {
  constructor(name) {
    super(name, "woof");   // call parent constructor
    this.tricks = [];
  }

  learn(trick) {
    this.tricks.push(trick);
  }

  // Override parent method
  speak() {
    return super.speak() + " 🐾";  // call parent method
  }
}

const dog = new Dog("Rex");
dog.speak();              // "Rex says woof! 🐾"
dog instanceof Dog        // true
dog instanceof Animal     // true

Animal.create("Cat", "meow").speak();  // "Cat says meow!"
```

---

## 🔗 Prototypes

Every object has a hidden `[[Prototype]]` link — this is how inheritance works under the hood.

```javascript
// Every function has a .prototype object
function Person(name) {
  this.name = name;
}
Person.prototype.greet = function() {
  return `Hi, I'm ${this.name}`;
};
const alice = new Person("Alice");
alice.greet();  // "Hi, I'm Alice"

// The prototype chain lookup:
// alice → Person.prototype → Object.prototype → null

// Inspect prototype:
Object.getPrototypeOf(alice) === Person.prototype  // true
alice.__proto__ === Person.prototype               // true (avoid this)

// Check own vs inherited:
alice.hasOwnProperty("name")    // true  (own)
alice.hasOwnProperty("greet")   // false (inherited)

// Class syntax is syntactic sugar over prototype-based inheritance
```

---

## ⏳ Asynchronous JavaScript

### The Event Loop

JavaScript is single-threaded. Long operations (network, timers) are offloaded to Web APIs / Node internals. When complete, callbacks are queued and the event loop picks them up when the call stack is empty.

```
Call Stack → Web APIs → Callback Queue → (Event Loop) → Call Stack
Microtask Queue (Promises) has higher priority than Callback Queue
```

### Callbacks

```javascript
// The old way — leads to "callback hell"
setTimeout(() => console.log("1 second later"), 1000);

readFile("data.txt", (err, data) => {
  if (err) throw err;
  parseData(data, (err, parsed) => {
    if (err) throw err;
    saveResult(parsed, (err) => {  // deeply nested = callback hell 😱
      if (err) throw err;
    });
  });
});
```

### Promises

A Promise represents a value that will be available in the future: **pending → fulfilled | rejected**.

```javascript
// Creating a promise
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    if (success) resolve("Data loaded!");
    else reject(new Error("Something went wrong"));
  }, 1000);
});

// Consuming a promise
promise
  .then(data => console.log(data))       // on success
  .catch(err => console.error(err))      // on failure
  .finally(() => console.log("Done"));   // always runs

// Chaining promises
fetch("/api/user")
  .then(res => res.json())
  .then(user => fetch(`/api/posts/${user.id}`))
  .then(res => res.json())
  .then(posts => console.log(posts))
  .catch(err => console.error(err));

// Promise combinators
Promise.all([p1, p2, p3])          // waits for ALL — rejects if any fail
Promise.allSettled([p1, p2, p3])   // waits for ALL — never rejects
Promise.race([p1, p2, p3])         // resolves/rejects with the FIRST to settle
Promise.any([p1, p2, p3])          // resolves with FIRST success, rejects if ALL fail
Promise.resolve(value)             // create an already-resolved promise
Promise.reject(reason)             // create an already-rejected promise
```

### Async/Await

Syntactic sugar over Promises — makes async code look synchronous.

```javascript
// async function always returns a Promise
async function loadUser(id) {
  try {
    const res = await fetch(`/api/users/${id}`);  // pause until resolved
    if (!res.ok) throw new Error(`HTTP error: ${res.status}`);
    const user = await res.json();
    return user;
  } catch (err) {
    console.error("Failed to load user:", err);
    throw err;  // re-throw if you want callers to handle it
  } finally {
    console.log("Request complete");
  }
}

// Top-level await (in modules or Node.js)
const user = await loadUser(1);

// Run in parallel with async/await
const [user, posts] = await Promise.all([
  fetchUser(id),
  fetchPosts(id),
]);

// Sequential vs parallel
// Sequential (slower):
const a = await fetchA();
const b = await fetchB();

// Parallel (faster):
const [a, b] = await Promise.all([fetchA(), fetchB()]);
```

---

## ❗ Error Handling

```javascript
// try / catch / finally
try {
  JSON.parse("invalid json");
} catch (err) {
  console.error(err.name);     // "SyntaxError"
  console.error(err.message);  // description
  console.error(err.stack);    // stack trace
} finally {
  // runs whether or not an error occurred
}

// Throwing errors
throw new Error("Something went wrong");
throw new TypeError("Expected a string");
throw new RangeError("Index out of bounds");
throw new ReferenceError("x is not defined");

// Custom error classes
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}

try {
  throw new ValidationError("Required", "email");
} catch (err) {
  if (err instanceof ValidationError) {
    console.log(`Validation failed on field: ${err.field}`);
  } else {
    throw err;  // re-throw unknown errors
  }
}
```

---

## 🌐 The DOM (Browser Only)

The DOM (Document Object Model) is the browser's representation of your HTML as a tree of objects.

```javascript
// Selecting elements
document.getElementById("myId")
document.querySelector(".class")         // first match (CSS selector)
document.querySelectorAll("p.intro")     // NodeList of all matches
document.getElementsByClassName("cls")  // HTMLCollection (live)
document.getElementsByTagName("div")

// Traversing
el.parentElement
el.children                  // HTMLCollection of child elements
el.firstElementChild
el.lastElementChild
el.nextElementSibling
el.previousElementSibling
el.closest(".container")     // nearest ancestor matching selector

// Reading & modifying content
el.textContent               // text only (safe)
el.innerHTML                 // HTML string (⚠️ XSS risk with user input)
el.value                     // for form inputs
el.textContent = "New text";
el.innerHTML = "<strong>Bold</strong>";

// Attributes
el.getAttribute("href")
el.setAttribute("href", "https://example.com")
el.removeAttribute("disabled")
el.hasAttribute("required")
el.dataset.userId            // read data-user-id attribute

// Classes
el.classList.add("active")
el.classList.remove("active")
el.classList.toggle("active")
el.classList.contains("active")  // true/false
el.classList.replace("old", "new")
el.className                     // full class string

// Styles
el.style.color = "red";
el.style.backgroundColor = "#fff";
getComputedStyle(el).color      // actual computed style

// Creating & inserting elements
const div = document.createElement("div");
div.textContent = "Hello";
div.className = "card";
parent.appendChild(div)                    // add at end
parent.prepend(div)                        // add at start
parent.insertBefore(div, referenceNode)
el.insertAdjacentElement("afterend", div)  // "beforebegin","afterbegin","beforeend","afterend"
el.replaceWith(newEl)

// Removing elements
el.remove()
parent.removeChild(el)

// Cloning
const clone = el.cloneNode(true);  // true = deep clone (includes children)
```

### Events

```javascript
// Add an event listener
el.addEventListener("click", function(event) {
  console.log(event.target);   // the element that was clicked
  console.log(event.type);     // "click"
  event.preventDefault();      // stop default action (e.g. link navigation)
  event.stopPropagation();     // stop event from bubbling up
});

// Remove a listener (requires reference to same function)
function handleClick(e) { /* ... */ }
el.addEventListener("click", handleClick);
el.removeEventListener("click", handleClick);

// Common events
"click"       "dblclick"     "mouseenter"    "mouseleave"
"mousemove"   "mousedown"    "mouseup"
"keydown"     "keyup"        "keypress"
"submit"      "change"       "input"         "focus"    "blur"
"load"        "DOMContentLoaded"             "resize"   "scroll"
"touchstart"  "touchend"     "touchmove"

// Event delegation (handle events on a parent for many children)
document.querySelector("#list").addEventListener("click", (e) => {
  if (e.target.matches("li")) {
    console.log("Clicked item:", e.target.textContent);
  }
});

// One-time listener
el.addEventListener("click", handler, { once: true });
```

---

## 🌍 Fetch API & HTTP

```javascript
// Basic GET request
const res = await fetch("https://api.example.com/data");
const data = await res.json();

// Check for errors (fetch only rejects on network failure, not HTTP errors)
if (!res.ok) throw new Error(`HTTP ${res.status}: ${res.statusText}`);

// POST with JSON body
const res = await fetch("/api/users", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": "Bearer <token>",
  },
  body: JSON.stringify({ name: "Alice", age: 30 }),
});

// Other HTTP methods
fetch(url, { method: "PUT", ... })
fetch(url, { method: "PATCH", ... })
fetch(url, { method: "DELETE" })

// Response methods (call only ONE per response)
res.json()         // parse JSON body → Promise
res.text()         // read as plain text → Promise
res.blob()         // read as Blob (file data) → Promise
res.arrayBuffer()  // raw binary → Promise
res.formData()     // FormData → Promise

// Response properties
res.ok             // true if status 200–299
res.status         // 200, 404, 500, etc.
res.statusText     // "OK", "Not Found", etc.
res.headers.get("Content-Type")

// Abort a fetch
const controller = new AbortController();
fetch(url, { signal: controller.signal });
controller.abort();  // cancel the request
```

---

## 🗃️ JSON

```javascript
// Serialize (JS → JSON string)
JSON.stringify({ name: "Alice", age: 30 })
// '{"name":"Alice","age":30}'

JSON.stringify(data, null, 2)    // pretty-print with 2-space indent

// replacer function to filter/transform
JSON.stringify(data, (key, value) => {
  if (typeof value === "number") return undefined;  // exclude numbers
  return value;
});

// Deserialize (JSON string → JS)
JSON.parse('{"name":"Alice"}')   // { name: "Alice" }

// Deep clone an object (simple objects only — no functions, undefined, circular refs)
const clone = JSON.parse(JSON.stringify(original));

// JSON values: string, number, boolean, null, array, object
// NOT supported: undefined, functions, Symbol, Date (becomes string), RegExp
```

---

## 💾 Web Storage (Browser Only)

```javascript
// localStorage — persists until manually cleared
localStorage.setItem("key", "value")
localStorage.getItem("key")         // "value" or null
localStorage.removeItem("key")
localStorage.clear()
localStorage.length

// Store objects (must serialize)
localStorage.setItem("user", JSON.stringify({ name: "Alice" }))
const user = JSON.parse(localStorage.getItem("user"))

// sessionStorage — cleared when tab closes (same API as localStorage)
sessionStorage.setItem("key", "value")
sessionStorage.getItem("key")
```

---

## 🧬 Modules (ES Modules)

```javascript
// math.js — named exports
export const PI = 3.14159;
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }

// user.js — default export
export default class User {
  constructor(name) { this.name = name; }
}

// app.js — importing
import { add, subtract } from "./math.js";
import { add as sum } from "./math.js";     // rename on import
import * as math from "./math.js";          // import all as namespace
import User from "./user.js";               // import default
import User, { add } from "./user.js";      // import default + named

// Dynamic import (lazy loading)
const module = await import("./math.js");
module.add(1, 2);

// In HTML: type="module" required for ES modules
<script type="module" src="app.js"></script>
```

---

## 🔧 Useful Built-ins

### Date

```javascript
const now = new Date();
const specific = new Date("2024-01-15");
const ts = new Date(1705276800000);     // from Unix timestamp

now.getFullYear()    // 2024
now.getMonth()       // 0–11 (0 = January!)
now.getDate()        // 1–31 (day of month)
now.getDay()         // 0–6 (0 = Sunday)
now.getHours()       // 0–23
now.getMinutes()     // 0–59
now.getSeconds()     // 0–59
now.getTime()        // Unix timestamp in ms
Date.now()           // current Unix timestamp (ms)

// Formatting
now.toISOString()            // "2024-01-15T00:00:00.000Z"
now.toLocaleDateString()     // "1/15/2024" (locale-specific)
now.toLocaleString("en-US", {
  weekday: "long",
  year: "numeric",
  month: "long",
  day: "numeric"
})  // "Monday, January 15, 2024"

// Measuring time
const start = performance.now();
// ... code ...
const elapsed = performance.now() - start;
```

### Regular Expressions

```javascript
const regex = /hello/i;           // literal (i = case-insensitive)
const regex2 = new RegExp("hello", "i");

// Flags: i (case insensitive), g (global), m (multiline), s (dotAll)

// Testing
/hello/i.test("Hello World")        // true

// Matching
"Hello World".match(/\w+/g)         // ["Hello", "World"]
"Hello World".matchAll(/(\w+)/g)    // iterator of all matches with groups

// Replacing
"Hello World".replace(/world/i, "JS")        // "Hello JS"
"a1b2c3".replace(/\d/g, "#")                // "a#b#c#"
"Hello World".replaceAll("o", "0")          // "Hell0 W0rld"

// Extracting groups
const m = "2024-01-15".match(/(\d{4})-(\d{2})-(\d{2})/);
m[1]  // "2024", m[2] // "01", m[3] // "15"

// Named groups
const { year, month, day } = "2024-01-15"
  .match(/(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/).groups;

// Common patterns
/^\d+$/          // only digits
/^[a-zA-Z]+$/   // only letters
/\S+@\S+\.\S+/  // simple email
/^\+?[\d\s\-()]+$/ // phone number
/^https?:\/\//   // URL starting with http or https
```

---

## 🎯 Destructuring & Spread (Deep Dive)

```javascript
// Swap variables
let a = 1, b = 2;
[a, b] = [b, a];   // a = 2, b = 1

// Function parameter destructuring
function display({ name, age = 0, address: { city } = {} }) {
  console.log(name, age, city);
}

// Dynamic key destructuring
const key = "name";
const { [key]: value } = { name: "Alice" };  // value = "Alice"

// Nested object spread (deep clone one level)
const state = { user: { name: "Alice", age: 30 }, theme: "dark" };
const newState = { ...state, user: { ...state.user, age: 31 } };
```

---

## 💡 Advanced Concepts

### `this` Keyword

```javascript
// In a method: 'this' refers to the object
const obj = {
  name: "Alice",
  greet() { return `Hi, I'm ${this.name}`; }
};

// In a regular function: 'this' is undefined (strict) or global object
function whoAmI() { console.log(this); }

// Arrow functions: inherit 'this' from surrounding scope
const obj2 = {
  name: "Bob",
  greet: () => `Hi, I'm ${this.name}`,  // ⚠️ 'this' is NOT obj2 here!
};

// Explicit binding
function sayHello(greeting) { return `${greeting}, ${this.name}`; }
sayHello.call({ name: "Alice" }, "Hi");       // "Hi, Alice"
sayHello.apply({ name: "Bob" }, ["Hey"]);     // "Hey, Bob"
const greet = sayHello.bind({ name: "Carol" }); // returns new function
greet("Hello");                               // "Hello, Carol"
```

### Generators

```javascript
function* range(start, end, step = 1) {
  for (let i = start; i < end; i += step) {
    yield i;
  }
}

for (const n of range(0, 10, 2)) {
  console.log(n);  // 0, 2, 4, 6, 8
}

[...range(1, 6)]  // [1, 2, 3, 4, 5]

// Infinite generator
function* naturals() {
  let n = 1;
  while (true) yield n++;
}
const gen = naturals();
gen.next().value  // 1
gen.next().value  // 2
```

### Symbols

```javascript
const id = Symbol("id");       // unique identifier
const id2 = Symbol("id");
id === id2                     // false — always unique

// Use as object keys that won't clash
const user = {
  [id]: 123,
  name: "Alice",
};
user[id]                       // 123
Object.keys(user)              // ["name"] — symbols are hidden!

// Well-known symbols (customize object behavior)
class MyArray {
  [Symbol.iterator]() {
    // make object iterable with for...of
  }
}
```

### Proxy & Reflect

```javascript
const handler = {
  get(target, key) {
    return key in target ? target[key] : `Property '${key}' not found`;
  },
  set(target, key, value) {
    if (typeof value !== "number") throw new TypeError("Only numbers!");
    target[key] = value;
    return true;
  }
};

const data = new Proxy({}, handler);
data.x = 42;      // ✅
data.x            // 42
data.y = "hello"; // ❌ TypeError
data.z            // "Property 'z' not found"
```

---

## 🚀 Performance Tips

```javascript
// Debounce — delay execution until after a pause (good for search inputs)
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}
const search = debounce(handleSearch, 300);

// Throttle — execute at most once per interval (good for scroll events)
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

// Memoization — cache expensive function results
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

// requestAnimationFrame for smooth animations
function animate() {
  // update state
  requestAnimationFrame(animate);
}
requestAnimationFrame(animate);

// Web Workers for CPU-heavy tasks (runs in background thread)
const worker = new Worker("worker.js");
worker.postMessage({ data: bigArray });
worker.onmessage = (e) => console.log("Result:", e.data);
```

---

## 📋 Quick Reference Card

| Concept | Syntax |
|---------|--------|
| Variable | `const x = 1` / `let y = 2` |
| Arrow function | `const fn = (x) => x * 2` |
| Template literal | `` `Hello ${name}` `` |
| Destructure array | `const [a, b] = arr` |
| Destructure object | `const { x, y } = obj` |
| Spread | `[...a, ...b]` / `{...a, ...b}` |
| Rest params | `function f(...args) {}` |
| Optional chaining | `obj?.prop?.sub` |
| Nullish coalescing | `value ?? "default"` |
| Ternary | `a ? b : c` |
| Short-circuit | `a && b` / `a \|\| b` |
| Async/await | `const data = await fetch(url)` |
| Array map | `arr.map(x => x * 2)` |
| Array filter | `arr.filter(x => x > 0)` |
| Array reduce | `arr.reduce((acc, x) => acc + x, 0)` |
| Object entries | `Object.entries(obj)` |
| Check type | `typeof x` / `x instanceof Y` |
| Check array | `Array.isArray(x)` |
| Import module | `import { fn } from "./mod.js"` |
| Error handling | `try { } catch(e) { }` |
| Class | `class Foo extends Bar { }` |
| Promise | `.then().catch().finally()` |

---

## 💡 Pro Tips

- **Use `===` always**, never `==` — loose equality has surprising coercion rules.
- **`const` by default**, `let` when needed, **never `var`**.
- **Prefer `async/await`** over raw `.then()` chains for readability.
- **Never mutate function arguments** — it leads to subtle bugs.
- **Avoid `null` — use `undefined`** for "not set"; reserve `null` for explicit "intentional empty".
- **Use `structuredClone(obj)`** (modern JS) for a proper deep clone instead of `JSON.parse(JSON.stringify(...))`.
- **`Array.from`** and spread `[...]` are your go-to tools for converting array-likes.
- **Guard against `NaN`** with `Number.isNaN()`, not the global `isNaN()` (which coerces).
- **Early returns** reduce nesting and improve readability — fail fast.
- **The reflog of async JS is `console.trace()`** — it shows the full call stack, invaluable for debugging async flows.
- **Learn the event loop deeply** — it explains 90% of confusing JS behavior.

---

*ECMAScript version reference: ES2024 | Works in all modern browsers and Node.js 18+*
