# JavaScript: A Complete Progressive Tutorial

---

## 1. What & Why

JavaScript is the only programming language that runs natively in web browsers, making it essential for any frontend development. Through Node.js, it also runs on servers, making it the only language where you can write both the browser-side and server-side of a web application in the same language, sharing code between them.

Originally designed to add interactivity to web pages in the mid-1990s, JavaScript has evolved dramatically. Modern JavaScript (ES2015 and later) is a capable, expressive language with features that rival Python or Ruby. It has the largest package ecosystem in the world (npm), an enormous developer community, and is the foundation of the entire modern web stack.

Key traits: dynamically typed, interpreted, single-threaded with an event loop for async I/O, prototype-based inheritance, first-class functions. Its weaknesses are well-known — type coercion surprises, `this` binding confusion, callback hell (now mostly solved) — but understanding why these work the way they do makes you a far more effective developer.

---

## 2. Mental Model

JavaScript runs in a single thread using an event loop. There is no second thread waiting to run your code — everything happens in sequence. Asynchronous operations (network requests, timers) are handled by handing them off to browser/Node APIs, and their callbacks get placed in a queue when complete.

```
Your Code (Call Stack)          Event Loop Model
──────────────────────          ──────────────────────────────────────
function main() {              ┌─────────────┐    ┌──────────────┐
  fetch('/api/data')  ─────────▶   Web APIs  │    │  Task Queue  │
    .then(process)             │  (browser/  │───▶│ (callbacks   │
}                              │   Node.js)  │    │  and events) │
                               └─────────────┘    └──────┬───────┘
                                                         │
Call stack:                                              ▼
[main] runs synchronously ────────────────────── Event Loop
When stack is EMPTY, the                        (picks next task
event loop takes the next                        when stack empty)
task from the queue
```

Variables are references to values. Primitives (numbers, strings, booleans) are passed by value. Objects and arrays are passed by reference — meaning two variables can point to the same object.

---

## 3. Progressive Examples

### Level 1: Variables, Types, and Operators

```javascript
// Use const by default. Use let when you need to reassign. Never use var.
const name = "Alice";    // block-scoped, cannot be reassigned
let count = 0;           // block-scoped, can be reassigned
count++;                 // count is now 1

// const with objects: the BINDING is constant, not the content
const user = { name: "Alice" };
user.name = "Bob";       // OK — mutating the object
user.age = 30;           // OK — adding a property
// user = {};            // TypeError — can't reassign the binding

// The 8 types: string, number, bigint, boolean, null, undefined, symbol, object

// Type checking
typeof "hello"     // "string"
typeof 42          // "number"
typeof true        // "boolean"
typeof undefined   // "undefined"
typeof null        // "object"  ← famous bug, null is NOT an object
typeof {}          // "object"
typeof []          // "object"  ← arrays are objects too
typeof function(){} // "function"

// Better checks:
Array.isArray([])              // true
value === null                 // check for null
value === undefined            // check for undefined
value == null                  // checks BOTH null and undefined (nullish check)

// Type coercion — JavaScript's most surprising feature
console.log(1 + "2")      // "12" — number coerced to string
console.log("5" - 2)      // 3   — string coerced to number
console.log(true + 1)     // 2   — true is 1
console.log([] + {})      // "[object Object]"
console.log([] == false)  // true

// Always use === (strict equality) — it never coerces types
console.log(1 === "1")    // false — different types
console.log(1 == "1")     // true  — after coercion (avoid this)
console.log(null === undefined)  // false
console.log(null == undefined)   // true (special case in loose equality)

// Template literals — always prefer over string concatenation
const greeting = `Hello, ${name}! Count: ${count * 2}`;
const multiline = `
  Line 1
  Line 2
`;

// Nullish coalescing ?? — only falls through on null/undefined (not 0 or "")
const port = config.port ?? 3000;    // use 3000 only if config.port is null/undefined
const title = "" || "Default";       // "Default" — || checks truthiness
const title2 = "" ?? "Default";      // "" — ?? only checks null/undefined

// Optional chaining ?. — safely access nested properties
const city = user?.address?.city;    // undefined (no error) if any part is null/undefined
const len = user?.friends?.length;   // undefined if friends doesn't exist
user?.greet?.();                     // call method only if it exists
```

