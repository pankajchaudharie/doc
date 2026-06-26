# Resume-Based Interview Questions + STAR Answers

These are the questions an interviewer will ask **straight from your resume**, with
ready-to-speak answers. Interviewers probe resume claims hard ("you said you improved
login by 4 seconds — *how exactly?*"). For each, the answer uses **STAR**
(Situation → Task → Action → Result) and anticipates the follow-up.

> Tailor names/numbers to your actual resume. The technical depth behind each story
> lives in the topic files (Angular, JS/TS, Spring, DB) — cross-reference them.

---

## Your headline story bank (memorize these 6)

| # | Story | Metric | Skills it proves |
|---|-------|--------|------------------|
| 1 | Angular 9 → 19 migration | **+10% Lighthouse** | modernization, perf, risk management |
| 2 | Merlin.net login 7s → 3s | **~57% faster** | caching, profiling, HTTP |
| 3 | Micro-frontend architecture (Wipro) | independent deploys | architecture, scaling teams |
| 4 | Module-based architecture | maintainability | design, lazy loading |
| 5 | AI-powered dev agent | **hours → 5–10 min** | automation, tooling, GitHub Actions |
| 6 | Reusable component/directive/pipe library | DRY, consistency | component design, DX |

---

## 1. The Angular 9 → 19 migration (your strongest story)

### Q: "Walk me through migrating Merlin.net from Angular 9 to 19."
**S:** Merlin.net PCN (a medical/patient app at Abbott) was on Angular 9 — module-heavy,
slow to load, hard to maintain.
**T:** Modernize to Angular 19 and improve the performance score without breaking a
production healthcare app.
**A:**
- Upgraded **incrementally, one major at a time** (`ng update` 9→10→…→19), reading each
  changelog and fixing breaking changes between steps so I never had a giant unstable
  jump.
- Updated third-party libs to Ivy-compatible versions.
- Migrated leaf components to **standalone**, replaced `*ngIf/*ngFor` with the new
  **`@if/@for`** control flow, and introduced **signals** for view state.
- Added **lazy-loaded routes** and **`@defer`** for heavy below-the-fold sections.
- Ran **Lighthouse** before/after to quantify gains, and regression-tested critical
  flows.
**R:** **+10% Lighthouse performance score**, smaller bundles, cleaner DI, and a
codebase ready for future Angular versions.

**Likely follow-ups & answers:**
- *"What broke during the upgrade?"* — typed reactive forms surfaced latent type
  errors; some libs lagged on Ivy; `ModuleWithProviders` needed generics. I handled
  them per-step.
- *"Why one version at a time?"* — Angular only officially supports n→n+1 migrations;
  jumping multiple majors hides which change caused a regression.
- *"How did you avoid regressions in a medical app?"* — feature-flagged, kept e2e/unit
  suites green per step, manual QA on critical journeys.

---

## 2. Login 7s → 3s via caching (your headline metric)

### Q: "How did you cut login time from 7 to 3 seconds?"
**S:** Merlin.net login took ~7s — users waited on repeated network calls and large
assets every load.
**T:** Make login feel fast without changing the backend contract.
**A:**
- **Profiled** with Chrome DevTools/Lighthouse to find the bottlenecks (waterfall of
  redundant API calls + large JS/JSON assets).
- **Cached idempotent GET API responses** (in memory / `localStorage` with a TTL) via an
  HTTP interceptor, serving repeat reads instantly.
- **Cached static JSON/JS assets** with proper HTTP cache headers / versioned URLs so
  the browser reused them.
- Reduced redundant calls and deferred non-critical work.
**R:** Login dropped to **~3s (≈57% faster)**, measurably better UX.

**Likely follow-ups:**
- *"How do you invalidate the cache?"* — TTL expiry, versioned asset URLs
  (`app.123.js`), and cache-busting on logout/data mutations. Never cache
  authentication-sensitive responses.
- *"Risk of stale data?"* — only cache idempotent, non-sensitive GETs; short TTL for
  semi-dynamic data; bypass cache on writes.
- *"Why not just a service worker?"* — could; for assets a SW/HTTP cache works, for API
  responses an interceptor gave finer control and TTL logic.

---

## 3. Micro-frontends (Wipro)

### Q: "You mention micro-frontends — what did you build and why?"
**S:** A large app at Wipro where multiple teams stepped on each other in one monolith
frontend.
**T:** Let teams build, test, and deploy features independently, then compose them.
**A:** Split the app into **independently buildable/deployable micro-frontends**
integrated into a host/shell (Module Federation approach). Standardized shared
dependencies (one Angular/RxJS instance) and isolated styles to avoid collisions.
**R:** Teams shipped independently with smaller, testable units and faster releases.

**Follow-ups:**
- *"Trade-offs?"* — dependency/version coordination, larger total payload, cross-app
  state & routing complexity; worth it only at sufficient scale/team size.
- *"How did you share state across MFEs?"* — minimal shared singletons / a shared
  service or custom events; avoid tight coupling.
- *"How did you avoid style leakage?"* — Angular `ViewEncapsulation` + naming
  conventions/prefixes.

---

## 4. Module-based architecture

### Q: "What do you mean by module-based architecture?"
**A:** Organizing the app into **feature modules** (+ shared/core modules) with
**lazy-loaded routes**, so each feature is encapsulated, independently testable, and
only loaded when needed — improving maintainability and initial load.
**Follow-up — "How does standalone change this?"** Standalone components remove the
`NgModule` boilerplate; you still organize by feature folders and lazy-load via
`loadComponent`/`loadChildren`. I migrated toward standalone in the 19 upgrade.

---

## 5. The AI-powered developer agent (your differentiator)

### Q: "Tell me about the AI agent that cut work from hours to minutes."
**S:** Repetitive dev tasks (reading Jira tickets, scaffolding code, wiring Git
workflows) ate hours.
**T:** Automate the boilerplate so engineers focus on real logic.
**A:** Built an **AI-powered agent** that **analyzes Jira tickets**, **generates
code/scaffolding**, and integrates with **GitHub Actions** to automate parts of the
pipeline. (Be concrete about *your* role: prompt/flow design, the GitHub Actions
integration, validation/guardrails.)
**R:** Reduced certain tasks from **hours to ~5–10 minutes**.

**Follow-ups:**
- *"How did you ensure correctness of generated code?"* — human-in-the-loop review,
  automated tests in CI, lint/build gates before merge.
- *"Where did GitHub Actions fit?"* — triggered analysis/generation on events, ran
  build/test, opened PRs/artifacts. (See [04-frontend-github-actions-ci.md](04-frontend-github-actions-ci.md).)
- *"What were the risks?"* — hallucinated/insecure code → mitigated with tests, review,
  least-privilege tokens, no secrets in prompts.

---

## 6. Auth / route guards / interceptors (DXC + your recent app)

### Q: "How have you handled authentication and route protection?"
**A:** Implemented **route guards** (`CanActivate`/functional `authGuard`) to block
unauthenticated access and redirect to login with a `redirect` query param, plus an
**HTTP interceptor** that attaches the JWT `Bearer` token and handles `401` by
**refreshing the token once** and retrying queued requests (single-flight refresh).
**Concrete proof:** "In a recent movie-booking app I built end-to-end, I implemented JWT
access + rotating refresh tokens, a stateless Spring Security filter chain, BCrypt
hashing, and even **email-OTP signup verification**." (See
[01-angular.md](01-angular.md) §guards/interceptors and [06-spring-boot.md](06-spring-boot.md) §security.)

