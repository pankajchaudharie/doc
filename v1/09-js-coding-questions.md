# JavaScript Coding Questions — Most Asked in Companies

Hands-on problems that show up again and again in real interviews (TCS, Infosys, Wipro,
product companies, and frontend rounds). Each has a **clear solution**, the **idea
behind it**, the **time/space complexity**, and **what the interviewer is really
testing**. Type these out by hand — muscle memory matters in a live round.

> How to use: cover the solution, attempt it, then compare. Say your approach out loud
> ("I'll use a hash map to get O(n)…") — interviewers score your *reasoning*, not just a
> working answer.

---

## Part A — Strings (the most common category)

### Q1. Reverse a string
```js
const reverse = str => str.split('').reverse().join('');
// Without built-ins (interviewers sometimes ask this):
function reverseManual(str) {
  let out = '';
  for (let i = str.length - 1; i >= 0; i--) out += str[i];
  return out;
}
```
**Idea:** turn into array → reverse → join. **Complexity:** O(n) time, O(n) space.
**Why asked:** warm-up; the follow-up "do it without `.reverse()`" tests loop basics.

### Q2. Check if a string is a palindrome
```js
function isPalindrome(str) {
  const clean = str.toLowerCase().replace(/[^a-z0-9]/g, ''); // ignore spaces/punctuation
  let i = 0, j = clean.length - 1;
  while (i < j) if (clean[i++] !== clean[j--]) return false;
  return true;
}
isPalindrome('A man, a plan, a canal: Panama'); // true
```
**Idea:** two pointers from both ends. **Complexity:** O(n) time, O(1) extra space.
**Why asked:** the two-pointer pattern is reused in dozens of problems.

### Q3. Check if two strings are anagrams
```js
function isAnagram(a, b) {
  if (a.length !== b.length) return false;
  const count = {};
  for (const ch of a) count[ch] = (count[ch] || 0) + 1;
  for (const ch of b) {
    if (!count[ch]) return false;
    count[ch]--;
  }
  return true;
}
isAnagram('listen', 'silent'); // true
```
**Idea:** count characters in one, decrement with the other. **Complexity:** O(n).
**Why asked:** tests the **frequency-map** technique (the single most reused trick).

### Q4. First non-repeating character
```js
function firstUnique(str) {
  const count = {};
  for (const ch of str) count[ch] = (count[ch] || 0) + 1;
  for (const ch of str) if (count[ch] === 1) return ch;
  return null;
}
firstUnique('swiss'); // 'w'
```
**Idea:** two passes — build counts, then find the first with count 1. **Complexity:** O(n).

### Q5. Count occurrences of each character (frequency map)
```js
const frequency = str =>
  [...str].reduce((acc, ch) => (acc[ch] = (acc[ch] || 0) + 1, acc), {});
frequency('hello'); // { h:1, e:1, l:2, o:1 }
```
**Why asked:** this exact pattern underlies anagrams, unique chars, and "most frequent".

### Q6. Most frequent element / character
```js
function mostFrequent(arr) {
  const count = {};
  let best = arr[0], max = 0;
  for (const x of arr) {
    count[x] = (count[x] || 0) + 1;
    if (count[x] > max) { max = count[x]; best = x; }
  }
  return best;
}
mostFrequent([1, 3, 3, 2, 3, 1]); // 3
```

### Q7. Reverse words in a sentence
```js
const reverseWords = s => s.trim().split(/\s+/).reverse().join(' ');
reverseWords('the sky is blue'); // 'blue is sky the'
```
**Watch-out:** `split(/\s+/)` collapses multiple spaces — mention you handle that.

### Q8. Capitalize the first letter of each word (title case)
```js
const titleCase = s =>
  s.toLowerCase().replace(/\b\w/g, c => c.toUpperCase());
titleCase('hello world'); // 'Hello World'
```