### Level 2: Functions — All the Forms

```javascript
// Function declaration — hoisted, available before definition
function add(a, b) {
    return a + b;
}

// Function expression — not hoisted, like a variable
const multiply = function(a, b) {
    return a * b;
};

// Arrow function — concise, lexical 'this' binding
const square = x => x * x;              // single param, implicit return
const sum = (a, b) => a + b;            // multiple params
const greet = name => {                  // block body needs explicit return
    const msg = `Hello, ${name}!`;
    return msg;
};

// Default parameters
function connect(host = "localhost", port = 5432) {
    return `${host}:${port}`;
}
connect()                // "localhost:5432"
connect("db.example.com")  // "db.example.com:5432"

// Rest parameters — collects remaining args into an array
function sum(...numbers) {
    return numbers.reduce((total, n) => total + n, 0);
}
sum(1, 2, 3, 4, 5)   // 15

// Destructuring parameters
function displayUser({ name, age = 0, role = "user" }) {
    console.log(`${name} (${age}) — ${role}`);
}
displayUser({ name: "Alice", age: 30 })   // "Alice (30) — user"

// Spread operator — expand array/object into arguments
const nums = [1, 2, 3];
Math.max(...nums)         // 3 — same as Math.max(1, 2, 3)
const copy = [...nums, 4, 5];  // [1, 2, 3, 4, 5]

// Higher-order functions
const double = x => x * 2;
[1, 2, 3].map(double)              // [2, 4, 6]
[1, 2, 3, 4].filter(x => x > 2)  // [3, 4]
[1, 2, 3, 4].reduce((acc, x) => acc + x, 0)  // 10

// Closures — functions that capture their defining scope
function makeCounter(start = 0) {
    let count = start;     // captured in closure
    return {
        increment() { return ++count; },
        decrement() { return --count; },
        value() { return count; }
    };
}
const counter = makeCounter(10);
counter.increment();   // 11
counter.increment();   // 12
counter.value();       // 12
```

### Level 3: Arrays and Objects — Modern Patterns

```javascript
// Array methods — the bread and butter of modern JS

const users = [
    { name: "Alice", age: 30, role: "admin" },
    { name: "Bob",   age: 25, role: "user"  },
    { name: "Carol", age: 35, role: "admin" },
    { name: "Dave",  age: 28, role: "user"  },
];

// map: transform each element → new array of same length
const names = users.map(u => u.name);
// ["Alice", "Bob", "Carol", "Dave"]

// filter: keep elements matching predicate
const admins = users.filter(u => u.role === "admin");
// [Alice, Carol]

// find: first matching element (or undefined)
const alice = users.find(u => u.name === "Alice");

// every / some: boolean checks
const allAdults = users.every(u => u.age >= 18);    // true
const hasAdmin = users.some(u => u.role === "admin"); // true

// reduce: accumulate to a single value
const totalAge = users.reduce((sum, u) => sum + u.age, 0);  // 118

// sort — MUTATES the original array (clone first if needed)
const sorted = [...users].sort((a, b) => a.age - b.age);  // ascending by age
// Positive result: b comes first. Negative: a comes first. Zero: equal.

// flatMap: map then flatten one level
const tags = [["js", "web"], ["python", "ml"]].flatMap(x => x);
// ["js", "web", "python", "ml"]

// Chaining — the power of array methods
const result = users
    .filter(u => u.role === "admin")
    .map(u => ({ ...u, displayName: u.name.toUpperCase() }))
    .sort((a, b) => a.age - b.age);

// Object operations
const base = { x: 1, y: 2 };
const extended = { ...base, z: 3 };          // spread: { x:1, y:2, z:3 }
const overridden = { ...base, x: 99 };       // { x:99, y:2 }

// Destructuring
const { name, age, role = "guest" } = alice;  // extract with default
const { name: displayName } = alice;          // rename

// Computed property names
const key = "dynamicKey";
const obj = { [key]: "value", [`${key}_2`]: "value2" };

// Object.entries / fromEntries — transform objects
const doubled = Object.fromEntries(
    Object.entries({ a: 1, b: 2, c: 3 }).map(([k, v]) => [k, v * 2])
);   // { a: 2, b: 4, c: 6 }
```

