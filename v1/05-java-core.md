# Java Core Interview Questions (Java 8–17) — Deep Dive

Written for a frontend dev learning backend. Focus on the concepts interviewers test
**most** and that you'll actually use in Spring Boot. Examples are runnable mentally.

---

## 0. New-to-backend primer (read this first)

If you're coming from Angular/TypeScript, here's the mental bridge so the rest of this
file clicks:

| You know (JS/TS) | Java equivalent | Note |
|------------------|-----------------|------|
| `let x = 1` | `int x = 1;` | Java needs an explicit **type**; it's *statically typed* like TS but stricter. |
| `interface User {}` | `interface User {}` / `class User {}` | Same idea; Java interfaces can have method bodies (`default`) since Java 8. |
| `Array.map/filter` | **Streams** (`.stream().map()`) | Lazy pipelines over collections. |
| `Promise` | `CompletableFuture` | Async composition. |
| `null` / `undefined` | `null` (no `undefined`) + `Optional` | `Optional` is Java's "maybe a value". |
| npm + `package.json` | **Maven/Gradle** + `pom.xml` | Build tool + dependency manager. |
| Node runtime | **JVM** | Compiles `.java` → bytecode → JVM runs it on any OS. |
| `class` (sugar over prototypes) | `class` (real, first-class) | Java is class-based from the ground up. |

**The one big difference:** TypeScript types vanish at runtime (they're just
compile-time hints). **Java types are real at runtime** — the JVM enforces them, which
is why backend code is so predictable and why interviewers care about types, generics,
and collections.

**How a backend request actually flows** (keep this picture in your head for the whole
backend section):
```
HTTP request → Controller (web layer)
             → Service (business logic)
             → Repository (data access)
             → Database
             ← returns objects back up the same chain → JSON response
```
Everything below — OOP, collections, exceptions, Spring, Hibernate — is about making
that pipeline correct, fast, and maintainable.

---

## 1. OOP & language fundamentals

### Q1. Four pillars of OOP (with Java examples).
- **Encapsulation** — hide state behind methods (private fields + getters/setters).
- **Inheritance** — `extends` to reuse/extend behavior.
- **Polymorphism** — same interface, different implementations (overriding/overloading).
- **Abstraction** — expose *what*, hide *how* (abstract classes/interfaces).
```java
abstract class Shape { abstract double area(); }            // abstraction
class Circle extends Shape {                                 // inheritance
  private final double r;                                    // encapsulation
  Circle(double r) { this.r = r; }
  double area() { return Math.PI * r * r; }                  // polymorphism (override)
}
```
**Real-world analogy (a car):**
- **Encapsulation** — you press the accelerator; you don't touch the fuel injectors.
  The engine's internals are *hidden* behind a simple pedal (public method).
- **Inheritance** — an `ElectricCar` *is a* `Car`; it reuses wheels/steering and adds a
  battery.
- **Polymorphism** — every car has a `start()` button, but a petrol car and an electric
  car *start differently*. Same call, different behavior.
- **Abstraction** — "Driving" is the abstract idea; you don't need to know combustion vs
  electric to drive.

**Where you'll use it in backend:** a `PaymentService` interface with `CreditCardPayment`
and `UpiPayment` implementations — your code calls `payment.process()` and the right one
runs (polymorphism), without the caller knowing the details (abstraction).

### Q2. Overloading vs overriding?
- **Overloading** — same method name, **different parameters**, same class
  (compile-time / static polymorphism).
- **Overriding** — subclass redefines a parent method with the **same signature**
  (runtime / dynamic polymorphism). Use `@Override`.

### Q3. `abstract class` vs `interface`?
| | Abstract class | Interface |
|--|---------------|-----------|
| Multiple inheritance | no (single) | yes (many) |
| State/fields | yes | only `static final` constants |
| Constructors | yes | no |
| Methods | abstract + concrete | abstract + `default`/`static` (Java 8+) |
| Use when | shared base + state | capability/contract |
```java
interface Payable { default String currency() { return "USD"; } }
```
**Modern note:** since Java 8 interfaces have `default`/`static` methods, blurring the
line — but interfaces still can't hold instance state.