### Q9. Check if a string contains balanced brackets `()[]{}`
```js
function isBalanced(str) {
  const pairs = { ')': '(', ']': '[', '}': '{' };
  const stack = [];
  for (const ch of str) {
    if (ch === '(' || ch === '[' || ch === '{') stack.push(ch);
    else if (ch in pairs) {
      if (stack.pop() !== pairs[ch]) return false;
    }
  }
  return stack.length === 0;
}
isBalanced('{[()]}'); // true
isBalanced('{[(])}'); // false
```
**Idea:** a **stack** — push openers, pop and match on closers. **Why asked:** classic
stack question; comes up in editors, compilers, JSON validators.

---

## Part B — Arrays

### Q10. Find the largest / smallest without `Math.max`
```js
function maxOf(arr) {
  let max = arr[0];
  for (const x of arr) if (x > max) max = x;
  return max;
}
```
**Why asked:** tests a clean single-pass loop and edge handling (empty array).

### Q11. Remove duplicates
```js
const unique = arr => [...new Set(arr)];
// preserve order without Set:
const uniqueManual = arr => arr.filter((x, i) => arr.indexOf(x) === i);
```
**Note:** `Set` is O(n); `indexOf` inside `filter` is O(n²) — mention the trade-off.

### Q12. Second largest element
```js
function secondLargest(arr) {
  let first = -Infinity, second = -Infinity;
  for (const x of arr) {
    if (x > first) { second = first; first = x; }
    else if (x > second && x !== first) second = x;
  }
  return second;
}
secondLargest([10, 5, 8, 10, 7]); // 8
```
**Why asked:** "find Nth largest in one pass" is a frequent follow-up; don't just sort.

### Q13. Two Sum — find a pair that adds to a target
```js
function twoSum(nums, target) {
  const seen = new Map();           // value -> index
  for (let i = 0; i < nums.length; i++) {
    const need = target - nums[i];
    if (seen.has(need)) return [seen.get(need), i];
    seen.set(nums[i], i);
  }
  return [];
}
twoSum([2, 7, 11, 15], 9); // [0, 1]
```
**Idea:** hash map → O(n) instead of the O(n²) brute force. **Why asked:** THE most
common array question; the naive answer is two loops, the good answer is a map.

### Q14. Move all zeros to the end (in place)
```js
function moveZeros(arr) {
  let insert = 0;
  for (let i = 0; i < arr.length; i++)
    if (arr[i] !== 0) arr[insert++] = arr[i];
  while (insert < arr.length) arr[insert++] = 0;
  return arr;
}
moveZeros([0, 1, 0, 3, 12]); // [1, 3, 12, 0, 0]
```
**Idea:** a write pointer (`insert`). **Complexity:** O(n) time, O(1) space.

### Q15. Max subarray sum (Kadane's algorithm)
```js
function maxSubArray(nums) {
  let best = nums[0], current = nums[0];
  for (let i = 1; i < nums.length; i++) {
    current = Math.max(nums[i], current + nums[i]);
    best = Math.max(best, current);
  }
  return best;
}
maxSubArray([-2, 1, -3, 4, -1, 2, 1, -5, 4]); // 6  ([4,-1,2,1])
```
**Why asked:** a famous DP-lite problem; shows you can beat the O(n²) brute force.

### Q16. Flatten a nested array
```js
const flatten = arr =>
  arr.reduce((acc, x) => acc.concat(Array.isArray(x) ? flatten(x) : x), []);
flatten([1, [2, [3, [4]]]]); // [1, 2, 3, 4]
// Modern built-in: arr.flat(Infinity)
```
**Why asked:** tests **recursion** and knowing the modern `.flat(Infinity)`.

### Q17. Chunk an array into groups of n
```js
function chunk(arr, size) {
  const out = [];
  for (let i = 0; i < arr.length; i += size) out.push(arr.slice(i, i + size));
  return out;
}
chunk([1, 2, 3, 4, 5], 2); // [[1,2],[3,4],[5]]
```
**Real-world:** paginating results or building a grid layout.

### Q18. Group an array of objects by a key
```js
function groupBy(arr, key) {
  return arr.reduce((acc, item) => {
    (acc[item[key]] ??= []).push(item);
    return acc;
  }, {});
}
groupBy([{ role: 'admin', name: 'A' }, { role: 'user', name: 'B' }], 'role');
// { admin: [{...}], user: [{...}] }
```
**Real-world:** grouping bookings by movie, orders by status — extremely common in UIs.