### Level 4: Async JavaScript — Promises and async/await

```javascript
// The problem: network requests are slow. If JS blocked while waiting,
// the browser would freeze. Solution: async callbacks, then Promises, then async/await.

// --- Promises ---
// A Promise represents a value that will be available in the future.
// States: pending → fulfilled (resolved) or rejected

function fetchUser(id) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (id > 0) {
                resolve({ id, name: "Alice" });  // success
            } else {
                reject(new Error("Invalid ID"));  // failure
            }
        }, 100);
    });
}

fetchUser(1)
    .then(user => console.log(user.name))  // called on success
    .catch(err => console.error(err))      // called on failure
    .finally(() => console.log("done"));   // always called

// Promise combinators
Promise.all([fetchUser(1), fetchUser(2)])    // resolves when ALL resolve, rejects if any rejects
    .then(([user1, user2]) => console.log(user1, user2));

Promise.allSettled([fetchUser(1), fetchUser(-1)])  // waits for all, doesn't reject
    .then(results => results.forEach(r => console.log(r.status, r.value ?? r.reason)));

Promise.race([fetchUser(1), fetchUser(2)])  // resolves/rejects with whichever finishes first

// --- async/await --- (preferred in modern code)
// async functions always return a Promise.
// await pauses execution of the async function (not the whole thread).

async function loadUserProfile(userId) {
    try {
        const user = await fetchUser(userId);     // waits for promise to resolve
        const posts = await fetchPosts(user.id);  // waits for next promise
        return { user, posts };
    } catch (error) {
        console.error("Failed to load profile:", error);
        throw error;   // re-throw to caller
    }
}

// Parallel fetching with async/await
async function loadDashboard(userId) {
    // WRONG: sequential — each waits for the previous (slow)
    const user  = await fetchUser(userId);
    const posts = await fetchPosts(userId);

    // CORRECT: parallel — start both at once
    const [user2, posts2] = await Promise.all([
        fetchUser(userId),
        fetchPosts(userId)
    ]);
    return { user: user2, posts: posts2 };
}

// Async iteration — for processing streams or paginated APIs
async function processAllPages(apiUrl) {
    let page = 1;
    let hasMore = true;

    while (hasMore) {
        const { data, nextPage } = await fetch(`${apiUrl}?page=${page}`).then(r => r.json());
        for (const item of data) {
            await processItem(item);
        }
        hasMore = nextPage !== null;
        page++;
    }
}
```

### Level 5: Classes, Modules, and the 'this' Problem

```javascript
// Classes — syntactic sugar over prototype-based inheritance
class Animal {
    #name;      // private field (# prefix, ES2022)
    #sound;

    constructor(name, sound) {
        this.#name = name;
        this.#sound = sound;
    }

    speak() {
        return `${this.#name} says ${this.#sound}!`;
    }

    get name() { return this.#name; }     // getter

    static create(name, sound) {          // static method — on the class, not instances
        return new Animal(name, sound);
    }
}

class Dog extends Animal {
    #tricks = [];

    constructor(name) {
        super(name, "Woof");   // must call super before using 'this'
    }

    learn(trick) {
        this.#tricks.push(trick);
        return this;   // return this for method chaining
    }

    perform() {
        return `${this.name} can: ${this.#tricks.join(", ")}`;
    }
}

const dog = new Dog("Rex");
dog.learn("sit").learn("shake").learn("roll over");
console.log(dog.perform());   // "Rex can: sit, shake, roll over"

// THE 'this' PROBLEM — the most confusing JS concept
function Timer() {
    this.seconds = 0;
}

// WRONG: regular function — 'this' is lost when called as a callback
Timer.prototype.startBad = function() {
    setInterval(function() {
        this.seconds++;     // 'this' is undefined (strict mode) or window (sloppy)
        console.log(this.seconds);
    }, 1000);
};

// CORRECT: arrow function captures 'this' from the enclosing scope (lexical this)
Timer.prototype.startGood = function() {
    setInterval(() => {
        this.seconds++;     // 'this' correctly refers to the Timer instance
        console.log(this.seconds);
    }, 1000);
};

// --- ES Modules ---
// math.js
export const PI = 3.14159;
export function add(a, b) { return a + b; }
export default class Calculator { /* ... */ }

