# JavaScript & TypeScript Interview Questions — Deep Dive

The bread-and-butter of every frontend interview. JS fundamentals separate people who
"use Angular" from people who **understand the platform**. TypeScript questions test
whether you write safe, maintainable code.

---

## Part A — JavaScript core

### Q1. `var` vs `let` vs `const`?
| | `var` | `let` | `const` |
|--|------|------|--------|
| Scope | function | block | block |
| Hoisting | hoisted, `undefined` | hoisted, **TDZ** | hoisted, **TDZ** |
| Re-assign | yes | yes | **no** |
| Re-declare | yes | no | no |

```js
console.log(a); // undefined (var hoisted)
var a = 1;
console.log(b); // ReferenceError (TDZ)
let b = 2;
```
**TDZ (Temporal Dead Zone):** the span between entering scope and the declaration where
`let`/`const` exist but can't be accessed.
**`const` caveat:** binding is constant, **not the value** — `const arr = []; arr.push(1)` is fine.

---

### Q2. Explain hoisting.
JS moves **declarations** to the top of their scope during compilation.
`var` and function **declarations** are hoisted (functions fully, `var` as
`undefined`). `let`/`const`/class are hoisted but in the TDZ. **Function expressions**
and arrow functions are not callable before assignment.
```js
foo();              // works — declaration hoisted
function foo() {}
bar();              // TypeError: bar is not a function
var bar = () => {};
```

---

### Q3. What is a closure? (the #1 JS question)
A **closure** is a function that **remembers the variables of the scope where it was
created**, even after that scope has returned.
```js
function counter() {
  let count = 0;                 // private state
  return () => ++count;          // closure over `count`
}
const inc = counter();
inc(); // 1
inc(); // 2
```
**Why it matters / uses:** data privacy (module pattern), function factories,
memoization, event handlers, `setTimeout` callbacks.

