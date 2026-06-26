# Spring & Spring Boot Interview Questions — Deep Dive

Grounded in the most-asked themes at big companies and tied to the **movie-booking app
you just built** (JWT auth, refresh tokens, email-OTP, JPA). Where useful, examples
mirror that code so you can speak from real experience.

---

## 0. New-to-backend primer (read this first)

**What is Spring, in one sentence?** A framework that wires your objects together and
handles all the repetitive plumbing (web server, database connections, security,
transactions) so you write mostly *business logic*.

**The Angular parallel that makes it click:** you already understand Angular's
**Dependency Injection** — you put `@Injectable()` on a service and Angular hands it to
any component that asks via the constructor. **Spring does the exact same thing on the
backend.** `@Service`/`@Component` is like `@Injectable()`, and constructor injection
works identically. If you get Angular DI, you already get the core of Spring.

| Angular concept | Spring equivalent |
|-----------------|-------------------|
| `@Injectable()` service | `@Service` / `@Component` bean |
| Constructor injection | Constructor injection (same!) |
| `providedIn: 'root'` singleton | default `singleton` bean scope |
| `HttpInterceptor` | `OncePerRequestFilter` / interceptor |
| Route guard | Spring Security filter / `@PreAuthorize` |
| `environment.ts` | `application.yml` + profiles |
| Angular module/providers | `@Configuration` + `@Bean` |

**The layered architecture you'll build (memorize this):**
```
@RestController   → handles HTTP, validates input, returns JSON   (the "waiter")
      ↓
@Service          → business logic, rules, orchestration          (the "kitchen")
      ↓
@Repository       → talks to the database (Spring Data JPA)        (the "pantry")
      ↓
  Database
```
Keep each layer focused: controllers don't touch the DB directly; services hold the
rules; repositories just fetch/save. This separation is what interviewers look for.

**"Spring" vs "Spring Boot":** Spring is the core framework (powerful but lots of manual
config). **Spring Boot** sits on top and auto-configures sensible defaults + an embedded
server, so you can run a real API with ~20 lines. You'll almost always use Spring Boot.

---

## 1. Core: IoC & DI

### Q1. What is Inversion of Control (IoC) and Dependency Injection (DI)?
- **IoC** — the framework, not your code, controls object creation and wiring.
- **DI** — a form of IoC where dependencies are **injected** into a class rather than
  the class creating them. Spring's **ApplicationContext** (the IoC container) manages
  this.
**Benefits:** loose coupling, testability (inject mocks), single responsibility.

**Real-world analogy (a restaurant):** A chef (your `OrderService`) needs ingredients
(a `PaymentGateway`, an `EmailService`). *Without* DI, the chef drives to the farm, the
bank, and the post office himself — tightly coupled and impossible to test. *With* DI,
the restaurant (Spring) **delivers** the exact ingredients to the chef's station. The
chef just cooks. Want to test the chef? Hand him fake ingredients (mocks). That's the
whole point: your class declares *what* it needs, Spring supplies it.

```java
// The class just declares its needs; Spring provides them.
@Service
public class OrderService {
  private final PaymentGateway payment;
  private final EmailService email;
  public OrderService(PaymentGateway payment, EmailService email) { // injected
    this.payment = payment; this.email = email;
  }
}
```

### Q2. Constructor vs field vs setter injection — which and why?
**Constructor injection is preferred.**
- Guarantees **required** dependencies (object always valid).
- Enables **`final`** fields (immutability).
- **Testable** without reflection (just `new Service(mock)`).
- Surfaces "too many dependencies" code smell.
```java
@Service
public class AuthServiceImpl implements AuthService {
  private final UserRepository userRepo;
  private final PasswordEncoder encoder;
  private final OtpService otpService;
  // single constructor → @Autowired optional
  public AuthServiceImpl(UserRepository userRepo, PasswordEncoder encoder, OtpService otpService) {
    this.userRepo = userRepo; this.encoder = encoder; this.otpService = otpService;
  }
}
```
**Field injection** (`@Autowired` on a field) is discouraged: hidden dependencies, can't
be `final`, hard to unit test, can cause circular-dependency surprises.

### Q3. What's a Spring Bean? Bean scopes?
A **bean** is an object managed by the Spring container. Scopes:
- **`singleton`** (default) — one instance per container.
- **`prototype`** — new instance every injection/request.
- Web scopes: **`request`**, **`session`**, **`application`**, **`websocket`**.