// app.js
import Calculator, { PI, add } from "./math.js";
import * as math from "./math.js";

// Dynamic import — load modules on demand
async function loadFeature() {
    const { heavyFeature } = await import("./heavy-feature.js");
    heavyFeature.initialize();
}
```

### Level 6: Modern Patterns and the Ecosystem

```javascript
// Iterators and generators
function* range(start, end, step = 1) {
    for (let i = start; i < end; i += step) {
        yield i;
    }
}

for (const n of range(0, 10, 2)) {
    console.log(n);  // 0, 2, 4, 6, 8
}

const first5 = [...range(0, 5)];   // [0, 1, 2, 3, 4]

// Proxy — intercept object operations
const handler = {
    get(target, key) {
        console.log(`Reading ${key}`);
        return key in target ? target[key] : `Key ${key} not found`;
    },
    set(target, key, value) {
        if (typeof value !== "number") throw new TypeError("Must be number");
        target[key] = value;
        return true;   // required: indicate success
    }
};

const safeObj = new Proxy({}, handler);
safeObj.x = 42;       // works
// safeObj.y = "hi"; // TypeError

// WeakMap — prevents memory leaks for private data
const privateData = new WeakMap();

class SecureUser {
    constructor(name, secret) {
        privateData.set(this, { secret });  // not accessible from outside
        this.name = name;
    }

    authenticate(input) {
        return input === privateData.get(this).secret;
    }
}

// Error handling patterns
class AppError extends Error {
    constructor(message, code, statusCode = 500) {
        super(message);
        this.name = "AppError";
        this.code = code;
        this.statusCode = statusCode;
    }
}

async function apiCall(endpoint) {
    const response = await fetch(endpoint);
    if (!response.ok) {
        throw new AppError(
            `API request failed: ${response.statusText}`,
            "API_ERROR",
            response.status
        );
    }
    return response.json();
}
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Using `var` in modern code**

```javascript
// WRONG: var is function-scoped and hoisted — creates subtle bugs
for (var i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100);
}
// Prints: 3, 3, 3 — all callbacks share the same 'i', which is 3 by the time they run

// CORRECT: let is block-scoped — each iteration has its own 'i'
for (let i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100);
}
// Prints: 0, 1, 2
```

**Mistake 2: Mutating arrays/objects you didn't intend to**

```javascript
// WRONG: sort mutates the original array
const nums = [3, 1, 4, 1, 5];
const sorted = nums.sort();
console.log(nums);    // [1, 1, 3, 4, 5] — original is mutated!

// CORRECT: clone before sorting
const sorted = [...nums].sort((a, b) => a - b);
console.log(nums);    // [3, 1, 4, 1, 5] — unchanged

// Same with objects — shallow copy is not deep copy
const original = { user: { name: "Alice" } };
const copy = { ...original };
copy.user.name = "Bob";
console.log(original.user.name);  // "Bob" — still shares the nested object!

// Deep copy (modern):
const deepCopy = structuredClone(original);
```

**Mistake 3: Using `==` instead of `===`**

```javascript
// == performs type coercion — results are surprising
0 == false     // true
"" == false    // true
null == undefined  // true
[] == false    // true
[] == ![]      // true  (!!)

// === is always safe — use it exclusively
0 === false    // false
null === undefined  // false
```

**Mistake 4: `async/await` without proper error handling**

```javascript
// WRONG: unhandled promise rejection — errors disappear silently
async function loadData() {
    const data = await fetchData();   // if this throws, the error is swallowed
    return data;
}

// WRONG: forgetting that async functions return Promises
function main() {
    const result = loadData();   // result is a Promise, not the data!
    console.log(result);         // Promise { <pending> }
}

// CORRECT: always handle errors, always await in async context
async function main() {
    try {
        const result = await loadData();
        console.log(result);
    } catch (err) {
        console.error("Failed:", err);
    }
}
```

**Mistake 5: Sequential `await` when parallel is possible**

```javascript
// SLOW: 3 seconds total (each waits for the previous)
async function slow() {
    const a = await fetchA();   // 1 second
    const b = await fetchB();   // 1 second
    const c = await fetchC();   // 1 second
    return [a, b, c];
}

// FAST: ~1 second total (all run in parallel)
async function fast() {
    const [a, b, c] = await Promise.all([fetchA(), fetchB(), fetchC()]);
    return [a, b, c];
}
```