**Real-world example (debounce — you've shipped this):** a search box that waits until
the user stops typing. The `timer` variable lives in the closure between keystrokes:
```js
function debounce(fn, ms) {
  let timer;                      // remembered across calls via closure
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), ms);
  };
}
const onSearch = debounce(q => api.search(q), 300);
```
The returned function "remembers" `timer` and `fn` even though `debounce` already
returned — that's the closure doing the work behind your typeahead.

**Classic gotcha:**
```js
for (var i = 0; i < 3; i++) setTimeout(() => console.log(i)); // 3,3,3
for (let i = 0; i < 3; i++) setTimeout(() => console.log(i)); // 0,1,2
```
`var` shares one binding; `let` creates a new binding per iteration.

---

### Q4. Explain the event loop, call stack, microtasks vs macrotasks.
JS is **single-threaded**. The **call stack** runs synchronous code. Async callbacks
wait in queues; the **event loop** moves them to the stack when it's empty.
- **Microtasks:** Promises (`.then`), `queueMicrotask`, `MutationObserver`. Drained
  **fully** after each task, **before** rendering.
- **Macrotasks:** `setTimeout`, `setInterval`, I/O, UI events.

```js
console.log('1');
setTimeout(() => console.log('2'));        // macrotask
Promise.resolve().then(() => console.log('3')); // microtask
console.log('4');
// Output: 1, 4, 3, 2  (sync → microtasks → macrotasks)
```
**Why interviewers ask:** predicts UI jank and async bugs.

**Real-world example:** in Angular, a `Promise.resolve().then(...)` callback runs
*before* the browser paints, but a `setTimeout(...)` runs *after*. That's why an
`ExpressionChangedAfterItHasBeenCheckedError` fix sometimes uses `setTimeout` (defer to
the next macrotask, after Angular finishes its check) rather than a microtask. Knowing
micro-vs-macro ordering is how you reason about "why did my value update one tick late?"

---

### Q5. How does `this` work? `call` / `apply` / `bind`?
`this` is determined by **how a function is called**:
1. **Method call** `obj.fn()` → `this = obj`.
2. **Plain call** `fn()` → `this = undefined` (strict) / `window` (sloppy).
3. **`new Fn()`** → `this` = the new object.
4. **Arrow function** → **no own `this`**; inherits from enclosing scope (lexical).
5. **Explicit** → `call`/`apply`/`bind` set `this`.

```js
function greet(greeting) { return `${greeting}, ${this.name}`; }
const user = { name: 'Ann' };
greet.call(user, 'Hi');     // "Hi, Ann"  (args list)
greet.apply(user, ['Hi']);  // "Hi, Ann"  (args array)
const bound = greet.bind(user); bound('Hi'); // returns new fn with fixed this
```
**Why arrow functions matter in Angular/React:** class methods used as callbacks keep
`this` without `.bind`.

---

### Q6. Prototypes and prototypal inheritance.
Every object has an internal `[[Prototype]]` link. Property lookups walk the
**prototype chain** until found or `null`.
```js
function Animal(name) { this.name = name; }
Animal.prototype.speak = function () { return `${this.name} makes a sound`; };
const dog = new Animal('Rex');
dog.speak();                  // found on prototype
Object.getPrototypeOf(dog) === Animal.prototype; // true
```
ES6 `class` is **syntactic sugar** over this prototype mechanism.

---

### Q7. `==` vs `===`?
`==` does **type coercion**; `===` checks value **and** type. Always prefer `===`.
```js
0 == '';        // true  (coercion)
0 === '';       // false
null == undefined; // true
null === undefined; // false
NaN === NaN;    // false (use Number.isNaN)
```

---

### Q8. Shallow vs deep copy?
```js
const o = { a: 1, nested: { b: 2 } };
const shallow = { ...o };               // nested still shared
const deep = structuredClone(o);        // fully independent (modern)
const deepJson = JSON.parse(JSON.stringify(o)); // loses Dates/functions/undefined
```

---

### Q9. Promises: `all` / `allSettled` / `race` / `any`?
- **`Promise.all`** — resolves when all resolve; **rejects fast** if any rejects.
- **`Promise.allSettled`** — waits for all, returns status of each (never short-circuits).
- **`Promise.race`** — settles with the **first** to settle (resolve or reject).
- **`Promise.any`** — resolves with the **first fulfilled**; rejects only if all reject.
```js
const [users, posts] = await Promise.all([getUsers(), getPosts()]); // parallel
```

---

### Q10. `async`/`await` — how does it relate to Promises? Error handling?
`async` functions **always return a Promise**; `await` pauses until the awaited
Promise settles (it's syntactic sugar over `.then`). Handle errors with `try/catch`.
```js
async function load() {
  try {
    const res = await fetch('/api');
    if (!res.ok) throw new Error(res.statusText);
    return await res.json();
  } catch (e) { console.error(e); throw e; }
}
```
**Gotcha:** sequential `await`s run serially — parallelize independent calls with
`Promise.all`.

---

### Q11. `map` / `filter` / `reduce` — explain reduce.
```js
const nums = [1, 2, 3, 4];
nums.map(n => n * 2);            // [2,4,6,8]
nums.filter(n => n % 2 === 0);  // [2,4]
nums.reduce((acc, n) => acc + n, 0); // 10
// reduce can build anything: group by, flatten, count
const byParity = nums.reduce((acc, n) => {
  (acc[n % 2 ? 'odd' : 'even'] ??= []).push(n); return acc;
}, {});
```

---

### Q12. Destructuring, spread, rest, default params.
```js
const { name, age = 18 } = user;          // object destructure + default
const [first, ...rest] = [1, 2, 3];        // rest
const merged = { ...a, ...b };             // spread
function sum(...nums) { return nums.reduce((a, b) => a + b, 0); } // rest params
```

---

### Q13. Debounce vs throttle?
- **Debounce** — run after activity **stops** for N ms (search input).
- **Throttle** — run at most **once per N ms** (scroll/resize).
```js
function debounce(fn, ms) {
  let t; return (...args) => { clearTimeout(t); t = setTimeout(() => fn(...args), ms); };
}
```

---

### Q14. `null` vs `undefined`? Optional chaining / nullish coalescing.
- `undefined` — declared but not assigned.
- `null` — intentional "no value".
```js
user?.address?.city            // optional chaining — no crash if missing
const port = config.port ?? 3000; // ?? only falls back on null/undefined (not 0/'')
```

---

### Q15. Other common JS topics (be ready)
- **Generators** (`function*`, `yield`) — pausable functions / lazy iteration.
- **Symbols** — unique keys; `Symbol.iterator` makes objects iterable.
- **`Map`/`Set`/`WeakMap`/`WeakSet`** — `Map` keeps insertion order, any key type;
  `WeakMap` allows GC of keys.
- **Modules** — `import`/`export`, tree-shaking; vs CommonJS `require`.
- **Currying** — `f(a)(b)(c)`.
- **Immutability** — `Object.freeze`, spread to copy.
- **`event.preventDefault()` vs `stopPropagation()`** — cancel default vs stop bubbling.

---

## Part B — TypeScript

### Q16. Why TypeScript? What does it add over JS?
**Static typing** catches errors at **compile time**, improves IDE autocomplete &
refactoring, documents intent, and scales large codebases. Compiles to plain JS.

---

### Q17. `interface` vs `type` — the classic.
Both describe shapes. Differences:
- **`interface`** can be **declaration-merged** and `extends`; idiomatic for object/API
  shapes and public contracts.
- **`type`** can express **unions, intersections, tuples, mapped/conditional types,
  primitives** — more flexible.
```ts
interface User { id: number; name: string; }
interface User { email: string; }      // merges
type ID = string | number;             // union — interface can't
type Point = { x: number } & { y: number }; // intersection
```
**Rule of thumb:** interfaces for objects/contracts, types for unions/utilities.

---

### Q18. Explain generics.
Type **parameters** for reusable, type-safe code.
```ts
function identity<T>(value: T): T { return value; }
function first<T>(arr: T[]): T | undefined { return arr[0]; }

interface ApiResponse<T> { data: T; status: number; }
const res: ApiResponse<User[]> = await api.get('/users');

// constraints
function longest<T extends { length: number }>(a: T, b: T): T {
  return a.length >= b.length ? a : b;
}
```

---

### Q19. Utility types — know these cold.
```ts
interface User { id: number; name: string; email: string; }

Partial<User>            // all optional
Required<User>           // all required
Readonly<User>           // immutable
Pick<User, 'id'|'name'>  // subset
Omit<User, 'email'>      // exclude
Record<string, User>     // dictionary { [k: string]: User }
Exclude<'a'|'b'|'c','a'> // 'b' | 'c'
Extract<'a'|'b', 'a'>    // 'a'
NonNullable<string|null> // string
ReturnType<typeof fn>    // infer return type
Parameters<typeof fn>    // tuple of param types
Awaited<Promise<User>>   // User (unwrap promise)
```
**Real example:** a DTO from a form often is `Omit<User, 'id'>` (server assigns id).

---

### Q20. `any` vs `unknown` vs `never`?
- **`any`** — disables type checking (avoid; defeats TS).
- **`unknown`** — type-safe `any`; you **must narrow** before use.
- **`never`** — value that never occurs (exhaustive checks, functions that throw).
```ts
function handle(x: unknown) {
  // x.toFixed(); // error — must narrow
  if (typeof x === 'number') x.toFixed(); // ok
}
function fail(msg: string): never { throw new Error(msg); }
```

---

### Q21. Type narrowing / type guards.
```ts
typeof x === 'string'                 // typeof guard
x instanceof Date                     // instanceof guard
'name' in obj                         // in guard
function isUser(x: any): x is User {  // custom type predicate
  return x && typeof x.id === 'number';
}
// discriminated union — the cleanest pattern
type Shape =
  | { kind: 'circle'; r: number }
  | { kind: 'square'; side: number };
function area(s: Shape) {
  switch (s.kind) {
    case 'circle': return Math.PI * s.r ** 2;
    case 'square': return s.side ** 2;
    default: const _exhaustive: never = s; return _exhaustive; // compile-time safety
  }
}
```

---

### Q22. Enums vs union types vs `as const`?
```ts
enum Role { Admin = 'ADMIN', User = 'USER' }   // runtime object generated
type Role2 = 'ADMIN' | 'USER';                  // erased, lighter, often preferred
const ROLES = ['ADMIN', 'USER'] as const;       // readonly tuple
type Role3 = typeof ROLES[number];              // 'ADMIN' | 'USER'
```
Many teams prefer **union types / `as const`** over enums (no runtime cost, better
tree-shaking).

---

### Q23. What does `as const` do?
Makes a literal **deeply readonly** and narrows to **literal types** instead of widened
ones (`'ADMIN'` instead of `string`). Great for config objects and action types.

---

### Q24. Mapped & conditional types (advanced).
```ts
type Optional<T> = { [K in keyof T]?: T[K] };           // mapped
type NonFunctionKeys<T> = { [K in keyof T]: T[K] extends Function ? never : K }[keyof T];
type IsString<T> = T extends string ? true : false;     // conditional
type Unwrap<T> = T extends Promise<infer U> ? U : T;     // infer
```

---

### Q25. Decorators (Angular relies on these).
Functions that annotate/modify classes, methods, properties, params. Angular uses them
for metadata: `@Component`, `@Injectable`, `@Input`, `@Output`.
```ts
function Log(target: any, key: string, desc: PropertyDescriptor) {
  const orig = desc.value;
  desc.value = function (...args: any[]) {
    console.log(`calling ${key}`); return orig.apply(this, args);
  };
}
class Service { @Log greet() { return 'hi'; } }
```

---

### Q26. `tsconfig` settings interviewers may ask about.
- **`strict`** — enables all strict checks (recommended).
- **`strictNullChecks`** — `null`/`undefined` not assignable unless declared.
- **`noImplicitAny`** — forbid implicit `any`.
- **`target`** / **`module`** — JS version / module system output.
- **`paths`** — path aliases (`@app/*`).

---

### Q27. Structural typing ("duck typing").
TS types are **structural**, not nominal — compatibility is by **shape**, not name.
```ts
interface Named { name: string; }
function greet(n: Named) {}
greet({ name: 'Ann', extra: 1 }); // OK — has required shape
```

---

### Q28. How do you type an async API call end-to-end?
```ts
interface User { id: number; name: string; }
async function getUser(id: number): Promise<User> {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error('Failed');
  return res.json() as Promise<User>; // (or validate with zod for real safety)
}
```
**Senior add-on:** runtime validation (e.g., **zod**) because `res.json()` is `any` —
TS can't guarantee the server's shape at runtime.

---

## More questions (with real-world examples)

### Q29. Deep copy vs shallow copy — and how do you actually copy an object?
- **Shallow** (`{...obj}`, `Object.assign`, `array.slice()`) copies the top level but
  **shares nested references**.
- **Deep** copies everything recursively.
```js
const a = { user: { name: 'Ann' } };
const shallow = { ...a };
shallow.user.name = 'Bob';   // also changes a.user.name (shared reference)
const deep = structuredClone(a); // modern, built-in deep copy
```
**Real-world:** this is the root cause of countless "state mutated somewhere else" bugs.
In NgRx/Redux you must return **new** references; a shallow spread of a nested object
silently shares the inner object and breaks `OnPush`/memoization.

### Q30. `==` vs `===` (and `null == undefined`).
`===` checks value **and** type (no coercion). `==` coerces, leading to surprises like
`0 == ''` → `true`. **Rule:** always use `===`, with one common exception: `x == null`
is a handy idiom that's `true` for **both** `null` and `undefined`.

### Q31. What is destructuring with defaults? (you use it daily)
```js
const { name = 'Guest' } = user;
const [first, ...rest] = items;
function f({ page = 1, size = 20 } = {}) {}  // named, defaulted params
```
**Real-world:** the `function f({...} = {})` pattern builds a clean options object for a
service method without a long positional argument list.

### Q32. `Promise.all` vs `allSettled` vs `race` (real choices).
- **`all`** — wait for everything; **fails fast** if any rejects (load user + roles +
  settings together; if any fails the page can't render).
- **`allSettled`** — wait for everything, **never short-circuits**; inspect each result
  (fire 5 independent widgets, render the ones that succeed).
- **`race`** — first to settle wins (request vs a timeout promise).
```js
const [user, roles] = await Promise.all([getUser(), getRoles()]); // parallel, fast
```

### Q33. `async/await` error handling — the gotcha.
`await` only catches if you `try/catch`. A common bug is forgetting to `await`, so the
`try/catch` never sees the rejection:
```js
try { await save(); } catch (e) { showError(e); }  // ✅
try { save(); }       catch (e) { /* never runs */ } // ❌ missing await
```

### Q34. What is currying and partial application?
**Currying** turns `f(a, b, c)` into `f(a)(b)(c)` — a chain of single-argument
functions. **Partial application** fixes *some* arguments now and the rest later.
```js
const add = a => b => a + b;       // curried
const add5 = add(5);               // partial — remembers a=5 via closure
add5(10);                          // 15
```
**Real-world:** building reusable helpers — `const withBase = url => path => fetch(base + path)`
gives you a pre-configured API caller. Libraries like Lodash/Ramda lean on this heavily.

### Q35. Generators and iterators — what problem do they solve?
A **generator** (`function*`) can **pause** (`yield`) and resume, producing values lazily
instead of all at once. Anything with a `[Symbol.iterator]` is **iterable** (`for...of`,
spread).
```js
function* ids() { let i = 1; while (true) yield i++; }  // infinite, but lazy
const gen = ids();
gen.next().value; // 1
gen.next().value; // 2
```
**Real-world:** streaming/paginating huge datasets without loading everything into
memory, and custom iteration (RxJS and Redux-Saga build on these ideas).

### Q36. `Map`/`Set` vs plain object/array — when to use which?
- **`Map`** — keys of **any type** (objects, not just strings), preserves insertion
  order, `.size`, easy iteration. Use for true key→value dictionaries, especially with
  non-string keys.
- **`Set`** — unique values; O(1) membership checks and easy de-duplication.
```js
const unique = [...new Set([1, 1, 2, 3])]; // [1,2,3] — classic one-liner
const seen = new Map();                    // object keys allowed
```
**Real-world:** de-duping IDs, counting occurrences, or caching by object reference —
all cleaner with `Map`/`Set` than abusing `{}`.

### Q37. `WeakMap`/`WeakSet` — why do they exist?
Keys are held **weakly**: if nothing else references the key object, it can be
garbage-collected (and its entry disappears). Not iterable.
**Real-world:** attaching private/cache data to objects (e.g., DOM nodes) **without
leaking memory** — when the node is removed, its metadata is auto-collected. This is how
you avoid the leaks that plague plain-`Map` caches.

### Q38. ESM vs CommonJS modules?
- **ESM** (`import`/`export`) — the standard; **static**, tree-shakable, async-friendly,
  used in browsers and modern Node.
- **CommonJS** (`require`/`module.exports`) — older Node format; **dynamic**, synchronous.
```js
import { thing } from './m.js';   // ESM (what Angular/Vite use)
const thing = require('./m');     // CommonJS (legacy Node)
```
**Why it matters:** tree-shaking (dead-code elimination) only works well with ESM's
static structure — directly affects your bundle size.

### Q39. Event delegation — what and why?
Attach **one** listener to a parent and use `event.target` to handle events from many
children, instead of one listener per child.
```js
list.addEventListener('click', e => {
  const li = e.target.closest('li');
  if (li) select(li.dataset.id);
});
```
**Real-world:** a list of 1,000 rows needs **one** listener, not 1,000 — less memory, and
it still works for rows added later (dynamic content). Frameworks do this internally.

### Q40. TypeScript: what does the `satisfies` operator do?
It checks a value matches a type **without widening** it — you keep the precise literal
types *and* get validation.
```ts
const config = {
  host: 'localhost',
  port: 8080,
} satisfies Record<string, string | number>;
config.port.toFixed(); // still known to be number, not string|number
```
**Real-world:** validating a config/theme object against a contract while keeping exact
key/value types for autocomplete — a modern TS feature interviewers like to see.

### Q41. TypeScript: `interface` declaration merging & module augmentation?
Two `interface`s with the same name **merge** into one. You can also **augment** library
types from outside.
```ts
interface Window { myApp: { version: string }; }  // adds to the global Window type
```
**Real-world:** adding a typed property to `Window`, or extending a third-party library's
types without forking it. (This is a key reason to pick `interface` over `type` for
public/extensible contracts.)

---

### Rapid-fire
- **`readonly` vs `const`?** `const` for variable bindings; `readonly` for properties.
- **Index signatures?** `{ [key: string]: number }`.
- **Function overloads?** Multiple signatures, one implementation.
- **`?.` vs `!`?** Optional chaining vs non-null assertion (`x!` = "trust me, not null").
- **`keyof`?** Union of an object's keys: `keyof User` → `'id'|'name'|'email'`.
- **`typeof` (type context)?** Get the type of a value/variable.
- **Why avoid `any`?** It silently turns off type safety and spreads.

---

### Likely deep-dive chain
> "What's a closure?" → "Show the `var` loop bug" → "How does the event loop order
> micro vs macro tasks?" → "`interface` vs `type`?" → "When `unknown` over `any`?"