**Plain-English:** a "bean" is just *an object that Spring creates and remembers for
you* so it can hand it out wherever needed. You rarely write `new MyService()` in
backend code — Spring news it up once (singleton) and reuses it, the same way Angular
keeps one `providedIn: 'root'` service for the whole app.

**When scope matters:** a `singleton` service must be **stateless** (no per-user fields)
because every request shares the same instance across threads. Per-request state goes in
method parameters/local variables, not bean fields.

### Q4. `@Component` vs `@Service` vs `@Repository` vs `@Controller`?
All are **stereotypes** (specializations of `@Component`) for component scanning:
- **`@Component`** — generic bean.
- **`@Service`** — business logic (semantic).
- **`@Repository`** — data access; adds **exception translation** (JDBC/JPA → Spring's
  `DataAccessException`).
- **`@Controller` / `@RestController`** — web layer.

### Q5. `@Component` (stereotype) vs `@Bean`?
- **`@Component`** — class-level; Spring auto-detects via component scanning. For *your*
  classes.
- **`@Bean`** — method-level inside `@Configuration`; you construct it manually. For
  **third-party** classes you can't annotate, or when construction needs logic.
```java
@Configuration
public class SecurityConfig {
  @Bean PasswordEncoder passwordEncoder() { return new BCryptPasswordEncoder(); }
}
```

### Q6. How does Spring resolve multiple beans of the same type? (`@Qualifier`, `@Primary`)
```java
@Bean @Primary Logger fileLogger() { … }     // default choice
@Bean Logger dbLogger() { … }
// at injection site:
public Svc(@Qualifier("dbLogger") Logger logger) { … }
```

### Q7. How do you handle circular dependencies?
Best fix: **redesign** (extract a third bean / break the cycle). Workarounds:
constructor → setter/`@Lazy` injection, or `@Lazy` on one dependency. Constructor-only
cycles fail at startup (which is good — it forces a fix).

---

## 2. Spring Boot specifics

### Q8. What problem does Spring Boot solve over plain Spring?
- **Auto-configuration** — sensible defaults based on the classpath.
- **Starters** — curated dependency bundles (`spring-boot-starter-web`).
- **Embedded server** (Tomcat) — runnable JAR, no WAR/external server.
- **Opinionated defaults** + externalized config + Actuator.
**One-liner:** "Convention over configuration — less boilerplate, faster to production."

### Q9. What does `@SpringBootApplication` do?
It's a meta-annotation combining three:
- **`@Configuration`** — this class can define beans.
- **`@EnableAutoConfiguration`** — turn on auto-config.
- **`@ComponentScan`** — scan this package and sub-packages for components.

### Q10. How does auto-configuration work?
Spring Boot reads `META-INF/spring/...AutoConfiguration.imports` and applies config
classes **conditionally** via `@ConditionalOnClass`, `@ConditionalOnMissingBean`,
`@ConditionalOnProperty`, etc. Example: H2 on the classpath → auto-configures a
DataSource. You can override any bean by defining your own.

### Q11. What are starters?
Dependency descriptors that pull a coherent set of libraries. E.g.,
`spring-boot-starter-web` brings Spring MVC, Jackson, embedded Tomcat;
`spring-boot-starter-security` brings Spring Security; `spring-boot-starter-data-jpa`
brings Hibernate + JPA; `spring-boot-starter-mail` brings JavaMail (you added this for
OTP emails).

### Q12. How do you externalize configuration? Precedence?
`application.yml`/`.properties`, env vars, command-line args, profiles. Precedence
(highest first): command-line args → env vars → profile-specific files → default file.
```yaml
spring:
  mail:
    host: ${MAIL_HOST:smtp.gmail.com}   # env override with default
    password: ${MAIL_PASSWORD:}         # secret from env, never hard-coded
```
Bind groups of properties with **`@ConfigurationProperties`**:
```java
@ConfigurationProperties("app.otp")
public record OtpProperties(int length, Duration ttl, int maxAttempts) {}
```

### Q13. Profiles — what and why?
Environment-specific config/beans (`dev`, `test`, `prod`).
```yaml
spring:
  config:
    activate:
      on-profile: prod
```
```java
@Profile("dev") @Bean DataSource embeddedDb() { … }
```
Activate via `--spring.profiles.active=prod` or `SPRING_PROFILES_ACTIVE`.

---

## 3. REST APIs