**Mistake 6: Not understanding arrow functions and `this`**

```javascript
const obj = {
    name: "MyObject",
    // Regular method — 'this' depends on HOW it's called
    regularMethod: function() {
        return this.name;   // 'this' = obj when called as obj.regularMethod()
    },
    // Arrow method — 'this' is lexically bound to where it's DEFINED
    arrowMethod: () => {
        return this.name;   // 'this' = outer scope (window/undefined), NOT obj!
    }
};

console.log(obj.regularMethod());  // "MyObject"
console.log(obj.arrowMethod());    // undefined or TypeError
```

---

## 5. The "Why Does This Work" Layer

### How the Event Loop Actually Works

JavaScript's runtime has: a call stack (where functions execute), a heap (where objects live), a task queue (for setTimeout, setInterval callbacks, DOM events), and a microtask queue (for Promise callbacks).

The event loop's algorithm: run all synchronous code until the call stack is empty. Then drain the entire microtask queue. Then take ONE task from the task queue and run it. Then drain microtasks again. Repeat.

This is why `Promise.resolve().then(cb)` runs before `setTimeout(cb, 0)` — Promises go on the microtask queue, which is processed before the task queue.

```javascript
console.log("1");
setTimeout(() => console.log("4"), 0);  // task queue
Promise.resolve().then(() => console.log("3"));  // microtask queue
console.log("2");
// Output: 1, 2, 3, 4
```

### Why Prototype Chains Enable "Inheritance"

Every JavaScript object has an internal `[[Prototype]]` link to another object. When you access a property, JavaScript walks up the prototype chain until it finds the property or reaches `null`.

```javascript
const animal = { breathe() { return "inhale/exhale"; } };
const dog = Object.create(animal);  // dog's prototype is animal
dog.bark = function() { return "woof"; };

dog.bark();    // found on dog itself
dog.breathe(); // not on dog → look on animal → found
// dog.__proto__ === animal

// class syntax is syntactic sugar for exactly this:
class Animal { breathe() { return "inhale/exhale"; } }
class Dog extends Animal { bark() { return "woof"; } }
// Dog.prototype.__proto__ === Animal.prototype
```

---

## 6. Quick Reference

### Variable Declaration

| | `const` | `let` | `var` |
|--|---------|-------|-------|
| Scope | Block | Block | Function |
| Hoisting | TDZ error | TDZ error | undefined |
| Reassignable | No | Yes | Yes |
| Use | Default | When reassigning | Never |

### Array Methods

```javascript
arr.map(fn)         // transform → new array
arr.filter(fn)      // keep matching → new array
arr.reduce(fn, init) // accumulate → single value
arr.find(fn)        // first match or undefined
arr.findIndex(fn)   // index of first match or -1
arr.some(fn)        // true if any match
arr.every(fn)       // true if all match
arr.flat(depth)     // flatten nested arrays
arr.flatMap(fn)     // map then flatten one level
arr.includes(val)   // true if value present
arr.indexOf(val)    // index of value (-1 if absent)
[...arr].sort(fn)   // sort (clone first!)
arr.slice(start, end) // extract without mutating
arr.splice(start, count) // remove/insert (mutates)
```

### Async Patterns

```javascript
// Sequential (when order matters)
for (const item of items) {
    await process(item);
}

// Parallel (when independent)
await Promise.all(items.map(process));

// Parallel with error handling
const results = await Promise.allSettled(items.map(process));
results.forEach(r => {
    if (r.status === 'fulfilled') handleSuccess(r.value);
    else handleError(r.reason);
});

// Race (first one wins)
const fastest = await Promise.race([fetch(url1), fetch(url2)]);
```

### Destructuring Cheat Sheet

```javascript
// Array
const [a, b, ...rest] = [1, 2, 3, 4, 5];
const [, second] = arr;   // skip first

// Object
const { x, y = 0 } = point;          // with default
const { name: alias } = obj;          // rename
const { a: { b } } = nested;          // nested

// Function params
function fn({ name, age = 0 } = {}) {}
```