---

## Part C — Numbers & logic

### Q19. FizzBuzz (the legendary screening question)
```js
for (let i = 1; i <= 100; i++) {
  let out = '';
  if (i % 3 === 0) out += 'Fizz';
  if (i % 5 === 0) out += 'Buzz';
  console.log(out || i);
}
```
**Why asked:** filters out people who can't write a basic loop + conditional. Build the
string (don't write 4 separate `if/else` branches) to look senior.

### Q20. Check prime
```js
function isPrime(n) {
  if (n < 2) return false;
  for (let i = 2; i <= Math.sqrt(n); i++) if (n % i === 0) return false;
  return true;
}
```
**Key optimization:** loop only to `√n`, not `n` — mention this; it's the whole point.

### Q21. Factorial (iterative + recursive)
```js
const factIter = n => { let r = 1; for (let i = 2; i <= n; i++) r *= i; return r; };
const factRec  = n => (n <= 1 ? 1 : n * factRec(n - 1));
```

### Q22. Fibonacci (and why naive recursion is bad)
```js
function fib(n) {              // O(n) iterative
  let a = 0, b = 1;
  for (let i = 0; i < n; i++) [a, b] = [b, a + b];
  return a;
}
// Naive recursion fib(n) = fib(n-1)+fib(n-2) is O(2^n) — mention memoization to fix it.
```
**Why asked:** the follow-up "make the recursive version fast" tests **memoization**.

### Q23. Swap two numbers without a temp variable
```js
let a = 5, b = 9;
[a, b] = [b, a];        // cleanest (destructuring)
// or: a = a + b; b = a - b; a = a - b;
```

---

## Part D — Functions, closures & "implement this" (frontend favourites)

### Q24. Implement `debounce`
```js
function debounce(fn, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}
```
**Real-world:** search-as-you-type, resize handlers. **Why asked:** tests closures +
`this` + timers in one go — a top-3 frontend coding question.

### Q25. Implement `throttle`
```js
function throttle(fn, limit) {
  let waiting = false;
  return function (...args) {
    if (waiting) return;
    fn.apply(this, args);
    waiting = true;
    setTimeout(() => (waiting = false), limit);
  };
}
```
**Difference to debounce:** throttle runs **at most once per interval**; debounce waits
for a pause. Be ready to state that distinction.

### Q26. Implement `Array.prototype.map` (polyfill)
```js
Array.prototype.myMap = function (callback) {
  const result = [];
  for (let i = 0; i < this.length; i++) result.push(callback(this[i], i, this));
  return result;
};
```
**Why asked:** proves you understand what `map` does under the hood (and `this`).

### Q27. Implement a `once` function
```js
function once(fn) {
  let called = false, result;
  return function (...args) {
    if (!called) { called = true; result = fn.apply(this, args); }
    return result;
  };
}
```
**Real-world:** run an initializer exactly once (analytics init, config load).

### Q28. Implement deep clone
```js
function deepClone(obj) {
  if (obj === null || typeof obj !== 'object') return obj;
  if (Array.isArray(obj)) return obj.map(deepClone);
  return Object.fromEntries(
    Object.entries(obj).map(([k, v]) => [k, deepClone(v)])
  );
}
// Modern built-in: structuredClone(obj)
```
**Why asked:** tests recursion + understanding why `{...obj}` (shallow) isn't enough.

### Q29. Curry a function
```js
function curry(fn) {
  return function curried(...args) {
    return args.length >= fn.length
      ? fn(...args)
      : (...rest) => curried(...args, ...rest);
  };
}
const add = (a, b, c) => a + b + c;
curry(add)(1)(2)(3); // 6
```

### Q30. Flatten a deeply nested object (dot notation)
```js
function flattenObject(obj, prefix = '', res = {}) {
  for (const [k, v] of Object.entries(obj)) {
    const key = prefix ? `${prefix}.${k}` : k;
    if (v && typeof v === 'object' && !Array.isArray(v)) flattenObject(v, key, res);
    else res[key] = v;
  }
  return res;
}
flattenObject({ a: { b: { c: 1 } }, d: 2 }); // { 'a.b.c': 1, d: 2 }
```
**Real-world:** turning a config tree into form field names or query params.

---

## Part E — Async (asked a lot for 2+ years experience)

### Q31. Run promises in parallel and wait for all
```js
const [user, roles] = await Promise.all([getUser(), getRoles()]);
```
**Follow-up:** "what if one fails?" → `Promise.all` rejects immediately; use
`Promise.allSettled` to get every result regardless.

### Q32. Run async tasks in sequence (one after another)
```js
async function runSequentially(tasks) {
  const results = [];
  for (const task of tasks) results.push(await task()); // awaits each in order
  return results;
}
```
**Why asked:** a common mistake is `tasks.forEach(async …)` which does NOT wait — be
ready to explain why `for…of` + `await` is correct.

### Q33. Add a timeout to a promise
```js
function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) =>
    setTimeout(() => reject(new Error('Timed out')), ms));
  return Promise.race([promise, timeout]);
}
```
**Real-world:** fail a slow API call instead of hanging the UI — `Promise.race` picks
whichever settles first.

### Q34. Retry an async function n times
```js
async function retry(fn, attempts = 3, delay = 500) {
  for (let i = 0; i < attempts; i++) {
    try { return await fn(); }
    catch (err) {
      if (i === attempts - 1) throw err;
      await new Promise(r => setTimeout(r, delay));
    }
  }
}
```
**Real-world:** transient network failures — a strong, practical answer.

---

## Part F — Output-based (tricky "what prints?" questions)

These test understanding, not typing. Always explain *why*.

```js
console.log(1 + '2');        // '12'  (number coerced to string)
console.log('6' - 1);        // 5     (string coerced to number)
console.log(0.1 + 0.2);      // 0.30000000000000004 (floating point)
console.log([] + []);        // ''    (both become empty strings)
console.log(typeof null);    // 'object' (historical JS bug)
console.log(null == undefined);  // true
console.log(null === undefined); // false
```

```js
// Closures in a loop — a guaranteed favourite
for (var i = 0; i < 3; i++) setTimeout(() => console.log(i)); // 3, 3, 3
for (let i = 0; i < 3; i++) setTimeout(() => console.log(i)); // 0, 1, 2
```
**Explain:** `var` is function-scoped (one shared `i`); `let` creates a new binding per
iteration. This is the single most-asked JS trick question.

```js
// Event loop ordering
console.log('A');
setTimeout(() => console.log('B'), 0);    // macrotask
Promise.resolve().then(() => console.log('C')); // microtask
console.log('D');
// A, D, C, B  — sync first, then microtasks, then macrotasks
```

---

## Part G — More problems (round 2)

### Q35. Find the longest substring without repeating characters
```js
function longestUnique(s) {
  const seen = new Map();   // char -> last index
  let start = 0, best = 0;
  for (let i = 0; i < s.length; i++) {
    if (seen.has(s[i]) && seen.get(s[i]) >= start) start = seen.get(s[i]) + 1;
    seen.set(s[i], i);
    best = Math.max(best, i - start + 1);
  }
  return best;
}
longestUnique('abcabcbb'); // 3  ('abc')
```
**Idea:** sliding window — move `start` past the last duplicate. **Complexity:** O(n).
**Why asked:** the sliding-window pattern shows up constantly; brute force is O(n²).

### Q36. Merge two sorted arrays
```js
function merge(a, b) {
  const out = [];
  let i = 0, j = 0;
  while (i < a.length && j < b.length)
    out.push(a[i] <= b[j] ? a[i++] : b[j++]);
  return out.concat(a.slice(i)).concat(b.slice(j));
}
merge([1, 3, 5], [2, 4, 6]); // [1, 2, 3, 4, 5, 6]
```
**Idea:** two pointers walking both arrays. **Complexity:** O(n + m). The merge step of
merge sort.

### Q37. Find the intersection of two arrays
```js
function intersection(a, b) {
  const set = new Set(a);
  return [...new Set(b.filter(x => set.has(x)))];
}
intersection([1, 2, 2, 3], [2, 3, 4]); // [2, 3]
```
**Idea:** a `Set` gives O(1) lookups → O(n) overall instead of O(n²).

### Q38. Check if an array is a subset of another
```js
const isSubset = (sub, arr) => { const s = new Set(arr); return sub.every(x => s.has(x)); };
isSubset([1, 2], [1, 2, 3]); // true
```

### Q39. Count words in a string
```js
const wordCount = s => s.trim() ? s.trim().split(/\s+/).length : 0;
wordCount('the sky is blue'); // 4
```
**Watch-out:** handle the empty string — `''.split(/\s+/)` returns `['']` (length 1).

### Q40. Sum all numbers in a nested array (recursion)
```js
function deepSum(arr) {
  return arr.reduce((sum, x) =>
    sum + (Array.isArray(x) ? deepSum(x) : x), 0);
}
deepSum([1, [2, [3, 4]], 5]); // 15
```

### Q41. Implement `Array.prototype.reduce` (polyfill)
```js
Array.prototype.myReduce = function (callback, initial) {
  let acc = initial, start = 0;
  if (acc === undefined) { acc = this[0]; start = 1; } // no seed -> use first element
  for (let i = start; i < this.length; i++) acc = callback(acc, this[i], i, this);
  return acc;
};
```
**Why asked:** proves you understand the accumulator and the no-initial-value edge case.

### Q42. Implement `flat(depth)` yourself
```js
function flat(arr, depth = 1) {
  return depth < 1 ? arr.slice()
    : arr.reduce((acc, x) =>
        acc.concat(Array.isArray(x) ? flat(x, depth - 1) : x), []);
}
flat([1, [2, [3]]], 1); // [1, 2, [3]]
```

### Q43. Memoize an expensive function
```js
function memoize(fn) {
  const cache = new Map();
  return function (...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}
```
**Real-world:** cache pure computations (your Merlin.net caching story in code form).

### Q44. Capitalize, reverse each word
```js
const reverseEachWord = s => s.split(' ').map(w => [...w].reverse().join('')).join(' ');
reverseEachWord('hello world'); // 'olleh dlrow'
```

### Q45. Find missing number in 1..n
```js
function missing(arr, n) {
  const expected = (n * (n + 1)) / 2;
  return expected - arr.reduce((a, b) => a + b, 0);
}
missing([1, 2, 4, 5], 5); // 3
```
**Idea:** sum formula trick → O(n) time, O(1) space, no sorting.

### Q46. Group anagrams together
```js
function groupAnagrams(words) {
  const map = new Map();
  for (const w of words) {
    const key = [...w].sort().join('');     // sorted letters identify an anagram group
    map.set(key, [...(map.get(key) || []), w]);
  }
  return [...map.values()];
}
groupAnagrams(['eat', 'tea', 'tan', 'ate', 'nat']); // [['eat','tea','ate'],['tan','nat']]
```
**Why asked:** combines the frequency/sort key idea with a hash map — a popular follow-up.

### Q47. Implement a simple `EventEmitter` (pub/sub)
```js
class EventEmitter {
  constructor() { this.events = {}; }
  on(name, fn) { (this.events[name] ??= []).push(fn); return this; }
  off(name, fn) { this.events[name] = (this.events[name] || []).filter(f => f !== fn); }
  emit(name, ...args) { (this.events[name] || []).forEach(fn => fn(...args)); }
}
```
**Why asked:** the Observer pattern — basis of Angular `@Output`, Node streams, DOM events.

### Q48. Promisify a callback-style function
```js
const promisify = fn => (...args) =>
  new Promise((resolve, reject) =>
    fn(...args, (err, data) => (err ? reject(err) : resolve(data))));
```
**Real-world:** wrapping old Node `(err, data)` callbacks so you can `await` them.

---

### Interview tips for coding rounds
1. **Clarify first** — ask about input size, empty/null cases, duplicates.
2. **State your approach** before coding ("hash map for O(n)").
3. **Start brute force, then optimize** — show you know the trade-off.
4. **Walk a small example** by hand to verify.
5. **Mention complexity** (time & space) — seniors always do this.
6. **Handle edge cases** — empty array, single element, negatives.