> **New-to-backend primer — what is a REST API?** It's a set of URL endpoints that
> return/accept **JSON**, using **HTTP verbs** to mean different actions on "resources"
> (users, movies, bookings). You already consume these from Angular with `HttpClient`;
> now you're building the other side. Quick reference you should memorize:
>
> | HTTP verb | Meaning | Example | Typical success code |
> |-----------|---------|---------|----------------------|
> | `GET` | read (never changes data) | `GET /movies` | 200 OK |
> | `POST` | create | `POST /bookings` | 201 Created |
> | `PUT` | replace whole resource | `PUT /movies/5` | 200 OK |
> | `PATCH` | partial update | `PATCH /movies/5` | 200 OK |
> | `DELETE` | remove | `DELETE /movies/5` | 204 No Content |
>
> **Status codes by family:** `2xx` success, `3xx` redirect, `4xx` *you* (client) made a
> mistake (400 bad input, 401 not logged in, 403 logged in but not allowed, 404 not
> found, 409 conflict), `5xx` *the server* broke (500). Returning the *right* code is
> half of good API design — interviewers notice when you say "that should be a 404, not a
> 200 with an error body".
>
> **Idempotent** = calling it repeatedly has the same effect. `GET`, `PUT`, `DELETE` are
> idempotent; `POST` is not (two POSTs create two rows). This matters for safe retries.

### Q14. How do you build a REST controller?
```java
@RestController
@RequestMapping("/api/v1/auth")
public class AuthController {
  private final AuthService auth;
  public AuthController(AuthService auth) { this.auth = auth; }

  @PostMapping("/login")
  public AuthResponse login(@Valid @RequestBody LoginRequest req) {
    return auth.login(req);
  }

  @PostMapping("/register")
  @ResponseStatus(HttpStatus.ACCEPTED)   // 202 — OTP pending
  public RegistrationResponse register(@Valid @RequestBody RegisterRequest req) {
    return auth.register(req);
  }

  @GetMapping("/users/{id}")
  public UserDto get(@PathVariable Long id) { return auth.findUser(id); }
}
```

### Q15. `@RequestParam` vs `@PathVariable` vs `@RequestBody`?
- **`@PathVariable`** — part of the URL path (`/users/{id}`).
- **`@RequestParam`** — query string (`/users?active=true`).
- **`@RequestBody`** — deserialized JSON body (POST/PUT).

### Q16. `@RestController` vs `@Controller`?
`@RestController` = `@Controller` + `@ResponseBody` — return values are serialized
straight to the response body (JSON), no view resolution.

### Q17. How do you return proper status codes?
- `ResponseEntity<T>` for full control (status, headers, body).
- `@ResponseStatus` on methods/exceptions.
```java
return ResponseEntity.status(HttpStatus.CREATED).body(dto);
```

### Q18. How do you validate request data?
Bean Validation (`jakarta.validation`) + `@Valid`:
```java
public record RegisterRequest(
  @NotBlank String name,
  @Email @NotBlank String email,
  @Size(min = 8) String password) {}
```
Invalid input → `MethodArgumentNotValidException` → handle to return 400.