---

## 7. RESTful API integration (Wipro)

### Q: "How do you integrate REST APIs in Angular?"
**A:** `HttpClient` with typed responses, RxJS for orchestration (`switchMap` for
type-ahead, `forkJoin` for parallel loads), interceptors for auth/error/caching,
`catchError` for graceful fallbacks, and DTO interfaces for type safety. Used all HTTP
verbs (GET/POST/PUT/PATCH/DELETE) for full CRUD.

---

## 8. Reusable components / directives / pipes

### Q: "Give an example of a reusable component you built."
**A:** Built a shared UI library — e.g., a configurable data-table/card component
(content projection + `@Input` config), custom **directives** (e.g., a permission/
`hasRole` directive, click-outside, autofocus), and custom **pipes** (formatting,
filtering). This enforced consistency, cut duplication, and sped up feature teams.
**Follow-up — "pure vs impure pipe?"** I kept pipes **pure** for performance; used
impure only when unavoidable.

---

## 9. State management (NgRx / RxJS)

### Q: "When did you use NgRx vs a simple service?"
**A:** NgRx for **shared, complex state** needing predictability/debugging; a service
with `BehaviorSubject` (or signals now) for **local/simple** state. I avoid
over-engineering — not every app needs a global store. (Full data-flow in
[01-angular.md](01-angular.md) §NgRx.)

---

## 10. UI libraries (Angular Material / PrimeNG)

### Q: "Which component libraries have you used?"
**A:** **Angular Material** (theming, CDK — virtual scroll, overlays) and **PrimeNG**
(rich data components). I choose based on design system needs and customize via
theming, always watching bundle size.

---

## 11. Docker / deployment

### Q: "How were your apps deployed?"
**A:** Containerized the Angular build (multi-stage **Docker**: build with Node, serve
static files via Nginx) and automated **build → test → deploy** with **GitHub Actions**.
(See [04-frontend-github-actions-ci.md](04-frontend-github-actions-ci.md).)
```dockerfile
# multi-stage example
FROM node:20 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build -- --configuration production

FROM nginx:alpine
COPY --from=build /app/dist/my-app/browser /usr/share/nginx/html
```

---

## Behavioral / experience questions (prepare these too)

### Q: "You're a frontend dev — why move to backend / full-stack?"
**A:** "I want to own features end-to-end and understand the systems my UIs depend on.
I've been learning by **building a real full-stack app** — Spring Boot + JPA/Hibernate
+ JWT auth + email OTP — not just reading. It makes me a better frontend engineer too:
I design better API contracts and reason about performance across the stack."

### Q: "Tell me about a challenging bug."
Use the **OTP/refresh-token single-flight** story or the **N+1 / lazy-loading** story —
concrete, technical, shows debugging method (reproduce → isolate → fix → verify).

### Q: "How do you keep up with Angular's fast pace?"
**A:** Following the Angular blog/changelogs, the migration itself (9→19) forced
hands-on learning of signals, standalone, control flow, `@defer`; plus side projects.

### Q: "Biggest impact you've had?"
Lead with the **migration (+10% Lighthouse)** or **login 7s→3s** — quantified, user-
facing, ownership.

### Q: "A time you disagreed with a technical decision?"
Pick a real one (e.g., pushing back on adding NgRx to a simple app, or on a risky
big-bang upgrade vs incremental). Show you reason with trade-offs and data, and commit
once decided.

---

## Questions YOU should ask them (always have 3–4)
- "How is the frontend architected today — modules vs standalone, any micro-frontends?"
- "What does your CI/CD and release process look like?"
- "Where are you on adopting signals / zoneless / SSR?"
- "How is frontend/backend collaboration structured — do engineers own full features?"

---

### Final tip
For **every** resume bullet, be able to answer: **"What exactly did *you* do, why, and
what was the measurable result?"** Vague claims collapse under follow-ups; specifics
(numbers, tools, trade-offs) build trust.