### Q4. `final`, `finally`, `finalize`?
- **`final`** — constant variable / non-overridable method / non-extendable class.
- **`finally`** — block that always runs after try/catch (cleanup).
- **`finalize`** — deprecated GC hook (don't use; use try-with-resources).

### Q5. `==` vs `.equals()`?
`==` compares **references** (identity) for objects; `.equals()` compares **value**
(if overridden).
```java
String a = new String("hi"), b = new String("hi");
a == b;        // false (different objects)
a.equals(b);   // true  (same value)
```

### Q6. `equals()` and `hashCode()` contract — why override both?
If two objects are `equals`, they **must** have the same `hashCode`. Break it and
`HashMap`/`HashSet` misbehave (lost keys, duplicates).
```java
@Override public boolean equals(Object o) {
  if (this == o) return true;
  if (!(o instanceof User u)) return false;
  return id == u.id && Objects.equals(email, u.email);
}
@Override public int hashCode() { return Objects.hash(id, email); }
```
**Why interviewers ask:** it's the #1 source of subtle collection bugs.

### Q7. String immutability & the String pool.
`String` is **immutable** — any "modification" creates a new object. Benefits: thread
safety, caching/interning, safe as map keys. Literal strings live in the **string pool**.
```java
String s = "hi"; s.concat("!"); // s is still "hi" (new string discarded)
```
**`String` vs `StringBuilder` vs `StringBuffer`:** use `StringBuilder` for heavy
concatenation (mutable, not synchronized); `StringBuffer` is the synchronized version.

### Q8. Primitives vs wrapper classes / autoboxing.
`int` (stack, fast) vs `Integer` (object, nullable, for collections/generics).
**Autoboxing** converts automatically — beware `NullPointerException` when unboxing
`null`, and `Integer` caching (`==` works for -128..127 only).

### Q9. `static` keyword?
Belongs to the **class**, not instances: static fields (shared), static methods (no
`this`), static blocks (run once at class load), static nested classes.

### Q10. Pass-by-value or pass-by-reference?
Java is **always pass-by-value**. For objects, the **value of the reference** is copied
— so you can mutate the object but not reassign the caller's variable.

---

## 2. Collections framework

### Q11. Explain the Collections hierarchy.
- **`List`** (ordered, duplicates): `ArrayList`, `LinkedList`.
- **`Set`** (no duplicates): `HashSet`, `LinkedHashSet`, `TreeSet`.
- **`Map`** (key→value): `HashMap`, `LinkedHashMap`, `TreeMap`, `ConcurrentHashMap`.
- **`Queue`/`Deque`**: `ArrayDeque`, `PriorityQueue`.

### Q12. `ArrayList` vs `LinkedList`?
- **`ArrayList`** — backed by an array; **O(1) random access**, slow inserts/removes in
  the middle. Best default.
- **`LinkedList`** — doubly linked; O(1) insert/remove at ends, **O(n) access**. Rarely
  better in practice (cache-unfriendly).

### Q13. How does `HashMap` work internally?
Array of **buckets**; key's `hashCode` → bucket index. Collisions chained in a
**linked list**, converted to a **balanced tree (red-black)** when a bucket exceeds 8
entries (Java 8+) for O(log n) worst case. `equals` resolves within a bucket. Resizes
(rehash) when load factor (0.75) exceeded.
**Why both `hashCode` & `equals` matter** (see Q6).

**Real-world analogy (a coat check / cloakroom):** the `hashCode` is like the *ticket
number* that tells the attendant which shelf (bucket) your coat is on. If two coats land
on the same shelf (collision), the attendant checks each one (`equals`) to find yours.
A good `hashCode` spreads coats evenly so lookups stay O(1). A broken `hashCode` (all
coats on one shelf) degrades it to a slow linear search.

**Where you'll use it:** caching user sessions by user-id, counting word frequencies,
de-duplicating records — `HashMap` is the workhorse of backend code.

### Q14. `HashMap` vs `Hashtable` vs `ConcurrentHashMap`?
- **`HashMap`** — not synchronized, allows one null key. Fast, single-threaded.
- **`Hashtable`** — legacy, fully synchronized (slow), no nulls.
- **`ConcurrentHashMap`** — thread-safe via fine-grained locking/CAS (bucket-level),
  no nulls. **Use this for concurrency**, not `Hashtable`.

### Q15. `HashSet` vs `TreeSet` vs `LinkedHashSet`?
- **`HashSet`** — unordered, O(1).
- **`LinkedHashSet`** — insertion order preserved.
- **`TreeSet`** — sorted (Comparable/Comparator), O(log n).

### Q16. `Comparable` vs `Comparator`?
- **`Comparable`** — natural ordering, `compareTo`, one per class.
- **`Comparator`** — external/multiple orderings.
```java
users.sort(Comparator.comparing(User::lastName).thenComparing(User::firstName).reversed());
```

### Q17. Fail-fast vs fail-safe iterators.
- **Fail-fast** (`ArrayList`, `HashMap`) — throw `ConcurrentModificationException` if
  modified during iteration.
- **Fail-safe** (`ConcurrentHashMap`, `CopyOnWriteArrayList`) — iterate over a copy/
  snapshot, no exception.

---

## 3. Generics

### Q18. Why generics? Type erasure?
Generics give **compile-time type safety** and remove casts. At runtime types are
**erased** (replaced with `Object`/bounds) for backward compatibility — so you can't do
`new T()` or `instanceof List<String>`.
```java
List<String> list = new ArrayList<>();  // no casts needed on get()
```

### Q19. Bounded types & wildcards (PECS).
- `<T extends Number>` — upper bound.
- `<? extends T>` — **Producer** (read-only source).
- `<? super T>` — **Consumer** (write-only sink).
- **PECS:** "Producer Extends, Consumer Super."
```java
void copy(List<? extends Src> from, List<? super Src> to) { … }
```

---

## 4. Streams & functional (Java 8) — heavily tested

### Q20. What are streams? Lazy evaluation?
A **pipeline** for processing collections declaratively: source → **intermediate ops**
(lazy: `map`, `filter`, `sorted`) → **terminal op** (eager: `collect`, `forEach`,
`reduce`). Nothing runs until the terminal op.
```java
List<String> names = users.stream()
    .filter(u -> u.getAge() > 18)
    .map(User::getName)
    .sorted()
    .collect(Collectors.toList());
```
**If you know JS:** this is exactly `users.filter(...).map(...).sort()` — same idea,
just Java syntax. The difference is **laziness**: intermediate ops don't run until a
terminal op pulls data through, so the JVM can optimize the whole chain in one pass.

**Real-world backend example** — turn a list of order entities into a report:
```java
// "Total revenue per city for completed orders over $100"
Map<String, Double> revenueByCity = orders.stream()
    .filter(o -> o.getStatus() == COMPLETED)
    .filter(o -> o.getTotal() > 100)
    .collect(Collectors.groupingBy(
        Order::getCity,
        Collectors.summingDouble(Order::getTotal)));
```
This replaces ~20 lines of nested loops with one readable pipeline — the kind of code
you'll write constantly in services.

### Q21. `map` vs `flatMap`?
- **`map`** — 1:1 transform.
- **`flatMap`** — flattens nested streams (1:many → one stream).
```java
List<String> all = orders.stream()
    .flatMap(o -> o.getItems().stream())   // List<Item> per order → one stream
    .map(Item::getName).toList();
```

### Q22. Common `Collectors`?
```java
Collectors.toList();
Collectors.toSet();
Collectors.joining(", ");
Collectors.groupingBy(User::getDept);                  // Map<Dept, List<User>>
Collectors.groupingBy(User::getDept, Collectors.counting());
Collectors.partitioningBy(u -> u.getAge() > 18);
Collectors.toMap(User::getId, Function.identity());
```

### Q23. Functional interfaces & lambdas.
A functional interface has **one abstract method** (`@FunctionalInterface`). Built-ins:
- **`Function<T,R>`** — `apply` (transform).
- **`Predicate<T>`** — `test` (boolean).
- **`Consumer<T>`** — `accept` (side effect).
- **`Supplier<T>`** — `get` (produce).
- **`BiFunction`, `UnaryOperator`, `BinaryOperator`**.
```java
Predicate<Integer> isEven = n -> n % 2 == 0;
Function<String, Integer> len = String::length;   // method reference
```

### Q24. `Optional` — why and how?
A container that may hold a value — replaces null checks, signals "may be absent."
```java
Optional<User> u = repo.findById(id);
String name = u.map(User::getName).orElse("Unknown");
u.ifPresent(this::send);
repo.findById(id).orElseThrow(() -> new NotFoundException(id));
```
**Anti-pattern:** `Optional.get()` without checking; don't use Optional for fields/params.

### Q25. `reduce`?
```java
int sum = nums.stream().reduce(0, Integer::sum);
Optional<String> longest = words.stream().reduce((a, b) -> a.length() >= b.length() ? a : b);
```

### Q26. Parallel streams — when (not) to use?
`stream().parallel()` splits work across the fork-join pool. Good for **large,
CPU-bound, stateless** operations; **bad** for small data, I/O, or shared mutable state
(non-deterministic, ordering issues). Usually not worth it — measure first.

---

## 5. Exceptions

### Q27. Checked vs unchecked exceptions.
- **Checked** (`IOException`, `SQLException`) — must be declared/caught; recoverable
  conditions.
- **Unchecked** (`RuntimeException`: `NullPointerException`, `IllegalArgumentException`)
  — programming errors; not forced to handle.
- **`Error`** (`OutOfMemoryError`) — don't catch.

**Real-world analogy:** a **checked** exception is like a restaurant telling you upfront
"we might be out of your dish — have a backup plan" (you're forced to deal with it). An
**unchecked** exception is like tripping over your own shoelace — a bug in *how you
walked*, not something the restaurant warned you about. `Error` is the building catching
fire — you don't "handle" it, you get out.

**Backend reality:** most modern Spring code prefers **unchecked** exceptions (e.g., a
custom `ResourceNotFoundException extends RuntimeException`) because checked exceptions
clutter every method signature. You then map them to HTTP status codes centrally (see
the Spring file's `@RestControllerAdvice`).

### Q28. try-with-resources?
Auto-closes resources implementing `AutoCloseable` — no leaky `finally`.
```java
try (var conn = dataSource.getConnection();
     var ps = conn.prepareStatement(sql)) {
  // use; auto-closed in reverse order even on exception
}
```

### Q29. Best practices for exceptions.
Catch specific types; don't swallow (`catch (Exception e) {}`); wrap with context;
prefer unchecked for unrecoverable; custom domain exceptions; never use exceptions for
control flow.

---

## 6. Concurrency (basics, commonly asked)

### Q30. Thread vs Runnable vs ExecutorService.
Prefer **`ExecutorService`** (thread pools) over manual `new Thread()`.
```java
ExecutorService pool = Executors.newFixedThreadPool(4);
Future<Integer> f = pool.submit(() -> compute());
pool.shutdown();
```

### Q31. `synchronized`, `volatile`, atomics.
- **`synchronized`** — mutual exclusion + visibility (one thread in a block at a time).
- **`volatile`** — visibility only (reads/writes go to main memory), no atomicity.
- **`AtomicInteger`/`AtomicLong`** — lock-free atomic operations via CAS.

### Q32. What is a race condition / deadlock?
- **Race condition** — outcome depends on thread timing on shared mutable state.
- **Deadlock** — two threads each hold a lock the other needs. Avoid by lock ordering,
  timeouts, fewer locks.

### Q33. `CompletableFuture`?
Async composition (like JS Promises).
```java
CompletableFuture.supplyAsync(() -> fetch())
    .thenApply(this::transform)
    .thenAccept(this::save)
    .exceptionally(ex -> { log(ex); return null; });
```

### Q34. What changed with Java 21 virtual threads? (good to mention)
**Virtual threads (Project Loom)** are lightweight threads managed by the JVM — millions
can run cheaply, making blocking I/O scalable without reactive complexity.

---

## 7. JVM & memory

### Q35. JDK vs JRE vs JVM.
- **JVM** — runs bytecode (platform-specific).
- **JRE** — JVM + standard libraries (run apps).
- **JDK** — JRE + compiler/tools (develop apps).

### Q36. Stack vs heap.
- **Stack** — per-thread; method frames, local variables, references. LIFO, fast.
- **Heap** — shared; all objects live here; managed by GC.

### Q37. Garbage collection basics.
GC reclaims unreachable objects. **Generational:** Young gen (Eden + Survivor, frequent
minor GC) → Old gen (major GC). Modern collectors: **G1** (default), **ZGC**/**Shenandoah**
(low pause). You don't free memory manually; you can hint with nulling references.

### Q38. What is a memory leak in Java?
Objects unintentionally kept reachable (static collections growing forever, unclosed
resources, listeners not removed) so GC can't collect them.

---

## 8. Java 8–17 features (know the timeline)

- **Java 8:** lambdas, streams, `Optional`, default methods, new Date/Time API.
- **Java 9:** modules (JPMS), `var` (10).
- **Java 10/11:** `var` for locals, `String` methods, HTTP Client (11 = LTS).
- **Java 14–16:** `switch` expressions, **records**, **pattern matching for
  `instanceof`**, text blocks, sealed classes (preview→17).
- **Java 17 (LTS):** sealed classes, finalized records/pattern matching.

### Q39. What is a `record`?
Immutable data carrier — auto-generates constructor, getters, `equals`, `hashCode`,
`toString`.
```java
public record User(Long id, String name, String email) {}
// used heavily for DTOs in Spring Boot
```

### Q40. `switch` expressions & pattern matching.
```java
String size = switch (n) {
  case 1, 2, 3 -> "small";
  case 4, 5 -> "medium";
  default -> "large";
};
// pattern matching
if (obj instanceof User u) { System.out.println(u.name()); }
```

### Q41. Sealed classes?
Restrict which classes can extend/implement — exhaustive, safer hierarchies.
```java
sealed interface Shape permits Circle, Square {}
```

### Q42. Text blocks?
```java
String json = """
    { "name": "Ann", "active": true }
    """;
```

---

## 9. More commonly-asked questions (with real-world framing)

### Q43. What is the difference between `interface` and `abstract class` — when do you pick which? (asked constantly)
Use an **interface** when you're defining a **capability/contract** many unrelated
classes can have (`Comparable`, `Runnable`, `PaymentMethod`). Use an **abstract class**
when classes share a **common base with state and partial implementation** (`Animal`
with a stored `name` + a `sleep()` method, but abstract `makeSound()`).
**Rule of thumb:** "Can do" (interface) vs "Is a" (abstract class). A `Duck` *is an*
`Animal` (abstract class) but *can* `Swim` (interface).

### Q44. What is the `this` keyword vs JavaScript's `this`?
In Java `this` **always** refers to the current object instance \u2014 no rebinding, no
`bind/call/apply` confusion like JS. It's used to disambiguate fields from parameters
(`this.name = name;`) and to pass the current object. Much simpler than JS.

### Q45. What is method chaining / the Builder pattern? (you'll see it everywhere)
A pattern for constructing objects with many optional fields readably \u2014 instead of a
10-argument constructor.
```java
User user = User.builder()
    .name(\"Ann\")
    .email(\"ann@x.com\")
    .active(true)
    .build();
```
**Real-world:** Lombok's `@Builder`, the `StringBuilder`, and Spring's `HttpSecurity`
config all use this. Interviewers like it because it shows you write maintainable APIs.

### Q46. Shallow copy vs deep copy?
- **Shallow** \u2014 copies the object but **shares** nested object references (change the
  nested object and both \"copies\" see it).
- **Deep** \u2014 copies everything recursively; fully independent.
```java
List<int[]> original = ...;\nList<int[]> shallow = new ArrayList<>(original); // inner arrays still shared\n```\n**Backend gotcha:** returning a shallow copy of an internal list lets callers mutate\nyour object's state. Return **defensive copies** or immutable views\n(`List.copyOf(...)`).\n\n### Q47. What is `Comparable`/`Comparator` used for in real apps?\nSorting query results that the DB didn't sort, ranking search results, ordering a\nleaderboard. Example \u2014 sort users by signup date, newest first:\n```java\nusers.sort(Comparator.comparing(User::getSignupDate).reversed());\n```\n\n### Q48. What is the `static` factory method pattern?\nA `static` method that returns an instance, often clearer than a constructor:\n```java\nOptional.of(x);  List.of(1, 2, 3);  Duration.ofMinutes(10);  User.fromEntity(entity);\n```\nBenefits: meaningful names, can return cached/subtype instances, no `new` everywhere.\n\n### Q49. How does Java handle memory \u2014 do I free objects like in C?\nNo. You `new` objects; the **garbage collector** automatically frees them when nothing\nreferences them anymore. Your job is just to **not hold references longer than needed**\n(e.g., don't stuff everything into a `static` list forever). Coming from JS, this is\nfamiliar \u2014 the JVM GC is like the V8 GC.\n\n### Q50. What is the difference between `==` and `.equals()` for `Integer`? (classic trap)\n```java\nInteger a = 127, b = 127;\nInteger c = 128, d = 128;\na == b;        // true  \u2014 Integer caches -128..127\nc == d;        // false \u2014 outside cache, different objects!\nc.equals(d);   // true  \u2014 always use .equals() for object value comparison\n```\nThis trips up *everyone* \u2014 always compare object values with `.equals()`.\n\n### Q51. What's the difference between `final`, immutability, and a constant?\n- `final` variable \u2014 reference can't be reassigned (but the object may still be mutable:\n  `final List` can still `.add()`).\n- **Immutable object** \u2014 its state never changes after construction (`String`, `record`).\n- **Constant** \u2014 `static final` (shared, fixed): `static final int MAX = 100;`.\n\n### Q52. Why is `String` immutable, practically?\nSecurity (a filename/URL passed to a method can't be changed underneath you), safe\nsharing across threads, and reuse via the string pool. If you need to build strings in\na loop, use `StringBuilder` (mutable) to avoid creating thousands of throwaway `String`\nobjects.\n\n---\n\n### Rapid-fire
- **Marker interface?** Empty interface signaling capability (`Serializable`).
- **`transient`?** Skip a field during serialization.
- **Immutable class — how?** `final` class, `final` private fields, no setters,
  defensive copies.
- **`this` vs `super`?** Current instance vs parent.
- **Can you override `static`?** No — it's **hidden**, not overridden.
- **Default access modifier?** Package-private.
- **`Iterable` vs `Iterator`?** Iterable produces an Iterator (`iterator()`); Iterator
  has `hasNext()/next()`.
- **Why is `String` final?** Security, caching, thread-safety.

---

### Likely deep-dive chain
> "Override `equals`?" → "Why also `hashCode`?" → "How does `HashMap` use them?" →
> "Stream to group users by dept?" → "Checked vs unchecked exception?"