### Q19. Centralized exception handling.
`@RestControllerAdvice` + `@ExceptionHandler` map exceptions to responses globally.
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
  @ExceptionHandler(ConflictException.class)
  public ResponseEntity<ApiError> conflict(ConflictException e) {
    return ResponseEntity.status(HttpStatus.CONFLICT).body(new ApiError(e.getMessage()));
  }
  @ExceptionHandler(IllegalArgumentException.class)
  public ResponseEntity<ApiError> badRequest(IllegalArgumentException e) {
    return ResponseEntity.badRequest().body(new ApiError(e.getMessage()));
  }
}
```
*(This is exactly how your app maps `IllegalArgumentException` → 400 and
`ConflictException` → 409 for OTP/registration errors.)*

### Q20. What is HATEOAS / Richardson Maturity Model? (occasionally asked)
A model of REST maturity: L0 (RPC) → L1 (resources) → L2 (HTTP verbs + status codes) →
L3 (hypermedia/HATEOAS — responses include links). Most APIs target L2.

### Q20b. What's the difference between `@RequestParam` required and optional, and how do you paginate? (very common)
Real APIs almost never return *all* rows — they paginate. Spring Data gives you this for
free with `Pageable`:
```java
@GetMapping("/movies")
public Page<MovieDto> list(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "20") int size,
    @RequestParam(required = false) String genre) {
  return movieService.search(genre, PageRequest.of(page, size));
}
// repository: Page<Movie> findByGenre(String genre, Pageable pageable);
```
**Real-world:** your movie app loads 1000 movies — you'd never send all 1000 at once; you
return `page=0, size=50` and load more on scroll. That's pagination, and it's a
guaranteed follow-up to "how do you build a list endpoint?"

---

## 4. Transactions

### Q21. What does `@Transactional` do?
Wraps a method in a DB transaction — commit on success, **rollback on unchecked
exception**. Spring implements it via an **AOP proxy** around the bean.
```java
@Transactional
public AuthResponse verifyEmail(VerifyOtpRequest req) {
  User user = userRepo.findByEmail(req.email()).orElseThrow();
  otpService.verify(req.email(), req.code());
  user.setEnabled(true);          // dirty checking persists on commit
  return issueTokens(user);
}
```

**Real-world analogy (a bank transfer):** moving ₹100 from account A to B is *two*
steps: subtract from A, add to B. If the app crashes *between* them, money vanishes. A
**transaction** says "do both, or neither." `@Transactional` wraps your method so if
*anything* throws, every DB change rolls back — the accounts look untouched. On success,
all changes commit together. This **all-or-nothing** guarantee is why you put
`@Transactional` on service methods that make multiple related DB writes.

### Q22. The #1 `@Transactional` gotcha — self-invocation.
Because it's a **proxy**, calling a `@Transactional` method **from within the same
class** (`this.method()`) bypasses the proxy → **no transaction**. Fixes: move the
method to another bean, or self-inject the proxy.

```java
@Service
public class OrderService {
  public void checkout(Cart cart) {
    this.saveOrder(cart);   // ❌ calls the RAW method — @Transactional is IGNORED
  }
  @Transactional
  public void saveOrder(Cart cart) { /* multiple DB writes */ }
}
```
**Why this bites beginners:** Spring wraps your bean in a proxy object that adds the
transaction *around* the method. When you call `this.saveOrder()`, you skip the proxy and
call the plain method directly — so no transaction is opened, and a mid-way failure won't
roll back. **Fix:** put `saveOrder` in a *separate* `@Service` and inject it, so the call
goes through the proxy. This is one of the most-asked "gotcha" questions in Spring
interviews precisely because the code *looks* correct.

### Q23. Other `@Transactional` gotchas interviewers love.
- Only **`public`** methods are advised (proxy limitation).
- Default rollback is on **runtime/unchecked** exceptions, **not checked** — use
  `rollbackFor = Exception.class` if needed.
- **Propagation** (`REQUIRED` default, `REQUIRES_NEW`, `NESTED`) and **isolation**
  levels control nested/concurrent behavior.
- A transaction's persistence context closing causes **`LazyInitializationException`**
  if you touch lazy associations afterward (see DB doc).

### Q24. Propagation types (quick).
- **`REQUIRED`** (default) — join existing or start new.
- **`REQUIRES_NEW`** — always a new, independent transaction (suspends current).
- **`NESTED`** — savepoint within the current.
- **`SUPPORTS`, `MANDATORY`, `NEVER`** — situational.

---

## 5. Spring Security & JWT (you built this)

### Q25. How does Spring Security work at a high level?
A **filter chain** intercepts requests. Key pieces: `SecurityFilterChain`,
`AuthenticationManager`, `AuthenticationProvider`, `UserDetailsService`,
`PasswordEncoder`, and the `SecurityContext` holding the authenticated principal.

### Q26. How did you implement JWT auth? (your app)
- **Login** → validate credentials via `AuthenticationManager` → issue a **short-lived
  access token** + **refresh token**.
- A custom **`OncePerRequestFilter`** reads `Authorization: Bearer <token>`, validates
  the JWT signature/expiry, and sets the `SecurityContext`.
- Stateless: `SessionCreationPolicy.STATELESS` (no server session).
```java
@Bean
SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
  http.csrf(csrf -> csrf.disable())
      .sessionManagement(s -> s.sessionCreationPolicy(STATELESS))
      .authorizeHttpRequests(a -> a
          .requestMatchers("/api/v1/auth/**").permitAll()   // login, register, verify-otp, refresh
          .anyRequest().authenticated())
      .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
  return http.build();
}
```

### Q27. Access token vs refresh token — why two?
- **Access token** — short-lived (minutes); sent on every request; if leaked, limited
  blast radius.
- **Refresh token** — long-lived; used **only** to get a new access token; can be
  revoked/rotated. *(Your interceptor calls `/refresh` on a 401 and retries queued
  requests.)*

**Real-world analogy (a hotel):** the **access token** is your room **key card** — it
works for a short time and opens doors, but if you drop it in the lobby, it expires soon
and a thief can't do much. The **refresh token** is your **ID at the front desk** — you
show it to get a *fresh* key card when the old one expires, without re-entering your
password. If your account is compromised, the hotel (server) can invalidate your
front-desk ID (refresh token) so no new key cards are issued. That's why two tokens:
convenience (don't log in every 15 minutes) + security (short-lived keys, revocable
refresh).

### Q28. How are passwords stored?
**Never plaintext.** Hash with a slow, salted algorithm — **BCrypt** via
`BCryptPasswordEncoder`. *(Your app uses this.)*

### Q29. JWT vs session-based auth?
- **JWT** — stateless, scales horizontally, good for APIs/SPAs/microservices; downside:
  hard to revoke before expiry (mitigate with short TTL + refresh rotation).
- **Sessions** — server stores state; easy revocation; needs sticky sessions / shared
  store at scale.

### Q30. How do you do role-based authorization?
```java
.requestMatchers("/api/v1/admin/**").hasRole("ADMIN")
// or method-level:
@PreAuthorize("hasRole('ADMIN')")
public void deleteMovie(Long id) { … }
```

---

## 6. Data access (overlaps with the Hibernate doc)

### Q31. What is Spring Data JPA?
A layer over JPA/Hibernate that generates repository implementations from interfaces.
```java
public interface UserRepository extends JpaRepository<User, Long> {
  Optional<User> findByEmailIgnoreCase(String email);   // derived query
  @Query("select u from User u where u.enabled = true")
  List<User> findEnabled();
}
```
**Derived query methods** parse the method name into SQL.

### Q32. `JpaRepository` vs `CrudRepository` vs `PagingAndSortingRepository`?
Hierarchy: `CrudRepository` (basic CRUD) → `PagingAndSortingRepository` (paging/sort) →
`JpaRepository` (JPA extras: `flush`, batch, `findAll(Sort)`).

---

## 7. Actuator, AOP, async, scheduling

### Q33. What is Spring Boot Actuator?
Production-ready endpoints for monitoring: `/actuator/health`, `/info`, `/metrics`,
`/loggers`, `/env`. Integrates with Prometheus/Micrometer. Secure sensitive endpoints.

### Q34. What is AOP? Where does Spring use it?
**Aspect-Oriented Programming** — modularize **cross-cutting concerns** (logging,
security, transactions) via aspects/advice around join points. Spring uses proxy-based
AOP for `@Transactional`, `@Async`, `@Cacheable`, method security.

### Q35. `@Async` and `@Scheduled`?
```java
@Async public CompletableFuture<Report> generate() { … }   // needs @EnableAsync
@Scheduled(cron = "0 0 2 * * *") public void nightlyCleanup() { … } // @EnableScheduling
```
(`@Async` has the same self-invocation proxy caveat as `@Transactional`.)

### Q36. Caching with `@Cacheable`?
```java
@Cacheable("movies")
public Movie findById(Long id) { … }    // result cached by id
@CacheEvict(value = "movies", key = "#id")
public void update(Long id, Movie m) { … }
```
Needs `@EnableCaching` + a cache provider (Caffeine, Redis).

---

## 8. More questions (beginner-friendly, with real examples)

### Q37. I'm new — walk me through what happens when a request hits `GET /api/movies/5`.
1. The embedded **Tomcat** server receives the HTTP request.
2. The **`DispatcherServlet`** (Spring's front door) looks for a controller method
   mapped to `GET /api/movies/{id}`.
3. Spring extracts `5` into your `@PathVariable Long id`, calls
   `movieController.get(5)`.
4. The controller calls `movieService.findById(5)` (business rules).
5. The service calls `movieRepository.findById(5)` — Spring Data runs
   `SELECT * FROM movies WHERE id = 5`.
6. The `Movie` object flows back up; Spring (Jackson) serializes it to **JSON**.
7. Tomcat sends the JSON response with status `200`.
Knowing this end-to-end flow cold is one of the best things you can show as a junior.

### Q38. What is the `ApplicationContext`?
It's the **container** that holds all your beans — think of it as a big registry Spring
builds at startup by scanning your classes. When a class needs a dependency, Spring
looks it up here and injects it. (Angular's injector tree is the same idea.)

### Q39. What is a DTO and why not just return my entity?
A **DTO** (Data Transfer Object) is a plain object shaped for the API — separate from
your database entity. Why bother?
- **Security:** your `User` entity has `passwordHash`, `enabled`, internal flags — you
  must NOT serialize those to JSON. A `UserDto` exposes only `id`, `name`, `email`.
- **Stability:** you can change the DB schema without breaking the API contract.
- **Avoids lazy-loading bugs** (see the Hibernate file's `LazyInitializationException`).
```java
public record UserDto(Long id, String name, String email) {
  static UserDto from(User u) { return new UserDto(u.getId(), u.getName(), u.getEmail()); }
}
```
**Real-world:** never return password hashes or internal columns — a classic security
review finding.

### Q40. How do I send the right HTTP status codes?
- `200 OK` — success with body.
- `201 Created` — after creating a resource (POST).
- `202 Accepted` — accepted but not finished (your OTP `register` returns this!).
- `204 No Content` — success, nothing to return (DELETE).
- `400 Bad Request` — invalid input (validation failed).
- `401 Unauthorized` — not logged in. `403 Forbidden` — logged in but not allowed.
- `404 Not Found` — resource doesn't exist.
- `409 Conflict` — duplicate/state clash (e.g., email already registered).
- `500 Internal Server Error` — your code threw unexpectedly.
Map these centrally with `@RestControllerAdvice` (see Q19).

### Q41. What is Maven / the `pom.xml`? (the npm of Java)
Maven is the build + dependency tool. `pom.xml` is like `package.json`: it lists
dependencies ("starters"), and Maven downloads them. `mvn package` builds a runnable
JAR (like `npm run build`). `mvn spring-boot:run` starts the app (like `npm start`).

### Q42. What is a "starter" dependency, concretely?
One line in `pom.xml` that pulls a whole working set of libraries. Example:
`spring-boot-starter-web` gives you Spring MVC + Jackson (JSON) + an embedded Tomcat —
so `@RestController` and JSON serialization "just work" with zero extra config.

### Q43. How do I read a config value (like an API key) in my code?
Put it in `application.yml` (or, better, an environment variable for secrets), then
inject it:
```java
@Value("${app.mail.from}")
private String mailFrom;
```
For groups of related settings, bind a whole object with `@ConfigurationProperties`
(your `OtpProperties` does this). **Never hard-code secrets** — use `${ENV_VAR}`.

### Q44. What's the difference between `@RestController` and `@RestControllerAdvice`?
`@RestController` handles requests for *one area* (e.g., movies). `@RestControllerAdvice`
is a **global** helper that catches exceptions thrown by *any* controller and turns them
into clean JSON error responses — so you don't write try/catch in every endpoint.

### Q45. How do I validate incoming data? (real example)
Annotate the request object and add `@Valid`:
```java
public record RegisterRequest(
    @NotBlank String name,
    @Email String email,
    @Size(min = 8, message = "Password must be at least 8 characters") String password) {}

