# JavaScript & Frontend Tech Stack Interview Questions (5 Years Experience)

---

## Table of Contents
1. [Core JavaScript](#1-core-javascript)
2. [Closures, Scope & Hoisting](#2-closures-scope--hoisting)
3. [Prototypes & OOP](#3-prototypes--oop)
4. [Asynchronous JavaScript](#4-asynchronous-javascript)
5. [ES6+ Features](#5-es6-features)
6. [DOM & Browser APIs](#6-dom--browser-apis)
7. [Error Handling & Debugging](#7-error-handling--debugging)
8. [JavaScript Design Patterns](#8-javascript-design-patterns)
9. [TypeScript](#9-typescript)
10. [HTML5 & Accessibility](#10-html5--accessibility)
11. [CSS & Responsive Design](#11-css--responsive-design)
12. [Web Performance](#12-web-performance)
13. [Testing (Unit, Integration, E2E)](#13-testing)
14. [Build Tools & Module Bundlers](#14-build-tools--module-bundlers)
15. [REST APIs & GraphQL](#15-rest-apis--graphql)
16. [Web Security](#16-web-security)
17. [Git & Version Control](#17-git--version-control)
18. [CI/CD & DevOps Basics](#18-cicd--devops-basics)
19. [PWA & Service Workers](#19-pwa--service-workers)
20. [JS Coding Problems](#20-js-coding-problems)
21. [System Design for Frontend](#21-system-design-for-frontend)
22. [Behavioral Questions for Frontend Devs](#22-behavioral-questions)

---

# PART 1: JAVASCRIPT

## 1. Core JavaScript

### Q1. What are the different data types in JavaScript? Explain primitive vs reference types. ⭐ *Commonly Asked*

**Answer:**

**Primitive types (7):** Stored on the stack, immutable, compared by value.
| Type | Example |
|------|---------|
| `string` | `'hello'` |
| `number` | `42`, `3.14`, `NaN`, `Infinity` |
| `bigint` | `9007199254740991n` |
| `boolean` | `true`, `false` |
| `undefined` | `undefined` |
| `null` | `null` |
| `symbol` | `Symbol('id')` |

**Reference types:** Stored on the heap, mutable, compared by reference.
- `Object`, `Array`, `Function`, `Date`, `RegExp`, `Map`, `Set`, etc.

```javascript
// Primitives - copy by value
let a = 10;
let b = a;
b = 20;
console.log(a); // 10 (unchanged)

// Reference - copy by reference
let obj1 = { name: 'John' };
let obj2 = obj1;
obj2.name = 'Jane';
console.log(obj1.name); // 'Jane' (both point to same object)
```

**Follow-up:** *What is `typeof null` and why?*
`typeof null` returns `'object'` — this is a historical bug in JavaScript from its first implementation. Null is NOT an object; it's a primitive.

---

### Q2. Explain `==` vs `===` and type coercion. ⭐ *Commonly Asked*

**Answer:**

- `==` (loose equality) — Performs **type coercion** before comparison
- `===` (strict equality) — No type coercion, checks both value AND type

```javascript
// == performs type coercion
0 == ''           // true (both coerced to 0)
0 == '0'          // true
'' == '0'         // false
null == undefined // true
false == '0'      // true
NaN == NaN        // false (NaN is never equal to anything)

// === no coercion
0 === ''          // false
0 === '0'         // false
null === undefined // false
```

**Type coercion rules:**
1. If comparing `string` and `number`, string is converted to number
2. If either is `boolean`, it's converted to number (true → 1, false → 0)
3. `null == undefined` is true, but neither equals anything else with `==`
4. Objects are converted via `valueOf()` then `toString()`

**Best practice:** Always use `===` unless you specifically need coercion (e.g., `value == null` to check both null and undefined).

---

### Q3. What is the difference between `var`, `let`, and `const`? ⭐ *Commonly Asked*

**Answer:**

| Feature | `var` | `let` | `const` |
|---------|-------|-------|---------|
| Scope | Function-scoped | Block-scoped | Block-scoped |
| Hoisting | Hoisted (initialized as `undefined`) | Hoisted (TDZ - not initialized) | Hoisted (TDZ) |
| Re-declaration | Allowed | Not allowed | Not allowed |
| Re-assignment | Allowed | Allowed | Not allowed |
| Global property | Yes (`window.x`) | No | No |

```javascript
// var - function scoped, hoisted
console.log(x); // undefined (hoisted)
var x = 5;

// let - block scoped, TDZ
console.log(y); // ReferenceError: Cannot access 'y' before initialization
let y = 5;

// const - must initialize, reference can't change
const obj = { a: 1 };
obj.a = 2;        // OK (mutation allowed)
obj = { a: 2 };   // TypeError: Assignment to constant variable

// Classic loop problem
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 3, 3, 3
}
for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 100); // 0, 1, 2
}
```

**Follow-up:** *What is Temporal Dead Zone (TDZ)?*
The period between entering a scope and the variable being declared. Accessing a `let`/`const` variable in TDZ throws a `ReferenceError`.

---

### Q4. Explain `this` keyword in different contexts. ⭐ *Commonly Asked*

**Answer:**

| Context | `this` refers to |
|---------|-----------------|
| Global (non-strict) | `window` (browser) / `global` (Node) |
| Global (strict mode) | `undefined` |
| Object method | The object calling the method |
| Arrow function | Lexically inherited from enclosing scope |
| Constructor (`new`) | The newly created instance |
| `call`/`apply`/`bind` | Explicitly set object |
| Event handler | The element that received the event |
| Class method | The instance (or undefined if destructured) |

```javascript
const obj = {
  name: 'JS',
  greet() {
    console.log(this.name); // 'JS' (obj is calling)
  },
  greetArrow: () => {
    console.log(this.name); // undefined (lexical - inherits from outer scope)
  },
  greetDelayed() {
    setTimeout(function() {
      console.log(this.name); // undefined (function has its own `this`)
    }, 100);
    setTimeout(() => {
      console.log(this.name); // 'JS' (arrow inherits from greetDelayed)
    }, 100);
  }
};

// Explicit binding
function sayHi() { console.log(this.name); }
sayHi.call({ name: 'Call' });     // 'Call'
sayHi.apply({ name: 'Apply' });   // 'Apply'
const bound = sayHi.bind({ name: 'Bind' });
bound(); // 'Bind'
```

**Follow-up:** *Can you rebind an arrow function's `this`?*
No. Arrow functions don't have their own `this`. `call`, `apply`, and `bind` have no effect on them.

---

### Q5. What is the difference between `null` and `undefined`?

**Answer:**

| Feature | `undefined` | `null` |
|---------|-------------|--------|
| Meaning | Variable declared but not assigned | Intentional absence of value |
| Type | `typeof undefined === 'undefined'` | `typeof null === 'object'` (bug) |
| Default | Function params, uninitialized vars | Must be explicitly assigned |
| Arithmetic | `undefined + 1 = NaN` | `null + 1 = 1` (null → 0) |
| JSON | Excluded from serialization | Included in serialization |

```javascript
let a;           // undefined (not assigned)
let b = null;    // null (explicitly empty)

// Nullish coalescing (handles null/undefined only)
const value = input ?? 'default'; // Only if input is null/undefined
const value2 = input || 'default'; // If input is ANY falsy value (0, '', false)
```

---

### Q6. Explain pass by value vs pass by reference in JavaScript.

**Answer:**

JavaScript is **always pass by value**, but for objects, the "value" is a reference (memory address).

```javascript
// Primitives - pass by value (copy)
function changeValue(x) { x = 100; }
let num = 5;
changeValue(num);
console.log(num); // 5 (unchanged)

// Objects - pass by reference value (shared reference)
function changeProp(obj) {
  obj.name = 'Changed'; // Mutates original (same reference)
}
function reassign(obj) {
  obj = { name: 'New' }; // Local reassignment (doesn't affect original)
}

let person = { name: 'Original' };
changeProp(person);
console.log(person.name); // 'Changed'
reassign(person);
console.log(person.name); // Still 'Changed' (not 'New')
```

---

## 2. Closures, Scope & Hoisting

### Q7. Explain closures with practical examples. ⭐ *Commonly Asked*

**Answer:**

A **closure** is a function that remembers and accesses variables from its outer (enclosing) scope, even after the outer function has returned.

```javascript
// Basic closure - data privacy
function createCounter() {
  let count = 0; // Private, "closed over"
  return {
    increment: () => ++count,
    decrement: () => --count,
    getCount: () => count
  };
}
const counter = createCounter();
counter.increment(); // 1
counter.increment(); // 2
// count is not accessible directly

// Function factory
function createMultiplier(multiplier) {
  return (number) => number * multiplier;
}
const double = createMultiplier(2);
const triple = createMultiplier(3);
double(5); // 10
triple(5); // 15

// Memoization
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const expensiveCalc = memoize((n) => {
  console.log('Computing...');
  return n * n;
});
expensiveCalc(4); // "Computing..." → 16
expensiveCalc(4); // 16 (cached, no log)
```

**Follow-up:** *What are memory implications of closures?*
Closed-over variables remain in memory as long as the closure exists. This can cause memory leaks if large objects are referenced in closures (e.g., event listeners never removed).

---

### Q8. Explain the event loop, call stack, and task queues. ⭐ *Commonly Asked*

**Answer:**

```
┌─────────────────────────────────┐
│         Call Stack               │  ← Executes synchronous code (LIFO)
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│         Event Loop               │  ← Checks: Stack empty? → Pick from queues
└──────┬───────────────────┬──────┘
       │                   │
       ▼                   ▼
┌──────────────┐   ┌──────────────────┐
│ Microtask Q  │   │  Macrotask Q     │
│ (Priority)   │   │  (After micro)   │
├──────────────┤   ├──────────────────┤
│ Promise.then │   │ setTimeout       │
│ queueMicro() │   │ setInterval      │
│ MutationObs  │   │ I/O callbacks    │
│ async/await  │   │ requestAnimFrame │
└──────────────┘   └──────────────────┘
```

**Execution order:**
1. Execute all synchronous code on the call stack
2. When stack is empty, drain ALL microtasks (Promises, queueMicrotask)
3. Execute ONE macrotask (setTimeout, setInterval)
4. Repeat step 2 (drain microtasks again)
5. Render/paint if needed
6. Repeat from step 3

```javascript
console.log('1');                          // Sync
setTimeout(() => console.log('2'), 0);     // Macrotask
Promise.resolve().then(() => console.log('3')); // Microtask
queueMicrotask(() => console.log('4'));    // Microtask
console.log('5');                          // Sync

// Output: 1, 5, 3, 4, 2
```

**Follow-up:** *What happens if microtasks keep adding more microtasks?*
The browser keeps processing microtasks until the queue is empty, potentially blocking rendering and freezing the UI.

---

### Q9. What is hoisting? Explain for variables, functions, and classes.

**Answer:**

| Declaration | Hoisted? | Accessible before declaration? |
|-------------|----------|-------------------------------|
| `function` declaration | Yes (fully) | Yes |
| `var` | Yes (as `undefined`) | Yes (returns `undefined`) |
| `let` / `const` | Yes (TDZ) | No (ReferenceError) |
| `class` | Yes (TDZ) | No (ReferenceError) |
| `function` expression | Variable hoisted only | No |

```javascript
// Function declarations - FULLY hoisted
greet(); // "Hello!" ✓
function greet() { console.log("Hello!"); }

// var - hoisted as undefined
console.log(a); // undefined
var a = 5;

// let/const - TDZ
console.log(b); // ReferenceError
let b = 5;

// Function expression - only var is hoisted
sayHi(); // TypeError: sayHi is not a function
var sayHi = function() { console.log("Hi!"); };
```

---

### Q10. Explain lexical scope and scope chain.

**Answer:**

**Lexical scope** means a variable's scope is determined by where it's written in source code, not where it's called.

```javascript
const globalVar = 'global';

function outer() {
  const outerVar = 'outer';

  function inner() {
    const innerVar = 'inner';
    console.log(innerVar);  // ✓ own scope
    console.log(outerVar);  // ✓ parent scope
    console.log(globalVar); // ✓ global scope
  }
  inner();
}
// Scope chain: inner → outer → global
```

**Scope chain resolution:**
1. Look in current function's scope
2. If not found, look in enclosing function's scope
3. Continue up until global scope
4. If not found → `ReferenceError`

---

## 3. Prototypes & OOP

### Q11. Explain prototypal inheritance in JavaScript. ⭐ *Commonly Asked*

**Answer:**

Every object has an internal `[[Prototype]]` link. When accessing a property, JS walks up the prototype chain.

```javascript
// Prototype chain
const animal = { eat() { return 'eating'; } };
const dog = Object.create(animal);
dog.bark = function() { return 'woof'; };

dog.bark(); // 'woof' (found on dog)
dog.eat();  // 'eating' (found on animal via chain)
// Chain: dog → animal → Object.prototype → null

// ES6 Class (syntactic sugar)
class Animal {
  constructor(name) { this.name = name; }
  speak() { return `${this.name} makes a sound`; }
}

class Dog extends Animal {
  speak() { return `${this.name} barks`; }
}

const d = new Dog('Rex');
d.speak(); // "Rex barks"
// d → Dog.prototype → Animal.prototype → Object.prototype → null
```

**Follow-up:** *Difference between `__proto__` and `prototype`?*
- `__proto__` — The actual prototype link on an instance
- `prototype` — Property on constructor functions, becomes `__proto__` of instances created with `new`

---

### Q12. Explain composition vs inheritance. When would you use each?

**Answer:**

```javascript
// INHERITANCE - "is-a" (tight coupling, fragile base class problem)
class Animal { eat() {} }
class Dog extends Animal { bark() {} }

// COMPOSITION - "has-a" (flexible, preferred)
const canEat = (state) => ({ eat: () => `${state.name} eats` });
const canBark = (state) => ({ bark: () => `${state.name} barks` });
const canSwim = (state) => ({ swim: () => `${state.name} swims` });

function createDog(name) {
  const state = { name };
  return { ...canEat(state), ...canBark(state) };
}

function createDuck(name) {
  const state = { name };
  return { ...canEat(state), ...canSwim(state) };
}
```

**Rule of thumb:** "Favor composition over inheritance" — most real-world entities don't fit neat hierarchies.

---

### Q13. What are getters, setters, and private fields in modern JS?

**Answer:**

```javascript
class Temperature {
  #celsius; // Private field (truly private)

  constructor(celsius) { this.#celsius = celsius; }

  get fahrenheit() { return this.#celsius * 9/5 + 32; }
  set fahrenheit(f) { this.#celsius = (f - 32) * 5/9; }

  get celsius() { return this.#celsius; }
  set celsius(c) {
    if (c < -273.15) throw new Error('Below absolute zero');
    this.#celsius = c;
  }
}

const temp = new Temperature(100);
console.log(temp.fahrenheit); // 212 (getter)
temp.fahrenheit = 32;         // (setter)
console.log(temp.celsius);    // 0
// temp.#celsius → SyntaxError (truly private)
```

---

## 4. Asynchronous JavaScript

### Q14. Explain Promises and their states. ⭐ *Commonly Asked*

**Answer:**

**Promise states:**
- **Pending** → Initial state
- **Fulfilled** → `.then()` called
- **Rejected** → `.catch()` called
- **Settled** → Either fulfilled or rejected (`.finally()`)

```javascript
// Promise combinators
Promise.all([p1, p2, p3]);      // All must succeed, fails fast
Promise.allSettled([p1, p2]);   // Waits for all, never rejects
Promise.race([p1, p2]);        // First to settle (resolve OR reject)
Promise.any([p1, p2]);         // First to resolve (ignores rejections until all fail)
```

```javascript
// Chaining
fetchData()
  .then(data => transform(data))
  .then(result => save(result))
  .catch(error => handleError(error))
  .finally(() => hideLoader());
```

---

### Q15. Explain `async/await` and common patterns. ⭐ *Commonly Asked*

**Answer:**

```javascript
// Sequential vs Parallel
// BAD - sequential (slow)
const users = await getUsers();
const posts = await getPosts();

// GOOD - parallel (fast)
const [users, posts] = await Promise.all([getUsers(), getPosts()]);

// Error handling
async function fetchUser(id) {
  try {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (error) {
    console.error('Fetch failed:', error);
    return null;
  }
}

// Async iteration
async function* fetchPages(url) {
  let page = 1;
  while (true) {
    const res = await fetch(`${url}?page=${page}`);
    const data = await res.json();
    if (data.length === 0) return;
    yield data;
    page++;
  }
}

for await (const page of fetchPages('/api/items')) {
  processItems(page);
}
```

**Follow-up:** *What does `async` actually return?*
Always a Promise. Return value is wrapped in `Promise.resolve()`, thrown errors in `Promise.reject()`.

---

### Q16. What are generators and iterators?

**Answer:**

```javascript
// Generator - can pause/resume with yield
function* fibonacci() {
  let [a, b] = [0, 1];
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

const fib = fibonacci();
fib.next(); // { value: 0, done: false }
fib.next(); // { value: 1, done: false }
fib.next(); // { value: 1, done: false }
fib.next(); // { value: 2, done: false }

// Custom iterator
const range = {
  [Symbol.iterator]() {
    let i = 0;
    return { next: () => i < 5 ? { value: i++, done: false } : { done: true } };
  }
};
for (const n of range) console.log(n); // 0,1,2,3,4
```

---

### Q17. Explain `requestAnimationFrame` vs `setTimeout` for animations.

**Answer:**

| Feature | `setTimeout` | `requestAnimationFrame` |
|---------|-------------|------------------------|
| Timing | Fixed delay (inaccurate) | Synced with display refresh (~60fps) |
| Hidden tab | Continues running | Pauses (saves resources) |
| Smoothness | Can cause jank | Smooth, no frame drops |
| Battery | Wasteful | Efficient |

```javascript
// Smooth animation with rAF
function animate(timestamp) {
  element.style.transform = `translateX(${position}px)`;
  position += 2;
  if (position < 500) requestAnimationFrame(animate);
}
requestAnimationFrame(animate);
```

---

## 5. ES6+ Features

### Q18. Explain destructuring, spread/rest operators. ⭐ *Commonly Asked*

**Answer:**

```javascript
// Object destructuring
const { name, age, address: { city } } = user;
const { name: userName, role = 'user' } = data; // Rename + default

// Array destructuring
const [first, , third, ...rest] = [1, 2, 3, 4, 5]; // first=1, third=3, rest=[4,5]

// Spread (expanding)
const merged = { ...defaults, ...userConfig }; // Object merge
const combined = [...arr1, ...arr2];           // Array concat
const copy = { ...original };                  // Shallow clone

// Rest (collecting)
function sum(...numbers) { return numbers.reduce((a, b) => a + b, 0); }

// Practical patterns
[a, b] = [b, a];                                    // Swap
const { password, ...safeUser } = user;             // Remove property
const config = { ...base, ...(isDev && { debug: true }) }; // Conditional spread
```

---

### Q19. Explain `Map`, `Set`, `WeakMap`, `WeakSet`.

**Answer:**

| Collection | Key Types | Iterable | GC Keys | Use Case |
|------------|-----------|----------|---------|----------|
| `Map` | Any | Yes | No | Key-value with any key type |
| `Set` | Values | Yes | No | Unique values |
| `WeakMap` | Objects only | No | Yes | Private data, caches |
| `WeakSet` | Objects only | No | Yes | Tracking objects |

```javascript
// Map
const cache = new Map();
cache.set(objKey, data); // Object as key
cache.get(objKey);

// Set
const unique = [...new Set([1, 2, 2, 3])]; // [1, 2, 3]

// WeakMap (GC-friendly metadata)
const metadata = new WeakMap();
metadata.set(domElement, { clicks: 0 });
// When domElement is GC'd, metadata entry is too
```

---

### Q20. What are Proxy and Reflect?

**Answer:**

```javascript
// Validation with Proxy
const validated = new Proxy({}, {
  set(target, prop, value) {
    if (prop === 'age' && (typeof value !== 'number' || value < 0)) {
      throw new TypeError('Age must be a positive number');
    }
    return Reflect.set(target, prop, value);
  },
  get(target, prop) {
    if (!(prop in target)) throw new ReferenceError(`${prop} not found`);
    return Reflect.get(target, prop);
  }
});

validated.age = 25;  // ✓
validated.age = -1;  // TypeError
```

**Use cases:** Validation, logging/debugging, reactive systems, API mocking, access control.

---

### Q21. Explain Optional Chaining, Nullish Coalescing, and other modern operators.

**Answer:**

```javascript
// Optional Chaining (?.) - short-circuits to undefined
const city = user?.address?.city;          // Property access
const name = users?.[0]?.name;             // Array access
const result = obj?.method?.();            // Method call

// Nullish Coalescing (??) - only null/undefined trigger default
const port = config.port ?? 3000;          // 0 is valid, won't use default
const name = user.name || 'Anonymous';     // '' would use default (BAD)
const name2 = user.name ?? 'Anonymous';    // '' is kept (GOOD)

// Logical Assignment
user.name ??= 'Anonymous';  // Assign only if null/undefined
config.debug ||= false;     // Assign only if falsy
config.items &&= [];        // Assign only if truthy

// Numeric Separators
const million = 1_000_000;
const bytes = 0xFF_FF_FF;
```

---

## 6. DOM & Browser APIs

### Q22. Explain event delegation and event propagation. ⭐ *Commonly Asked*

**Answer:**

**Propagation phases:** Capturing → Target → Bubbling

```javascript
// Event delegation - one listener on parent
document.getElementById('list').addEventListener('click', (e) => {
  const item = e.target.closest('li.item'); // Find nearest matching ancestor
  if (item) handleItemClick(item.dataset.id);
});

// Benefits:
// - Works for dynamically added elements
// - Less memory (fewer listeners)
// - Simpler cleanup
```

**Key difference:**
- `event.target` — Element that was actually clicked
- `event.currentTarget` — Element the listener is attached to

```javascript
// Stopping propagation
e.stopPropagation();          // Stop bubbling to parent
e.stopImmediatePropagation(); // Stop other listeners on same element too
e.preventDefault();           // Prevent browser default (not propagation)
```

---

### Q23. Explain Web Storage APIs.

**Answer:**

| Storage | Capacity | Expiry | Sent with requests |
|---------|----------|--------|--------------------|
| `localStorage` | ~5-10MB | Never | No |
| `sessionStorage` | ~5-10MB | Tab close | No |
| Cookies | ~4KB | Configurable | Yes (every request!) |
| IndexedDB | Large (GB+) | Never | No |

**Security:**
- Never store auth tokens in localStorage (XSS accessible)
- Use `httpOnly` + `secure` + `SameSite` cookies for tokens
- IndexedDB for large structured offline data

---

### Q24. What is the IntersectionObserver API?

**Answer:**

```javascript
// Lazy loading images
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;
      observer.unobserve(img);
    }
  });
}, { rootMargin: '100px', threshold: 0.1 });

document.querySelectorAll('img[data-src]').forEach(img => observer.observe(img));

// Infinite scroll
const sentinel = document.getElementById('sentinel');
const scrollObserver = new IntersectionObserver(([entry]) => {
  if (entry.isIntersecting) loadMoreItems();
});
scrollObserver.observe(sentinel);
```

---

## 7. Error Handling & Debugging

### Q25. Explain error handling patterns in JavaScript. ⭐ *Commonly Asked*

**Answer:**

```javascript
// Custom Error classes
class ApiError extends Error {
  constructor(status, message, endpoint) {
    super(message);
    this.name = 'ApiError';
    this.status = status;
    this.endpoint = endpoint;
  }
}

// Async error handling
async function fetchData(url) {
  try {
    const res = await fetch(url);
    if (!res.ok) throw new ApiError(res.status, 'Request failed', url);
    return await res.json();
  } catch (error) {
    if (error instanceof ApiError && error.status === 404) {
      return null; // Handle gracefully
    }
    throw error; // Re-throw unexpected
  }
}

// Global handlers
window.addEventListener('error', (e) => logError(e.error));
window.addEventListener('unhandledrejection', (e) => {
  logError(e.reason);
  e.preventDefault();
});
```

---

### Q26. What are common memory leaks and how do you detect them?

**Answer:**

**Common causes:**
1. Forgotten event listeners (not removed)
2. Closures holding large objects
3. Detached DOM nodes still referenced
4. `setInterval` never cleared
5. Accidental globals (missing `let`/`const`)

**Detection:**
- Chrome DevTools → Memory tab → Heap Snapshots (compare before/after)
- Performance Monitor → JS Heap Size trend
- Take 3 snapshots: baseline → action → action again → check for growing retained objects

---

## 8. JavaScript Design Patterns

### Q27. Implement Debounce and Throttle. ⭐ *Commonly Asked*

**Answer:**

```javascript
// Debounce - fires after silence period (search input)
function debounce(fn, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn.apply(this, args), delay);
  };
}

// Throttle - fires at most once per interval (scroll)
function throttle(fn, limit) {
  let inThrottle = false;
  return function(...args) {
    if (!inThrottle) {
      fn.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

// Usage
input.addEventListener('input', debounce(search, 300));
window.addEventListener('scroll', throttle(updatePos, 100));
```

| Pattern | Fires | Use Case |
|---------|-------|----------|
| Debounce | After silence | Search, resize end, autosave |
| Throttle | At intervals | Scroll, mousemove, API polling |

---

### Q28. Explain Module, Singleton, and Observer patterns.

**Answer:**

```javascript
// Module (encapsulation via IIFE or ES modules)
const Counter = (() => {
  let count = 0;
  return { increment: () => ++count, get: () => count };
})();

// Singleton
class DB {
  static #instance;
  static getInstance() {
    if (!DB.#instance) DB.#instance = new DB();
    return DB.#instance;
  }
}

// Observer (pub/sub)
class EventBus {
  #listeners = new Map();
  on(event, fn) {
    if (!this.#listeners.has(event)) this.#listeners.set(event, []);
    this.#listeners.get(event).push(fn);
    return () => this.off(event, fn); // Unsubscribe
  }
  off(event, fn) {
    const fns = this.#listeners.get(event);
    if (fns) this.#listeners.set(event, fns.filter(f => f !== fn));
  }
  emit(event, ...args) {
    (this.#listeners.get(event) || []).forEach(fn => fn(...args));
  }
}
```

---

# PART 2: TYPESCRIPT

## 9. TypeScript

### Q29. Explain the difference between `interface` and `type` in TypeScript. ⭐ *Commonly Asked*

**Answer:**

| Feature | `interface` | `type` |
|---------|-------------|--------|
| Extend | `extends` (merging) | `&` (intersection) |
| Declaration merging | Yes | No |
| Primitives/unions/tuples | No | Yes |
| `implements` | Yes | Yes |
| Computed properties | No | Yes |

```typescript
// Interface - best for object shapes, extendable
interface User {
  name: string;
  email: string;
}
interface Admin extends User {
  role: 'admin';
}

// Declaration merging (interfaces only)
interface Window { myApp: AppConfig; }

// Type - best for unions, computed types, primitives
type Status = 'active' | 'inactive' | 'pending';
type Coords = [number, number];
type Handler = (event: Event) => void;
type Result<T> = { data: T } | { error: string };
```

**Rule of thumb:** Use `interface` for public APIs and object shapes. Use `type` for unions, intersections, mapped types, and complex type transformations.

---

### Q30. Explain Generics with practical examples. ⭐ *Commonly Asked*

**Answer:**

```typescript
// Basic generic function
function identity<T>(arg: T): T { return arg; }

// Generic with constraints
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Generic interface
interface ApiResponse<T> {
  data: T;
  status: number;
  timestamp: Date;
}

// Generic class
class TypedStorage<T> {
  private store = new Map<string, T>();
  set(key: string, value: T): void { this.store.set(key, value); }
  get(key: string): T | undefined { return this.store.get(key); }
}

// Conditional types
type IsArray<T> = T extends any[] ? true : false;
type A = IsArray<string[]>; // true
type B = IsArray<number>;   // false

// Utility types
type Partial<T> = { [K in keyof T]?: T[K] };
type Required<T> = { [K in keyof T]-?: T[K] };
type Pick<T, K extends keyof T> = { [P in K]: T[P] };
type Omit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;
```

---

### Q31. Explain TypeScript utility types and mapped types.

**Answer:**

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

// Built-in utility types
type PartialUser = Partial<User>;          // All optional
type RequiredUser = Required<User>;        // All required
type UserPreview = Pick<User, 'id' | 'name'>; // Only id, name
type SafeUser = Omit<User, 'password'>;    // All except password
type ReadonlyUser = Readonly<User>;        // All readonly
type UserRecord = Record<string, User>;    // { [key: string]: User }

// Custom mapped types
type Nullable<T> = { [K in keyof T]: T[K] | null };
type Getters<T> = { [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K] };

// Template literal types
type EventName = `on${Capitalize<'click' | 'focus' | 'blur'>}`;
// "onClick" | "onFocus" | "onBlur"

// Discriminated unions
type Shape =
  | { kind: 'circle'; radius: number }
  | { kind: 'rect'; width: number; height: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case 'circle': return Math.PI * shape.radius ** 2;
    case 'rect': return shape.width * shape.height;
  }
}
```

---

### Q32. What are type guards and how do you narrow types?

**Answer:**

```typescript
// typeof guard
function padLeft(value: string | number) {
  if (typeof value === 'number') {
    return ' '.repeat(value); // TypeScript knows it's number here
  }
  return value; // TypeScript knows it's string here
}

// instanceof guard
function handleError(error: Error | string) {
  if (error instanceof TypeError) {
    console.log(error.message); // TypeScript knows TypeError properties
  }
}

// in operator guard
interface Fish { swim(): void; }
interface Bird { fly(): void; }
function move(animal: Fish | Bird) {
  if ('swim' in animal) animal.swim();
  else animal.fly();
}

// Custom type guard (type predicate)
function isUser(value: unknown): value is User {
  return typeof value === 'object' && value !== null && 'name' in value && 'email' in value;
}

if (isUser(data)) {
  console.log(data.name); // TypeScript knows it's User
}

// Exhaustive checks with never
function assertNever(value: never): never {
  throw new Error(`Unexpected value: ${value}`);
}
```

---

### Q33. Explain `unknown` vs `any` vs `never`.

**Answer:**

| Type | Assignable TO | Assignable FROM | Use Case |
|------|--------------|-----------------|----------|
| `any` | Anything | Anything | Escape hatch (avoid!) |
| `unknown` | Only after narrowing | Anything | Safe "any" (external data) |
| `never` | Nothing | Nothing | Impossible values, exhaustive checks |

```typescript
// any - bypasses all type checking (dangerous)
let danger: any = 'hello';
danger.nonExistent.method(); // No error at compile time, crashes at runtime

// unknown - must narrow before use (safe)
let safe: unknown = getExternalData();
// safe.name; // Error! Must narrow first
if (typeof safe === 'object' && safe !== null && 'name' in safe) {
  console.log((safe as { name: string }).name); // OK after narrowing
}

// never - function never returns
function throwError(msg: string): never { throw new Error(msg); }
function infiniteLoop(): never { while (true) {} }
```

---

# PART 3: HTML5 & CSS

## 10. HTML5 & Accessibility

### Q34. What are semantic HTML elements and why do they matter? ⭐ *Commonly Asked*

**Answer:**

Semantic elements describe their meaning to both the browser and developer:

```html
<!-- BAD - div soup -->
<div class="header">
  <div class="nav">...</div>
</div>
<div class="main">
  <div class="article">...</div>
  <div class="sidebar">...</div>
</div>
<div class="footer">...</div>

<!-- GOOD - semantic HTML -->
<header>
  <nav aria-label="Main navigation">...</nav>
</header>
<main>
  <article>
    <h1>Title</h1>
    <section>...</section>
  </article>
  <aside>...</aside>
</main>
<footer>...</footer>
```

**Benefits:**
- Accessibility (screen readers understand page structure)
- SEO (search engines weight semantic elements)
- Maintainability (code is self-documenting)
- Default browser behaviors (e.g., `<button>` has keyboard support)

---

### Q35. What is ARIA and how do you make a component accessible? ⭐ *Commonly Asked*

**Answer:**

ARIA (Accessible Rich Internet Applications) adds semantic meaning for assistive technologies.

```html
<!-- Custom dropdown (accessible) -->
<div role="combobox" aria-expanded="true" aria-haspopup="listbox" aria-label="Select country">
  <input aria-autocomplete="list" aria-controls="country-list" />
  <ul id="country-list" role="listbox" aria-label="Countries">
    <li role="option" aria-selected="true" id="opt-1">United States</li>
    <li role="option" aria-selected="false" id="opt-2">Canada</li>
  </ul>
</div>

<!-- Live region (announces dynamic changes) -->
<div aria-live="polite" aria-atomic="true">
  {{ notification }}
</div>

<!-- Skip navigation -->
<a href="#main-content" class="skip-link">Skip to main content</a>
```

**Key principles:**
1. Use native HTML elements first (`<button>`, not `<div onclick>`)
2. All interactive elements must be keyboard accessible
3. Color alone should never convey information
4. Focus management for modals and dynamic content
5. Minimum contrast ratio: 4.5:1 (WCAG AA)

---

### Q36. What are Web Components?

**Answer:**

```javascript
// Custom Element
class MyTooltip extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
  }

  connectedCallback() {
    this.shadowRoot.innerHTML = `
      <style>
        .tooltip { position: relative; display: inline-block; }
        .content { display: none; position: absolute; background: #333; color: white; padding: 8px; border-radius: 4px; }
        :host(:hover) .content { display: block; }
      </style>
      <div class="tooltip">
        <slot></slot>
        <div class="content"><slot name="content"></slot></div>
      </div>
    `;
  }
}

customElements.define('my-tooltip', MyTooltip);
```

**Three pillars:**
1. **Custom Elements** — Define new HTML tags
2. **Shadow DOM** — Encapsulated styling
3. **HTML Templates** — `<template>` and `<slot>` for composition

---

## 11. CSS & Responsive Design

### Q37. Explain CSS specificity and the cascade. ⭐ *Commonly Asked*

**Answer:**

**Specificity (from lowest to highest):**
| Selector | Specificity | Example |
|----------|------------|---------|
| Universal, combinators | 0-0-0 | `*`, `>`, `+` |
| Element, pseudo-element | 0-0-1 | `div`, `::before` |
| Class, attribute, pseudo-class | 0-1-0 | `.btn`, `[type]`, `:hover` |
| ID | 1-0-0 | `#header` |
| Inline style | 1-0-0-0 | `style=""` |
| `!important` | Overrides all | |

```css
/* Specificity calculation */
div.container #main .content p { }  /* 0-1-2-2 (1 ID, 2 classes, 2 elements) */
#header .nav li a:hover { }         /* 0-1-2-2 (1 ID, 2 class+pseudo, 2 elements) */
```

**Cascade order (later wins):**
1. Origin (user-agent → user → author)
2. Specificity
3. Source order (last declaration wins at same specificity)

---

### Q38. Explain Flexbox vs Grid. When would you use each? ⭐ *Commonly Asked*

**Answer:**

| Feature | Flexbox | Grid |
|---------|---------|------|
| Dimension | 1D (row OR column) | 2D (rows AND columns) |
| Use case | Component layout, alignment | Page layout, complex grids |
| Content-driven | Yes (items determine size) | No (grid defines size) |
| Gap support | Yes | Yes |

```css
/* Flexbox - 1D layouts */
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
}

.card-list {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
}

/* Grid - 2D layouts */
.page-layout {
  display: grid;
  grid-template-columns: 250px 1fr 300px;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header header header"
    "sidebar main aside"
    "footer footer footer";
  min-height: 100vh;
}

/* Responsive grid */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 1.5rem;
}
```

**Rule:** Use Flexbox for components (navbar, card content), Grid for page layouts and 2D arrangements.

---

### Q39. Explain CSS Container Queries vs Media Queries.

**Answer:**

```css
/* Media Queries - viewport-based */
@media (max-width: 768px) {
  .card { flex-direction: column; }
}

/* Container Queries - parent container-based (modern CSS) */
.card-wrapper {
  container-type: inline-size;
  container-name: card;
}

@container card (max-width: 400px) {
  .card { flex-direction: column; }
  .card img { width: 100%; }
}

@container card (min-width: 401px) {
  .card { flex-direction: row; }
  .card img { width: 200px; }
}
```

**Benefits of Container Queries:**
- Components adapt to their container size, not viewport
- Truly reusable components (same component works in sidebar AND main area)
- No need for parent to communicate size via classes

---

### Q40. What are CSS Custom Properties (variables) and how do they differ from Sass variables?

**Answer:**

```css
/* CSS Custom Properties - runtime, inherited, dynamic */
:root {
  --primary: #1976d2;
  --spacing-md: 16px;
  --font-body: 'Inter', sans-serif;
}

.theme-dark {
  --primary: #90caf9;
  --bg: #121212;
}

.btn {
  background: var(--primary);
  padding: var(--spacing-md);
  /* Fallback value */
  color: var(--text-color, #333);
}

/* Dynamic theming with JS */
document.documentElement.style.setProperty('--primary', tenant.color);
```

| Feature | CSS Variables | Sass Variables |
|---------|-------------|----------------|
| Runtime | Yes (dynamic) | No (compile-time) |
| Cascade/inheritance | Yes | No |
| JS accessible | Yes | No |
| Responsive changes | Yes (in media queries) | No |
| Scoped | Yes (per selector) | Yes (per scope) |

---

## 12. Web Performance

### Q41. What are Core Web Vitals and how do you optimize them? ⭐ *Commonly Asked*

**Answer:**

| Metric | Measures | Good | Optimization |
|--------|----------|------|-------------|
| **LCP** (Largest Contentful Paint) | Loading speed | < 2.5s | Preload hero image, SSR, CDN, optimize images |
| **INP** (Interaction to Next Paint) | Responsiveness | < 200ms | Break long tasks, web workers, reduce JS |
| **CLS** (Cumulative Layout Shift) | Visual stability | < 0.1 | Set dimensions on media, avoid FOUT, reserve space |

```html
<!-- LCP optimization -->
<link rel="preload" as="image" href="hero.webp" fetchpriority="high">
<img src="hero.webp" width="1200" height="600" alt="Hero" fetchpriority="high">

<!-- CLS prevention -->
<img src="photo.jpg" width="400" height="300" alt="" loading="lazy">
<div style="aspect-ratio: 16/9;"><!-- Video container --></div>

<!-- INP optimization -->
<script type="module">
  // Code-split and defer non-critical JS
  const module = await import('./heavy-feature.js');
</script>
```

---

### Q42. Explain lazy loading strategies for frontend applications.

**Answer:**

```javascript
// 1. Native lazy loading (images/iframes)
<img src="photo.jpg" loading="lazy" alt="">
<iframe src="embed.html" loading="lazy"></iframe>

// 2. Dynamic imports (code splitting)
const module = await import('./feature.js');

// 3. IntersectionObserver (custom lazy loading)
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      loadComponent(entry.target);
      observer.unobserve(entry.target);
    }
  });
});

// 4. Route-based (Angular/React)
{ path: 'admin', loadChildren: () => import('./admin/admin.module') }

// 5. Prefetch on hover/visibility
<link rel="prefetch" href="/next-page.js">
document.addEventListener('mouseover', (e) => {
  if (e.target.matches('a[data-prefetch]')) {
    const link = document.createElement('link');
    link.rel = 'prefetch';
    link.href = e.target.href;
    document.head.appendChild(link);
  }
});
```

---

### Q43. What causes layout thrashing and how do you avoid it?

**Answer:**

```javascript
// BAD - read/write interleaving forces multiple reflows
elements.forEach(el => {
  const height = el.offsetHeight;           // READ → layout
  el.style.height = height * 2 + 'px';     // WRITE → invalidate
});

// GOOD - batch reads, then batch writes
const heights = elements.map(el => el.offsetHeight); // All reads
elements.forEach((el, i) => {
  el.style.height = heights[i] * 2 + 'px';          // All writes
});
```

**Properties that trigger layout:** `offsetHeight`, `clientWidth`, `getBoundingClientRect()`, `scrollTop`, `getComputedStyle()`

**Best practices:**
- Use `transform` and `opacity` for animations (composited, no reflow)
- Use `will-change: transform` sparingly
- Prefer `requestAnimationFrame` for DOM writes

---

## 13. Testing

### Q44. Explain testing pyramid for frontend. What tools would you use? ⭐ *Commonly Asked*

**Answer:**

```
         ╱╲
        ╱ E2E ╲         Few (Cypress, Playwright)
       ╱────────╲       — Full user flows, slow, flaky
      ╱Integration╲     Some (Testing Library, MSW)
     ╱──────────────╲   — Component interactions, API mocking
    ╱   Unit Tests    ╲  Many (Jest, Vitest, Jasmine)
   ╱────────────────────╲— Pure functions, services, pipes
```

| Layer | Tool | Tests |
|-------|------|-------|
| Unit | Jest / Vitest / Jasmine | Services, utils, pipes, pure logic |
| Component | Testing Library / ComponentHarness | Component rendering, interactions |
| Integration | Testing Library + MSW | Multi-component flows with mocked APIs |
| E2E | Cypress / Playwright | Full user journeys in real browser |

```typescript
// Unit test (Jest/Jasmine)
describe('calculateTotal', () => {
  it('should sum items with tax', () => {
    const items = [{ price: 10, qty: 2 }, { price: 5, qty: 1 }];
    expect(calculateTotal(items, 0.1)).toBe(27.5);
  });
});

// Component test (Angular Testing Library)
it('should display user name', async () => {
  await render(UserCardComponent, {
    inputs: { user: { name: 'John', email: 'john@test.com' } }
  });
  expect(screen.getByText('John')).toBeInTheDocument();
});

// E2E (Cypress)
it('should login successfully', () => {
  cy.visit('/login');
  cy.get('[data-testid=email]').type('user@test.com');
  cy.get('[data-testid=password]').type('password123');
  cy.get('[data-testid=submit]').click();
  cy.url().should('include', '/dashboard');
});
```

---

### Q45. What is Test-Driven Development (TDD) and how do you apply it in frontend?

**Answer:**

**TDD Cycle: Red → Green → Refactor**
1. **Red** — Write a failing test first
2. **Green** — Write minimum code to pass
3. **Refactor** — Clean up while keeping tests green

```typescript
// Example: Building a password validator

// Step 1: RED - Write failing test
describe('PasswordValidator', () => {
  it('should reject passwords shorter than 8 chars', () => {
    expect(validate('short')).toEqual({ valid: false, errors: ['Too short'] });
  });
});

// Step 2: GREEN - Minimum code to pass
function validate(password: string) {
  const errors: string[] = [];
  if (password.length < 8) errors.push('Too short');
  return { valid: errors.length === 0, errors };
}

// Step 3: Add more tests, refactor
it('should require at least one number', () => {
  expect(validate('abcdefgh')).toEqual({ valid: false, errors: ['Needs a number'] });
});
```

**Frontend TDD works best for:** Utility functions, services, state management, form validation, pure components.

---

### Q46. How do you mock API calls in tests?

**Answer:**

```typescript
// Jest mock
jest.mock('../api/userService');
const mockGetUser = getUserById as jest.MockedFunction<typeof getUserById>;
mockGetUser.mockResolvedValue({ id: 1, name: 'John' });

// MSW (Mock Service Worker) - intercepts at network level
import { setupServer } from 'msw/node';
import { http, HttpResponse } from 'msw';

const server = setupServer(
  http.get('/api/users/:id', ({ params }) => {
    return HttpResponse.json({ id: params.id, name: 'John' });
  }),
  http.post('/api/users', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({ id: 1, ...body }, { status: 201 });
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// Override for specific test
it('should handle 500 error', async () => {
  server.use(
    http.get('/api/users/:id', () => HttpResponse.json(null, { status: 500 }))
  );
  // Test error handling...
});
```

---

## 14. Build Tools & Module Bundlers

### Q47. Explain the difference between Webpack, Vite, and esbuild. ⭐ *Commonly Asked*

**Answer:**

| Tool | Dev Server | Build | Config | Use Case |
|------|-----------|-------|--------|----------|
| **Webpack** | HMR (slow rebuild) | Versatile | Complex | Legacy apps, enterprise (Angular) |
| **Vite** | Native ESM (instant) | Rollup-based | Minimal | Modern apps, fast DX |
| **esbuild** | - | Extremely fast | Go-based | Build step, transpilation |
| **Rollup** | - | Clean ES modules | Moderate | Libraries |
| **Turbopack** | Rust-based (fast) | In progress | Next.js | Next.js 13+ |

```javascript
// Vite config (minimal)
export default defineConfig({
  plugins: [angular()],
  build: { target: 'es2020', sourcemap: true },
  server: { proxy: { '/api': 'http://localhost:3000' } }
});

// Webpack key concepts
module.exports = {
  entry: './src/index.ts',
  output: { filename: '[name].[contenthash].js' },
  module: { rules: [/* loaders */] },
  plugins: [/* plugins */],
  optimization: { splitChunks: { chunks: 'all' } }
};
```

**Angular CLI uses:** Webpack (legacy) or esbuild (Angular 16+ with `"builder": "@angular-devkit/build-angular:application"`).

---

### Q48. What is tree-shaking and how does it work?

**Answer:**

Tree-shaking eliminates dead code based on ES module static `import`/`export` analysis.

```javascript
// math.js
export function add(a, b) { return a + b; }     // USED
export function multiply(a, b) { return a * b; } // UNUSED

// app.js
import { add } from './math.js';
console.log(add(1, 2));
// `multiply` is NOT included in the bundle

// Things that PREVENT tree-shaking:
// 1. CommonJS (require/module.exports) - not statically analyzable
// 2. Side effects in modules
// 3. Re-exporting everything: export * from './utils'
```

**`package.json` sideEffects:**
```json
{
  "sideEffects": false,  // "This package has no side effects, tree-shake freely"
  // or specify files with side effects:
  "sideEffects": ["*.css", "./src/polyfills.ts"]
}
```

---

### Q49. What is code splitting and how do you implement it?

**Answer:**

```javascript
// 1. Route-based splitting (most common)
const routes = [
  { path: '/dashboard', component: () => import('./Dashboard') },
  { path: '/admin', component: () => import('./Admin') }
];

// 2. Component-based splitting
const HeavyChart = React.lazy(() => import('./HeavyChart'));
// Angular: loadComponent: () => import('./chart.component')

// 3. Vendor splitting (webpack)
optimization: {
  splitChunks: {
    cacheGroups: {
      vendor: { test: /node_modules/, name: 'vendors', chunks: 'all' }
    }
  }
}

// 4. Conditional splitting
if (user.isPremium) {
  const { PremiumFeatures } = await import('./premium');
}

// Analyzing bundles
// ng build --stats-json → webpack-bundle-analyzer
// npx source-map-explorer dist/main.*.js
```

---

## 15. REST APIs & GraphQL

### Q50. Explain REST principles and HTTP methods. ⭐ *Commonly Asked*

**Answer:**

| Method | Purpose | Idempotent | Body |
|--------|---------|-----------|------|
| GET | Read resource | Yes | No |
| POST | Create resource | No | Yes |
| PUT | Replace resource | Yes | Yes |
| PATCH | Partial update | No | Yes |
| DELETE | Remove resource | Yes | No |

**REST principles:**
1. **Stateless** — Each request contains all info needed
2. **Resource-based** — URLs represent resources (`/users/123`)
3. **HTTP methods** — Define action on resource
4. **HATEOAS** — Response includes links to related resources

```javascript
// Good REST API design
GET    /api/users          // List users
GET    /api/users/123      // Get user 123
POST   /api/users          // Create user
PUT    /api/users/123      // Replace user 123
PATCH  /api/users/123      // Update user 123 partially
DELETE /api/users/123      // Delete user 123

// Pagination, filtering, sorting
GET /api/users?page=2&limit=20&sort=-createdAt&role=admin

// Status codes
200 OK, 201 Created, 204 No Content
400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found
500 Internal Server Error
```

---

### Q51. What is GraphQL and how does it compare to REST?

**Answer:**

| Feature | REST | GraphQL |
|---------|------|---------|
| Endpoints | Multiple (`/users`, `/posts`) | Single (`/graphql`) |
| Over-fetching | Common (get entire resource) | Request only needed fields |
| Under-fetching | Multiple requests needed | Single query, nested data |
| Caching | HTTP caching built-in | More complex (Apollo cache) |
| Versioning | URL versions (`/v2/users`) | Evolve schema, deprecate fields |

```graphql
# Query - exactly what you need
query {
  user(id: "123") {
    name
    email
    posts(last: 5) {
      title
      comments { author { name } }
    }
  }
}

# Mutation
mutation {
  createUser(input: { name: "John", email: "john@test.com" }) {
    id
    name
  }
}

# Subscription (real-time)
subscription {
  newMessage(channelId: "general") {
    text
    sender { name }
  }
}
```

**When to use GraphQL:**
- Complex UIs with nested/related data
- Mobile apps (bandwidth optimization)
- Rapid frontend iteration (no waiting for backend API changes)

**When REST is fine:**
- Simple CRUD APIs
- File uploads
- Caching-heavy applications
- Public APIs

---

### Q52. How do you handle API error responses gracefully in the frontend?

**Answer:**

```typescript
// Centralized error handling
class ApiClient {
  async request<T>(url: string, options?: RequestInit): Promise<T> {
    try {
      const response = await fetch(url, {
        ...options,
        headers: { 'Content-Type': 'application/json', ...options?.headers }
      });

      if (!response.ok) {
        const error = await response.json().catch(() => ({}));
        throw new ApiError(response.status, error.message || 'Request failed', url);
      }

      return response.status === 204 ? null as T : await response.json();
    } catch (error) {
      if (error instanceof ApiError) throw error;
      throw new ApiError(0, 'Network error', url);
    }
  }
}

// Error boundary pattern
function withErrorHandling(apiCall, { onNotFound, onForbidden, onError }) {
  return apiCall.catch(error => {
    switch (error.status) {
      case 404: return onNotFound?.() ?? null;
      case 403: return onForbidden?.() ?? redirectToLogin();
      case 429: return retryAfter(error);
      default: return onError?.(error) ?? showToast('Something went wrong');
    }
  });
}
```

---

## 16. Web Security

### Q53. Explain XSS attacks and how to prevent them. ⭐ *Commonly Asked*

**Answer:**

**Types of XSS:**
| Type | Vector | Example |
|------|--------|---------|
| Stored | Saved in DB, served to users | Comment with `<script>` |
| Reflected | URL parameter reflected in page | Search query in response |
| DOM-based | Client-side JS manipulates DOM | `innerHTML = location.hash` |

**Prevention:**
```javascript
// 1. Never use innerHTML with user input
element.textContent = userInput; // Safe (escapes HTML)
// element.innerHTML = userInput; // DANGEROUS

// 2. Sanitize if HTML is needed
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);

// 3. Content Security Policy header
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'

// 4. Escape in templates (Angular does this automatically)
{{ userInput }} // Angular auto-escapes

// 5. HttpOnly cookies (JS can't access)
Set-Cookie: token=abc; HttpOnly; Secure; SameSite=Strict
```

---

### Q54. Explain CSRF attacks and prevention.

**Answer:**

**CSRF** (Cross-Site Request Forgery): Attacker tricks user's browser into making authenticated requests to a target site.

```html
<!-- Attacker's page -->
<img src="https://bank.com/transfer?to=hacker&amount=10000">
<!-- User's browser sends cookies automatically! -->
```

**Prevention:**
1. **CSRF tokens** — Unique token in form/header that attacker can't guess
2. **SameSite cookies** — `SameSite=Strict` or `Lax`
3. **Check Origin/Referer headers**
4. **Double submit cookie pattern**

```javascript
// Angular's built-in XSRF protection
// Server sets XSRF-TOKEN cookie
// Angular reads it and sends as X-XSRF-TOKEN header
// Server validates header matches cookie
HttpClientXsrfModule.withOptions({
  cookieName: 'XSRF-TOKEN',
  headerName: 'X-XSRF-TOKEN'
})
```

---

### Q55. What is CORS and how does it work?

**Answer:**

**CORS** (Cross-Origin Resource Sharing): Browser security mechanism that restricts cross-origin HTTP requests.

```
Browser (https://app.com) → Request to https://api.com
                          ← Server responds with CORS headers
                          → Browser allows/blocks based on headers
```

**Simple vs Preflight requests:**
- **Simple:** GET/POST with standard headers → sent directly
- **Preflight:** PUT/DELETE or custom headers → OPTIONS request first

```
// Server response headers
Access-Control-Allow-Origin: https://app.com  (or * for public APIs)
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400  (cache preflight for 24h)
```

**Common issues:**
- Credentials require specific origin (not `*`)
- Missing headers in preflight response
- Fix: Configure CORS on backend, or use proxy in development

---

## 17. Git & Version Control

### Q56. Explain Git branching strategies. ⭐ *Commonly Asked*

**Answer:**

**Git Flow:**
```
main (production) ←── release/1.0 ←── develop ←── feature/login
                                                  ←── feature/dashboard
                  ←── hotfix/critical-bug
```

**Trunk-Based Development (preferred for CI/CD):**
```
main ←── short-lived feature branches (< 1 day)
     ←── feature flags for incomplete work
```

| Strategy | Best For | Complexity |
|----------|----------|-----------|
| Git Flow | Releases with versions | High |
| Trunk-Based | CI/CD, continuous deploy | Low |
| GitHub Flow | Simple (main + feature branches) | Medium |

**Common commands:**
```bash
git rebase main              # Rebase feature on latest main
git cherry-pick <commit>     # Pick specific commit
git stash                    # Temporarily save work
git reflog                   # Find lost commits
git bisect                   # Binary search for bug introduction
```

---

### Q57. What is the difference between `merge` and `rebase`?

**Answer:**

```
# Merge - creates merge commit, preserves history
git checkout main
git merge feature  →  Creates merge commit M
    A─B─C─M (main)
       ╲ ╱
        D─E (feature)

# Rebase - rewrites history, linear
git checkout feature
git rebase main  →  Replays commits on top
    A─B─C─D'─E' (feature rebased on main)
```

| | Merge | Rebase |
|---|-------|--------|
| History | Non-linear (preserves branches) | Linear (clean) |
| Conflicts | Resolve once | Resolve per commit |
| Shared branches | Safe | NEVER rebase shared/public branches |
| Use case | Integrating feature to main | Updating feature with latest main |

**Golden rule:** Never rebase commits that have been pushed to a shared branch.

---

## 18. CI/CD & DevOps Basics

### Q58. Explain a typical CI/CD pipeline for a frontend app. ⭐ *Commonly Asked*

**Answer:**

```yaml
# GitHub Actions example
name: CI/CD Pipeline
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci                      # Install (frozen lockfile)
      - run: npm run lint                # Lint check
      - run: npm run test -- --coverage  # Unit tests
      - run: npm run build -- --prod     # Production build
      - run: npm run e2e                 # E2E tests
      
      # Deploy on main branch
      - if: github.ref == 'refs/heads/main'
        run: npm run deploy
```

**Pipeline stages:**
1. **Install** — `npm ci` (deterministic from lockfile)
2. **Lint** — ESLint, Prettier check
3. **Test** — Unit tests with coverage threshold
4. **Build** — Production build, check bundle size budget
5. **E2E** — End-to-end tests
6. **Deploy** — Push to CDN/cloud (staging → production)

**Frontend-specific checks:**
- Bundle size budgets (fail if exceeded)
- Lighthouse CI (performance regression)
- Visual regression (Chromatic/Percy)
- Accessibility audits (axe-core)

---

### Q59. What is Docker and how is it used in frontend development?

**Answer:**

```dockerfile
# Multi-stage build for Angular app
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build -- --configuration=production

FROM nginx:alpine AS production
COPY --from=build /app/dist/my-app /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```nginx
# nginx.conf for SPA routing
server {
  listen 80;
  root /usr/share/nginx/html;
  index index.html;

  location / {
    try_files $uri $uri/ /index.html;  # SPA fallback
  }

  location /api/ {
    proxy_pass http://backend:3000/;
  }

  # Cache static assets
  location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff2)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
  }
}
```

---

## 19. PWA & Service Workers

### Q60. What is a Progressive Web App (PWA)?

**Answer:**

**PWA characteristics:**
- **Installable** — Add to home screen, app-like experience
- **Offline capable** — Service Worker caches resources
- **Push notifications** — Re-engage users
- **Responsive** — Works on any device
- **Secure** — HTTPS required

**Required files:**
```json
// manifest.json
{
  "name": "My App",
  "short_name": "App",
  "start_url": "/",
  "display": "standalone",
  "theme_color": "#1976d2",
  "icons": [
    { "src": "/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

```javascript
// Service Worker - cache-first strategy
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open('v1').then(cache =>
      cache.addAll(['/index.html', '/styles.css', '/app.js'])
    )
  );
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then(cached =>
      cached || fetch(event.request).then(response => {
        const clone = response.clone();
        caches.open('v1').then(cache => cache.put(event.request, clone));
        return response;
      })
    )
  );
});
```

---

### Q61. Explain Service Worker caching strategies.

**Answer:**

| Strategy | Description | Use Case |
|----------|-------------|----------|
| Cache First | Check cache, fallback to network | Static assets, fonts |
| Network First | Try network, fallback to cache | API responses, fresh data |
| Stale While Revalidate | Serve from cache, update in background | Semi-dynamic content |
| Network Only | Always network | Auth requests, real-time data |
| Cache Only | Always cache | Offline-first resources |

```javascript
// Stale While Revalidate (best for most cases)
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.open('dynamic').then(cache =>
      cache.match(event.request).then(cached => {
        const fetchPromise = fetch(event.request).then(response => {
          cache.put(event.request, response.clone());
          return response;
        });
        return cached || fetchPromise;
      })
    )
  );
});
```

---

# PART 4: CODING PROBLEMS

## 20. JS Coding Problems

### Q62. Implement deep clone. ⭐ *Commonly Asked*

**Answer:**
```javascript
function deepClone(obj, visited = new WeakMap()) {
  if (obj === null || typeof obj !== 'object') return obj;
  if (visited.has(obj)) return visited.get(obj); // Circular ref

  if (obj instanceof Date) return new Date(obj.getTime());
  if (obj instanceof RegExp) return new RegExp(obj.source, obj.flags);
  if (obj instanceof Map) {
    const clone = new Map();
    visited.set(obj, clone);
    obj.forEach((v, k) => clone.set(deepClone(k, visited), deepClone(v, visited)));
    return clone;
  }
  if (obj instanceof Set) {
    const clone = new Set();
    visited.set(obj, clone);
    obj.forEach(v => clone.add(deepClone(v, visited)));
    return clone;
  }

  const clone = Array.isArray(obj) ? [] : Object.create(Object.getPrototypeOf(obj));
  visited.set(obj, clone);
  for (const key of Reflect.ownKeys(obj)) {
    clone[key] = deepClone(obj[key], visited);
  }
  return clone;
}

// Modern alternative: structuredClone(obj)
```

---

### Q63. Implement `Promise.all` from scratch.

**Answer:**
```javascript
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    const results = [];
    let resolved = 0;
    const total = promises.length;
    if (total === 0) return resolve([]);

    promises.forEach((p, i) => {
      Promise.resolve(p)
        .then(value => {
          results[i] = value;
          if (++resolved === total) resolve(results);
        })
        .catch(reject);
    });
  });
}
```

---

### Q64. Implement flatten array to any depth.

**Answer:**
```javascript
function flatten(arr, depth = Infinity) {
  if (depth <= 0) return arr.slice();
  return arr.reduce((acc, val) =>
    acc.concat(Array.isArray(val) ? flatten(val, depth - 1) : val), []);
}

// Iterative (no recursion limit)
function flattenIterative(arr) {
  const stack = [...arr];
  const result = [];
  while (stack.length) {
    const item = stack.pop();
    Array.isArray(item) ? stack.push(...item) : result.unshift(item);
  }
  return result;
}

// Modern: [1,[2,[3]]].flat(Infinity)
```

---

### Q65. Implement currying.

**Answer:**
```javascript
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) return fn(...args);
    return (...next) => curried(...args, ...next);
  };
}

const add = curry((a, b, c) => a + b + c);
add(1)(2)(3);  // 6
add(1, 2)(3);  // 6
add(1)(2, 3);  // 6
```

---

### Q66. Implement an LRU Cache.

**Answer:**
```javascript
class LRUCache {
  #capacity;
  #cache = new Map();

  constructor(capacity) { this.#capacity = capacity; }

  get(key) {
    if (!this.#cache.has(key)) return -1;
    const value = this.#cache.get(key);
    this.#cache.delete(key);
    this.#cache.set(key, value); // Move to end (most recent)
    return value;
  }

  put(key, value) {
    this.#cache.delete(key);
    if (this.#cache.size >= this.#capacity) {
      this.#cache.delete(this.#cache.keys().next().value); // Remove oldest
    }
    this.#cache.set(key, value);
  }
}
```

---

### Q67. Implement retry with exponential backoff.

**Answer:**
```javascript
async function retry(fn, { maxRetries = 3, baseDelay = 1000, maxDelay = 30000 } = {}) {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn(attempt);
    } catch (error) {
      if (attempt === maxRetries) throw error;
      const delay = Math.min(baseDelay * 2 ** attempt, maxDelay);
      const jitter = delay * (0.5 + Math.random() * 0.5);
      await new Promise(r => setTimeout(r, jitter));
    }
  }
}
```

---

### Q68. Implement `Function.prototype.bind`.

**Answer:**
```javascript
Function.prototype.myBind = function(context, ...boundArgs) {
  const fn = this;
  return function(...callArgs) {
    if (new.target) return new fn(...boundArgs, ...callArgs);
    return fn.apply(context, [...boundArgs, ...callArgs]);
  };
};
```

---

### Q69. Implement a pub/sub event emitter with `once`.

**Answer:**
```javascript
class EventEmitter {
  #events = new Map();

  on(event, fn) {
    if (!this.#events.has(event)) this.#events.set(event, []);
    this.#events.get(event).push({ fn, once: false });
    return () => this.off(event, fn);
  }

  once(event, fn) {
    if (!this.#events.has(event)) this.#events.set(event, []);
    this.#events.get(event).push({ fn, once: true });
  }

  emit(event, ...args) {
    const listeners = this.#events.get(event) || [];
    this.#events.set(event, listeners.filter(l => {
      l.fn(...args);
      return !l.once;
    }));
  }

  off(event, fn) {
    if (!fn) return this.#events.delete(event);
    const ls = this.#events.get(event);
    if (ls) this.#events.set(event, ls.filter(l => l.fn !== fn));
  }
}
```

---

### Q70. Implement `Array.prototype.reduce`.

**Answer:**
```javascript
Array.prototype.myReduce = function(cb, initial) {
  let acc = initial !== undefined ? initial : this[0];
  let start = initial !== undefined ? 0 : 1;
  if (this.length === 0 && initial === undefined) throw new TypeError('Reduce of empty array');
  for (let i = start; i < this.length; i++) {
    acc = cb(acc, this[i], i, this);
  }
  return acc;
};
```

---

### Q71. Implement a function that detects circular references.

**Answer:**
```javascript
function hasCircular(obj) {
  const seen = new WeakSet();
  return (function detect(val) {
    if (val === null || typeof val !== 'object') return false;
    if (seen.has(val)) return true;
    seen.add(val);
    return Object.values(val).some(detect);
  })(obj);
}

// Safe stringify
function safeStringify(obj) {
  const seen = new WeakSet();
  return JSON.stringify(obj, (_, value) => {
    if (typeof value === 'object' && value !== null) {
      if (seen.has(value)) return '[Circular]';
      seen.add(value);
    }
    return value;
  });
}
```

---

# PART 5: SYSTEM DESIGN & BEHAVIORAL

## 21. System Design for Frontend

### Q72. Design a real-time chat application frontend. ⭐ *Commonly Asked*

**Answer:**

```
Architecture:
┌───────────────────────────────────────────┐
│              Angular App                   │
├───────────┬───────────┬──────────────────┤
│  Chat UI  │ User List │  Settings        │
├───────────┴───────────┴──────────────────┤
│         State Management (NgRx/Signals)   │
├───────────────────────────────────────────┤
│  WebSocket Service    │  REST API Service │
└──────────┬────────────┴────────┬─────────┘
           │                     │
      WebSocket              REST API
      (real-time)            (history, auth)
```

**Key decisions:**
- **WebSocket** for real-time messages, typing indicators, presence
- **Virtual scrolling** for message history (thousands of messages)
- **IndexedDB** for offline message queue
- **Optimistic updates** — show message immediately, retry on failure
- **Message pagination** — load older messages on scroll up
- **Reconnection** — exponential backoff with jitter

**State structure:**
```typescript
interface ChatState {
  rooms: Map<string, Room>;
  messages: Map<string, Message[]>; // roomId → messages
  activeRoomId: string;
  onlineUsers: Set<string>;
  typingUsers: Map<string, string[]>; // roomId → userIds
  connectionStatus: 'connected' | 'reconnecting' | 'offline';
}
```

---

### Q73. Design a dashboard with 20+ widgets loading data from different APIs.

**Answer:**

**Challenges:** Multiple API calls, varying refresh rates, error isolation, layout flexibility.

**Architecture:**
```typescript
// Each widget is independent
@Component({
  selector: 'app-widget-container',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    @defer (on viewport) {
      <ng-container [ngComponentOutlet]="widgetComponent"></ng-container>
    } @placeholder {
      <app-widget-skeleton></app-widget-skeleton>
    } @error {
      <app-widget-error [widgetId]="config.id" (retry)="reload()"></app-widget-error>
    }
  `
})
```

**Key patterns:**
- **Error boundaries** — One widget failing doesn't crash others
- **Independent loading** — Each widget fetches its own data
- **Lazy rendering** — `@defer (on viewport)` for below-fold widgets
- **Caching** — `shareReplay` per API, stale-while-revalidate
- **Web Workers** — Heavy data transformations off main thread
- **Grid layout** — CSS Grid with user-configurable positions
- **Refresh strategies** — Per-widget configurable polling interval

---

### Q74. Design a file upload system supporting large files (1GB+).

**Answer:**

```typescript
// Chunked upload with resume support
class ChunkUploader {
  private chunkSize = 5 * 1024 * 1024; // 5MB chunks

  async upload(file: File, onProgress: (pct: number) => void): Promise<string> {
    const uploadId = await this.initUpload(file.name, file.size);
    const totalChunks = Math.ceil(file.size / this.chunkSize);
    const uploaded = await this.getUploadedChunks(uploadId);

    for (let i = 0; i < totalChunks; i++) {
      if (uploaded.includes(i)) continue; // Resume support

      const chunk = file.slice(i * this.chunkSize, (i + 1) * this.chunkSize);
      await this.uploadChunk(uploadId, i, chunk);
      onProgress(((i + 1) / totalChunks) * 100);
    }

    return this.completeUpload(uploadId);
  }
}
```

**Key features:**
- **Chunked uploads** — Break file into 5MB pieces
- **Resume support** — Track uploaded chunks, skip on retry
- **Web Worker** — Hash computation (SHA-256) off main thread
- **Progress tracking** — Per-chunk progress with overall percentage
- **Concurrent chunks** — Upload 3 chunks in parallel
- **Drag & drop zone** — HTML5 Drag and Drop API
- **File validation** — Type, size, dimensions (for images)
- **Cancel support** — AbortController for in-flight requests

---

## 22. Behavioral Questions

### Q75. Tell me about a time you had to make a difficult technical decision.

**Expected STAR answer:**
- **S:** "Our team debated between migrating to micro-frontends vs monolith refactoring"
- **T:** "I needed to evaluate both approaches and present a recommendation"
- **A:** "Created proof-of-concepts for both, measured build times, DX, and deployment complexity. Presented tradeoffs to stakeholders"
- **R:** "We chose module federation. Deploy time dropped from 20min to 3min per team. Teams became autonomous"

---

### Q76. How do you handle disagreements with team members on technical approaches?

**Key points:**
- Listen first, understand their perspective
- Focus on data/metrics, not opinions
- Create small POCs to compare approaches
- Consider: "What if we're both right for different reasons?"
- Disagree and commit — once a decision is made, support it
- Document decisions in ADRs (Architecture Decision Records)

---

### Q77. How do you stay up to date with frontend technologies?

**Good answers include:**
- Follow Angular/React/Web platform blogs and RFCs
- Read release notes for major updates
- Build side projects to experiment
- Attend conferences (ng-conf, etc.) / watch recordings
- Contribute to open source
- Internal tech talks and knowledge sharing
- Twitter/X, dev.to, newsletters (JavaScript Weekly, etc.)

---

### Q78. Describe a time you mentored a junior developer.

**Expected elements:**
- Specific example with context
- Your approach (pairing, code reviews, documentation)
- Balance between giving answers and guiding to solutions
- Measurable improvement (they shipped features independently, code quality improved)
- What you learned from mentoring

---

### Q79. How do you estimate and break down work for a sprint?

**Answer:**
- Break into vertical slices (not horizontal: "frontend" then "backend")
- Each ticket: clear acceptance criteria, testable
- Estimate in story points (relative sizing)
- Consider: development + testing + code review + edge cases
- Flag unknowns as spikes (time-boxed research)
- Add buffer for integration, rework, meetings

---

### Q80. Tell me about a production bug you resolved under pressure.

**Expected structure:**
- **Detection:** How you found out (monitoring, user report, alert)
- **Triage:** How you assessed severity and impact
- **Diagnosis:** Tools used (logs, DevTools, reproduction steps)
- **Fix:** What you did (hotfix, rollback, feature flag)
- **Prevention:** What you put in place to prevent recurrence (test, monitor, process)

---

## Quick Reference: Tech Stack Summary

| Category | Tools to Know |
|----------|---------------|
| **Language** | JavaScript (ES6+), TypeScript |
| **Framework** | Angular (primary), React (awareness) |
| **State** | NgRx, Signals, RxJS services |
| **Testing** | Jasmine, Jest, Karma, Cypress, Playwright |
| **Build** | Angular CLI, Webpack, esbuild, Vite |
| **CSS** | SCSS/Sass, CSS Grid, Flexbox, Tailwind |
| **API** | REST, GraphQL, WebSockets |
| **Auth** | OAuth 2.0, JWT, OIDC, PKCE |
| **CI/CD** | GitHub Actions, Jenkins, Azure DevOps |
| **Containers** | Docker, Nginx |
| **Cloud** | AWS (S3, CloudFront), Azure (Blob, CDN) |
| **Monitoring** | Sentry, Datadog, Lighthouse |
| **Version Control** | Git (rebase, cherry-pick, bisect) |
| **Package Mgmt** | npm, yarn, pnpm |
| **Accessibility** | WCAG 2.1, ARIA, axe-core |

---

## Bonus: Quick-Fire Questions

| Question | Key Answer |
|----------|-----------|
| `for...in` vs `for...of`? | `in` = enumerable keys (objects), `of` = iterable values (arrays) |
| `Object.freeze` vs `Object.seal`? | Freeze: no modifications at all. Seal: no add/remove, CAN modify |
| What is `globalThis`? | Universal reference to global object (browser, Node, Worker) |
| `typeof NaN`? | `'number'` |
| `0.1 + 0.2 === 0.3`? | `false` (floating point precision: 0.30000000000000004) |
| What is tree-shaking? | Dead code removal via static ES module analysis |
| What is a polyfill vs a transpiler? | Polyfill adds missing APIs at runtime; transpiler converts syntax at build time |
| `event.preventDefault()` vs `return false`? | preventDefault stops default action only; return false (jQuery) also stops propagation |
| What's a pure function? | Same input → same output, no side effects |
| Explain CSP header | Content-Security-Policy: whitelist allowed script/style/img sources to prevent XSS |

---

*Document covers: JavaScript, TypeScript, HTML5, CSS, Web Performance, Security, Testing, Build Tools, APIs, CI/CD, PWA, Git, System Design, and Behavioral questions for a senior frontend developer interview.*
