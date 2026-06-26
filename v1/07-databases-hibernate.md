# Databases, SQL, JPA & Hibernate — Deep Dive

For a frontend dev moving to backend: the database round is where many people stumble.
Master **joins, indexing, transactions, the N+1 problem, and lazy loading** — these come
up constantly.

---

## 0. New-to-backend primer (read this first)

**What is a relational database?** Think of it as a set of **Excel spreadsheets that
know how to reference each other**. Each table is a sheet; each row is a record; each
column is a field. The "relational" part means tables link via **keys** (e.g., an
`orders` row points to a `users` row by `user_id`).

**The restaurant analogy for the whole stack:**
- The **database** is the *pantry/storeroom* — where all ingredients (data) live.
- **SQL** is the *language you use to ask the storeroom* for things ("give me all
  expired items").
- **Hibernate/JPA** is a *smart assistant* that lets you say "get me this `User` object"
  in Java, and it writes the SQL for you and hands back a Java object.

**The Angular parallel:** on the frontend you call `http.get<User[]>('/api/users')` and
get typed objects. On the backend, **JPA/Hibernate is your "HttpClient for the
database"** — you call `userRepository.findAll()` and get `List<User>`, and Hibernate
handles the SQL "network call" to the DB underneath.

**The three layers of database access in Spring:**
```
Java object (User entity)           ← you work with this
      ↕  Hibernate/JPA (the ORM)    ← translates objects ↔ rows automatically
SQL table rows (users table)        ← what's actually stored
```

**Key terms in 10 seconds each:**
- **Table** — a collection of rows (like `users`).
- **Row / record** — one entry (one user).
- **Column / field** — one attribute (`email`).
- **Primary key (PK)** — the unique ID of a row (like a person's Aadhaar/SSN).
- **Foreign key (FK)** — a column pointing to another table's PK (the link).
- **Query** — a request for data (`SELECT …`).
- **Transaction** — a group of changes that succeed or fail **together** (all-or-nothing).
- **ORM** — Object-Relational Mapping: the tech (Hibernate) that maps classes ↔ tables.

Now the questions — they go fundamentals → advanced.

---

## Part A — SQL & relational fundamentals

### Q1. What are the SQL command categories?
- **DDL** (define): `CREATE`, `ALTER`, `DROP`, `TRUNCATE`.
- **DML** (manipulate): `INSERT`, `UPDATE`, `DELETE`.
- **DQL** (query): `SELECT`.
- **DCL** (control): `GRANT`, `REVOKE`.
- **TCL** (transactions): `COMMIT`, `ROLLBACK`, `SAVEPOINT`.

### Q2. Explain JOINs.
```sql
-- INNER: rows matching in both tables
SELECT u.name, o.total FROM users u
JOIN orders o ON o.user_id = u.id;

-- LEFT: all users, orders if present (NULLs otherwise)
SELECT u.name, o.total FROM users u
LEFT JOIN orders o ON o.user_id = u.id;
```
- **INNER** — intersection.
- **LEFT/RIGHT OUTER** — all from one side + matches.
- **FULL OUTER** — everything from both.
- **CROSS** — Cartesian product.
- **SELF** — table joined to itself (e.g., employee→manager).

**Beginner analogy:** imagine two guest lists — `users` (everyone who signed up) and
`orders` (everyone who bought something).
- **INNER JOIN** = people on **both** lists (signed up *and* bought).
- **LEFT JOIN** = **everyone who signed up**, with their purchase next to them if any
  (people who never bought show `NULL` for order columns). This is the most common one —
  e.g., "show all users and their total spend, including those who spent ₹0".
- **RIGHT JOIN** = the mirror image (rarely used; people just flip the table order).

**Real-world backend use:** a movie-booking dashboard query — "list every registered
user and how many tickets they booked" is a `LEFT JOIN users → bookings` so users with
zero bookings still appear.

### Q3. `WHERE` vs `HAVING`?
- **`WHERE`** filters **rows before** grouping.
- **`HAVING`** filters **groups after** `GROUP BY`/aggregation.
```sql
SELECT dept, COUNT(*) FROM employees
WHERE active = true            -- before grouping
GROUP BY dept
HAVING COUNT(*) > 5;           -- after grouping
```

### Q4. Logical order of execution of a SELECT.
`FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT`.
(Explains why you can't use a `SELECT` alias in `WHERE` but can in `ORDER BY`.)

### Q5. Aggregate functions & `GROUP BY`.
`COUNT, SUM, AVG, MIN, MAX`. Every non-aggregated `SELECT` column must be in `GROUP BY`.

### Q6. `UNION` vs `UNION ALL`?
- **`UNION`** — combines + **removes duplicates** (slower, sorts).
- **`UNION ALL`** — combines, keeps duplicates (faster). Prefer when you know there are
  no dupes.

### Q7. Subquery vs JOIN vs CTE?
- **Subquery** — nested query; correlated subqueries run per row (can be slow).
- **JOIN** — usually faster for combining tables.
- **CTE (`WITH`)** — named subquery for readability/recursion.
```sql
WITH top_spenders AS (
  SELECT user_id, SUM(total) AS spend FROM orders GROUP BY user_id HAVING SUM(total) > 1000
)
SELECT u.name, t.spend FROM users u JOIN top_spenders t ON t.user_id = u.id;
```

### Q8. Window functions? (senior signal)
Compute across a set of rows **without collapsing** them.
```sql
SELECT name, dept, salary,
       RANK() OVER (PARTITION BY dept ORDER BY salary DESC) AS rank_in_dept
FROM employees;
```
`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `LAG`, `LEAD`, running `SUM() OVER (...)`.

---

## Part B — Indexing & performance

### Q9. What is an index and how does it speed up queries?
A data structure (usually a **B-tree**) that lets the DB find rows without a full table
scan — like a book index. Speeds up `WHERE`, `JOIN`, `ORDER BY`.
```sql
CREATE INDEX idx_users_email ON users(email);
```
*(Your `email_otps` table indexes `email` for fast OTP lookups.)*

**Beginner analogy:** an index is **literally the index at the back of a textbook**.
Without it, to find every mention of "Hibernate" you'd read all 500 pages (a *full table
scan*). With the index, you jump straight to pages 42, 87, 210. A DB index does the same
for a column like `email`.

**Real-world impact:** a `users` table with 5 million rows. `SELECT * FROM users WHERE
email = 'x@y.com'`:
- **No index:** the DB scans all 5M rows → hundreds of milliseconds.
- **With index on `email`:** the DB does ~23 B-tree hops (log₂ of 5M) → sub-millisecond.
This is the single most common "why is my API slow?" fix in real backend work — the query
is filtering on an un-indexed column.

### Q10. Trade-offs of indexes?
Faster **reads**, but slower **writes** (each insert/update maintains the index) and
extra storage. Don't index everything — index columns used in filters/joins/sorts.

### Q11. Clustered vs non-clustered index?
- **Clustered** — table rows physically ordered by the index (usually the primary key);
  one per table.
- **Non-clustered** — separate structure pointing to rows; many allowed.

### Q12. Composite index & column order.
`INDEX(a, b)` helps queries filtering on `a` or `a AND b`, but **not `b` alone**
(leftmost-prefix rule). Order columns by selectivity/usage.

### Q13. How do you find a slow query?
**`EXPLAIN` / `EXPLAIN ANALYZE`** shows the execution plan — look for full table scans,
missing index usage, expensive sorts/joins. Then add indexes, rewrite, or denormalize.

---

## Part C — Normalization & design

### Q14. What is normalization? 1NF/2NF/3NF.
Organizing tables to reduce **redundancy** and anomalies.
- **1NF** — atomic values, no repeating groups.
- **2NF** — 1NF + no partial dependency on part of a composite key.
- **3NF** — 2NF + no transitive dependency (non-key → non-key).
**Denormalization** intentionally adds redundancy for read performance (trade-off).

### Q15. Primary key vs foreign key vs unique?
- **Primary key** — unique + not null, identifies a row.
- **Foreign key** — references another table's PK (referential integrity).
- **Unique** — enforces uniqueness, allows one NULL (DB-dependent).

### Q16. Constraints?
`NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, `DEFAULT`.

---

## Part D — Transactions & ACID

### Q17. What is ACID?
- **Atomicity** — all-or-nothing.
- **Consistency** — moves DB from one valid state to another (constraints hold).
- **Isolation** — concurrent transactions don't corrupt each other.
- **Durability** — committed data survives crashes.

**The classic real-world example — a bank transfer of ₹100 from A to B:**
```
step 1: subtract ₹100 from A
step 2: add ₹100 to B
```
- **Atomicity:** if the server crashes *between* step 1 and 2, you must NOT lose ₹100.
  The transaction rolls back — either *both* steps happen or *neither* does.
- **Consistency:** the total money in the system is the same before and after (no money
  invented or destroyed); all constraints (e.g., balance ≥ 0) still hold.
- **Isolation:** if two transfers run at the same time, they don't read each other's
  half-finished state.
- **Durability:** once the app says "transfer complete", a power cut won't undo it —
  it's written to disk permanently.

**In your code this is just one annotation:** `@Transactional` on a service method wraps
all its DB writes in one transaction — if any line throws, *everything* rolls back. (See
[06-spring-boot.md](06-spring-boot.md) §transactions.)

### Q18. Isolation levels & the anomalies they prevent.
| Level | Dirty read | Non-repeatable read | Phantom read |
|-------|-----------|---------------------|--------------|
| READ UNCOMMITTED | ✗ allowed | ✗ | ✗ |
| READ COMMITTED | ✓ prevented | ✗ | ✗ |
| REPEATABLE READ | ✓ | ✓ | ✗ (mostly) |
| SERIALIZABLE | ✓ | ✓ | ✓ |
Higher isolation = more safety, less concurrency. Most apps use **READ COMMITTED**
(Postgres default) or REPEATABLE READ (MySQL InnoDB default).
- **Dirty read** — reading uncommitted data.
- **Non-repeatable read** — same row read twice gives different values.
- **Phantom read** — same query returns new rows.

### Q19. Optimistic vs pessimistic locking?
- **Optimistic** — assume no conflict; use a **`@Version`** column; on conflict throw
  `OptimisticLockException`. Great for low-contention web apps.
- **Pessimistic** — lock the row (`SELECT … FOR UPDATE`) so others wait. For
  high-contention/critical sections (e.g., booking the last seat).

---

## Part E — JPA & Hibernate (the core backend round)

### Q20. JPA vs Hibernate?
- **JPA** — the **specification** (interfaces/annotations: `@Entity`, `EntityManager`).
- **Hibernate** — the most popular **implementation** of JPA (plus extras). Spring Data
  JPA sits on top.

### Q21. What is an ORM and why use one?
Object-Relational Mapping maps **objects ↔ tables**, removing boilerplate JDBC, handling
dirty checking, caching, lazy loading, and dialect differences. Trade-off: abstraction
can hide performance issues (N+1, lazy loading pitfalls).

### Q22. Basic entity mapping.
```java
@Entity
@Table(name = "users", indexes = @Index(columnList = "email"))
public class User {
  @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  @Column(nullable = false, unique = true)
  private String email;

  @Enumerated(EnumType.STRING)         // store enum name, not ordinal
  private Role role;

  @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
  private List<Booking> bookings = new ArrayList<>();
}
```

### Q23. Entity relationship mappings.
- **`@OneToOne`**, **`@OneToMany`** / **`@ManyToOne`** (the FK side is `@ManyToOne`),
  **`@ManyToMany`** (join table).
- **`mappedBy`** marks the **inverse** side (no FK column); the owning side holds the FK.

### Q24. FetchType LAZY vs EAGER — THE classic.
- **`LAZY`** — load the association only when accessed (proxy). **Default for
  collections** (`@OneToMany`, `@ManyToMany`).
- **`EAGER`** — load immediately with the parent. **Default for `@ManyToOne`/`@OneToOne`**.
**Best practice:** make everything **LAZY** and fetch explicitly when needed — EAGER
causes surprise joins and over-fetching.

**Beginner analogy:** you order a pizza (the parent `Order`).
- **EAGER** = the moment your pizza arrives, the restaurant *also* sends every side,
  drink, and dessert you *might* want — whether you asked or not. Wasteful if you only
  wanted the pizza.
- **LAZY** = you get the pizza now; the garlic bread is delivered *only if and when you
  actually ask for it*. Efficient, but if the restaurant has closed (the DB session
  ended) by the time you ask, you get an error — that's the `LazyInitializationException`
  in Q26.

### Q25. What is the N+1 select problem? How do you fix it?
Loading N parents then **one extra query per parent** for a lazy association = N+1
queries.
```java
List<Order> orders = repo.findAll();          // 1 query
for (Order o : orders) o.getItems().size();   // N more queries 😱
```
**Fixes:**
- **`JOIN FETCH`** in JPQL: `SELECT o FROM Order o JOIN FETCH o.items`.
- **`@EntityGraph`** on the repository method.
- **Batch fetching** (`@BatchSize` / `hibernate.default_batch_fetch_size`).
- **DTO projection** to select only needed columns.
**Why interviewers ask:** it's the most common ORM performance bug in production.

**Real-world story to tell:** "A list endpoint returning 100 orders was making **101
database calls** — 1 to load the orders, then 1 per order to load its items — and taking
2+ seconds. I spotted it by enabling `spring.jpa.show-sql=true` and seeing a flood of
identical `SELECT * FROM items WHERE order_id = ?` queries. I added `JOIN FETCH` so it
became **a single query**, and response time dropped to ~50ms." That story alone often
passes the JPA round — it shows you can *diagnose*, not just define.

```java
// The fix as a repository method:
@Query("SELECT o FROM Order o JOIN FETCH o.items")
List<Order> findAllWithItems();
```

### Q26. What is `LazyInitializationException` and why does it happen?
Accessing a LAZY association **after the persistence context (session) is closed** —
e.g., in the controller/serialization after the `@Transactional` service method
returned.
**Fixes:** fetch within the transaction (`JOIN FETCH`/`@EntityGraph`), map to a **DTO**
inside the transaction, or (anti-pattern) Open-Session-In-View (enabled by default in
Boot but discouraged for performance).

### Q27. Hibernate caching — first vs second level.
- **First-level (L1)** — the **persistence context / session**; mandatory, per
  transaction. Same entity fetched twice → one DB hit.
- **Second-level (L2)** — optional, **shared across sessions** (Ehcache, Caffeine,
  Redis). Cache rarely-changing reference data. Plus a **query cache** for query results.

### Q28. Entity lifecycle states.
- **Transient** — new object, not associated with a session, no DB row.
- **Persistent/Managed** — associated with a session; changes auto-flushed (dirty
  checking).
- **Detached** — was persistent, session closed; changes not tracked.
- **Removed** — marked for deletion.

### Q29. What is dirty checking?
For managed entities, Hibernate compares their state at flush/commit and **auto-generates
`UPDATE`** for changed fields — no explicit `save()` needed.
```java
@Transactional
public void enable(Long id) {
  User u = repo.findById(id).orElseThrow();
  u.setEnabled(true);   // no save() call — flushed on commit (your verifyEmail does this)
}
```

### Q30. `save` / `persist` / `merge` / `saveAndFlush`?
- **`persist`** — make a transient entity managed (JPA).
- **`merge`** — copy a detached entity's state into a managed one (returns the managed
  copy).
- **`save`** (Spring Data) — persist or merge; returns the entity.
- **`flush`** — push pending changes to the DB now (still within the transaction).

### Q31. `@GeneratedValue` strategies?
- **`IDENTITY`** — DB auto-increment (one insert per row; disables JDBC batching).
- **`SEQUENCE`** — DB sequence (best for batching; preferred on Postgres/Oracle).
- **`AUTO`** — provider picks.
- **`TABLE`** — emulated via a table (slow; avoid).

### Q32. DTO projection — why and how?
Fetch only the fields you need (avoids loading full entities, N+1, lazy issues, and
over-exposing data).
```java
public interface UserView { Long getId(); String getName(); }   // interface projection
@Query("select u.id as id, u.name as name from User u")
List<UserView> findAllViews();
// or constructor expression / record DTO
```

### Q33. `CascadeType` & `orphanRemoval`?
- **Cascade** propagates operations to associations (`PERSIST`, `MERGE`, `REMOVE`,
  `ALL`).
- **`orphanRemoval = true`** deletes a child when removed from the parent's collection.

---

## Part F — Production concerns

### Q34. Why is `spring.jpa.hibernate.ddl-auto=update`/`create` dangerous in prod?
Hibernate auto-generating schema changes is **unpredictable and can drop/alter data**.
In production set **`ddl-auto: validate`** (or `none`) and manage schema with
**migrations**.

### Q35. Flyway vs Liquibase?
Database **migration / version-control** tools — apply ordered, repeatable schema
changes across environments.
- **Flyway** — SQL-first, simple versioned scripts (`V1__init.sql`).
- **Liquibase** — changelog in XML/YAML/JSON/SQL, more abstraction & rollback support.
**Why:** reproducible, auditable schema evolution; never hand-edit prod schema.

### Q36. SQL injection — how do you prevent it?
Use **parameterized queries / prepared statements** (JPA/Spring Data do this for you).
Never concatenate user input into SQL.
```java
// safe — bound parameter
@Query("select u from User u where u.email = :email")
Optional<User> findByEmail(@Param("email") String email);
```
Never: `"… where email = '" + input + "'"`.

**Real-world example of the attack:** a login form builds SQL by string concatenation:
```java
"SELECT * FROM users WHERE email = '" + email + "' AND password = '" + pwd + "'"
```
An attacker types `' OR '1'='1` as the email. The query becomes
`... WHERE email = '' OR '1'='1' AND ...` — `'1'='1'` is always true, so it returns the
first user and logs them in **without a password**. Famous breaches (and the "Bobby
Tables" XKCD comic) are exactly this. Parameterized queries fix it because the input is
sent as *data*, never parsed as SQL. This is OWASP's #1 risk — a guaranteed interview
topic for backend roles.

### Q37. Connection pooling?
Reusing DB connections (expensive to open) via a pool. Spring Boot uses **HikariCP** by
default. Tune `maximum-pool-size` to your DB/load.

### Q38. SQL vs NoSQL — when?
- **SQL (relational)** — structured data, relationships, transactions, complex queries
  (most business apps).
- **NoSQL** — flexible schema, horizontal scale, specific access patterns: document
  (MongoDB), key-value (Redis), wide-column (Cassandra), graph (Neo4j).

**Real-world rule of thumb:** start with SQL (Postgres/MySQL) for anything with
relationships and money/correctness needs (users, orders, payments) — you get ACID
transactions for free. Reach for NoSQL for specific jobs: **Redis** to cache sessions or
hot data, **MongoDB** for flexible document blobs (e.g., product catalogs with varying
fields), **Elasticsearch** for full-text search. Most real systems use *both* (e.g.,
Postgres as the source of truth + Redis as a cache).

### Q39. What is a primary key strategy — auto-increment vs UUID? (common follow-up)
- **Auto-increment / sequence** (`1, 2, 3…`) — small, fast, index-friendly, but
  *guessable* and reveals row counts; harder across distributed systems.
- **UUID** (`550e8400-e29b-…`) — globally unique, safe to generate on the client or
  across services, not guessable; but larger (16 bytes) and worse for index locality.
**Practical answer:** sequence/identity for single-DB internal IDs; UUID when IDs are
exposed publicly or generated across services.

### Q40. What is database normalization vs denormalization in practice?
- **Normalize** to avoid duplicating data (store a `user_id` in `orders`, not the user's
  whole name/email in every order row). Prevents update anomalies.
- **Denormalize** *deliberately* for read speed when joins get expensive — e.g., store a
  pre-computed `total_orders` count on the user row so a dashboard doesn't `COUNT` every
  time. Trade-off: you must keep the duplicated value in sync.
**Interview-safe stance:** "Normalize first; denormalize only when profiling shows a real
read bottleneck."

### Q41. `JOIN FETCH` vs `@EntityGraph` — what's the difference? (JPA follow-up)
Both solve N+1 by fetching associations in one query. **`JOIN FETCH`** is written inside
a JPQL `@Query`; **`@EntityGraph`** is a declarative annotation on a repository method
that keeps the method name/derived query and just tells Hibernate what to fetch eagerly
for *that* call. Use `@EntityGraph` when you want to reuse Spring Data derived queries
without writing JPQL.
```java
@EntityGraph(attributePaths = "items")
List<Order> findByStatus(String status);   // fetches items in one query
```

---

### Rapid-fire
- **`DELETE` vs `TRUNCATE` vs `DROP`?** Rows (logged, rollback-able) / all rows (fast,
  resets) / whole table.
- **`CHAR` vs `VARCHAR`?** Fixed vs variable length.
- **Composite key?** PK over multiple columns.
- **`COUNT(*)` vs `COUNT(col)`?** All rows vs non-null values of col.
- **What is a deadlock in DB?** Two transactions waiting on each other's locks; DB kills
  one.
- **Idempotency?** Same request repeated has the same effect (important for retries).
- **Read replica?** Copy of DB for scaling reads.

---

### Likely deep-dive chain
> "LAZY vs EAGER?" → "What's the N+1 problem?" → "Fix it with `JOIN FETCH`/EntityGraph"
> → "What's `LazyInitializationException`?" → "Why not `ddl-auto=update` in prod?" →
> "How do indexes work and their trade-offs?"