@PostMapping("/register")
public RegistrationResponse register(@Valid @RequestBody RegisterRequest req) { ... }
```
If validation fails, Spring throws automatically → you map it to a `400` with a helpful
message. This is how you stop bad data at the front door.

### Q46. How do I test backend code as a beginner?
- **Unit test a service** with **Mockito** — mock the repository, verify logic:
```java
@Test void rejectsDuplicateEmail() {
  when(userRepo.findByEmail("a@b.com")).thenReturn(Optional.of(existingUser));
  assertThrows(ConflictException.class, () -> service.register(req));
}
```
- **Test a controller** with `@WebMvcTest` + `MockMvc` (no real server needed).
- **Full integration** with `@SpringBootTest` (loads everything, hits a real test DB).
Start with service unit tests — they're fast and teach you the logic.

---

### Rapid-fire
- **`@PathVariable` required by default?** Yes (set `required=false` to relax).
- **What is `CommandLineRunner`?** Run code at startup (e.g., your data seeders).
- **`@Value`?** Inject a single property: `@Value("${app.mail.from}")`.
- **Embedded vs external server?** Boot embeds Tomcat → self-contained JAR.
- **What is `DispatcherServlet`?** Front controller routing HTTP to handlers.
- **CORS in Spring Boot?** `@CrossOrigin` or global `CorsConfigurationSource`.
- **How to test a controller?** `@WebMvcTest` + `MockMvc`; service tests with Mockito;
  full slice with `@SpringBootTest`.
- **What is `@MockBean`?** Replace a bean with a Mockito mock in a Spring test context.

---

### Likely deep-dive chain
> "Constructor vs field injection?" → "Why is `@Transactional` self-invocation broken?"
> → "How does auto-configuration decide?" → "Walk me through your JWT filter" →
> "Access vs refresh token and how you refresh on 401."
