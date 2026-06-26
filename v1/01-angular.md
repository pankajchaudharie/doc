# Angular Interview Questions (v9 → v19) — Deep Dive

Tailored for someone who worked on **Angular 9 (module-based)** and migrated to
**Angular 19 (standalone + signals)**. Interviewers love this exact journey because
it tests both the "classic" framework and the modern one.

---

## Table of contents
1. Fundamentals & architecture
2. Components, templates & data binding
3. Modules vs Standalone
4. Dependency Injection
5. Change Detection & Zone.js
6. Signals (v16+)
7. RxJS & Observables
8. Routing & Guards
9. Forms (Template-driven & Reactive)
10. HTTP & Interceptors
11. NgRx / State management
12. Performance optimization
13. Micro-frontends
14. Testing (Jasmine/Karma)
15. Angular 9 vs 19 — what actually changed
16. Rapid-fire

---

## 1. Fundamentals & architecture

### Q1. What is Angular and how is it different from AngularJS / React?
**Answer.** Angular is a **TypeScript-based, opinionated SPA framework** by Google. It
ships batteries-included: DI, router, forms, HttpClient, RxJS, testing, CLI.

- **vs AngularJS (1.x):** complete rewrite — component-based instead of
  `$scope`/controllers, TypeScript, AOT compilation, hierarchical DI, mobile-first.
- **vs React:** React is a *library* (view layer) — you assemble routing/state/HTTP
  yourself. Angular is a *framework* — conventions + tooling out of the box. Angular
  uses real DOM + change detection; React uses a virtual DOM + reconciliation.

**Why interviewers ask:** to see if you understand "framework vs library" trade-offs.

---

### Q2. Explain Angular's architecture / building blocks.
- **Modules** (`NgModule`) — *(classic)* grouping mechanism; **Standalone components** *(modern)* remove this need.
- **Components** — a class + template + styles; the unit of UI.
- **Templates** — HTML with Angular syntax (bindings, directives, control flow).
- **Directives** — attribute (`ngClass`) and structural (`*ngIf`, `*ngFor`).
- **Services** — reusable logic/state, provided via DI.
- **Dependency Injection** — supplies services to classes.
- **Pipes** — transform values in templates (`date`, `async`, custom).
- **Router** — maps URLs to components.

---

### Q3. What is AOT vs JIT compilation?
- **JIT (Just-in-Time):** templates compiled **in the browser at runtime**. Faster
  builds, larger bundle, used historically in `ng serve`.
- **AOT (Ahead-of-Time):** templates compiled **at build time**. Smaller/faster,
  template errors caught at build, no Angular compiler shipped to the browser.
  **Default for production since Angular 9** (Ivy made AOT the default everywhere).

**Follow-up:** *Why is AOT safer?* Template type-checking happens at build, so
`{{ user.naem }}` fails the build instead of silently rendering nothing.

---

### Q4. What is Ivy?
The **rendering/compilation engine** that became default in **Angular 9**. Benefits:
- **Smaller bundles** via tree-shaking (locality principle — code is generated per
  component, unused features are dropped).
- **Faster compilation & better debugging** (readable generated code).
- Enabled later features like **standalone components** and improved
  **incremental builds**.

> Resume tie-in: your **Angular 9 → 19 migration** rode on top of Ivy; mention that
> Ivy's tree-shaking is part of why bundle size / Lighthouse score improved.

---

## 2. Components, templates & data binding

### Q5. Explain the four types of data binding.
```html
<!-- 1. Interpolation (component → view) -->
<h1>{{ title }}</h1>

<!-- 2. Property binding (component → view) -->
<img [src]="imageUrl" />

<!-- 3. Event binding (view → component) -->
<button (click)="save()">Save</button>

<!-- 4. Two-way binding (sugar for property + event) -->
<input [(ngModel)]="name" />
<!-- equals: -->
<input [value]="name" (input)="name = $event.target.value" />
```
`[(ngModel)]` is the "banana in a box" — `[()]` = property binding `[ ]` + event `( )`.

---

### Q6. What are the component lifecycle hooks (in order)?
| Hook | When | Typical use |
|------|------|-------------|
| `ngOnChanges` | on `@Input()` change (before `ngOnInit`) | react to input changes |
| `ngOnInit` | once, after first `ngOnChanges` | init logic, fetch data |
| `ngDoCheck` | every CD run | custom change detection |
| `ngAfterContentInit` | after `<ng-content>` projected | — |
| `ngAfterContentChecked` | after projected content checked | — |
| `ngAfterViewInit` | after view + child views init | access `@ViewChild`, DOM |
| `ngAfterViewChecked` | after view checked | — |
| `ngOnDestroy` | before component destroyed | **unsubscribe, clear timers** |

```ts
export class TimerComponent implements OnInit, OnDestroy {
  private sub?: Subscription;
  ngOnInit() { this.sub = interval(1000).subscribe(/*…*/); }
  ngOnDestroy() { this.sub?.unsubscribe(); } // prevent memory leaks
}
```
**Gotcha they probe:** memory leaks from not unsubscribing in `ngOnDestroy`
(or use `takeUntilDestroyed()` / the `async` pipe instead).

---

### Q7. `@ViewChild` vs `@ContentChild`?
- **`@ViewChild`** queries elements/components in **this component's own template**.
- **`@ContentChild`** queries elements **projected via `<ng-content>`** from a parent.

```ts
@ViewChild('search') searchInput!: ElementRef;     // in my template
@ContentChild(TabComponent) firstTab!: TabComponent; // passed into me
```

---

### Q8. What is content projection (`ng-content`)?
A way to build reusable wrapper components by passing markup into them — like React
`children`. Supports **multi-slot** projection with `select`:
```html
<!-- card.component.html -->
<div class="card">
  <header><ng-content select="[card-title]"></ng-content></header>
  <section><ng-content></ng-content></section>
</div>

<!-- usage -->
<app-card>
  <h2 card-title>Title</h2>
  <p>Body content</p>
</app-card>
```

---

### Q9. ViewEncapsulation — what are the modes?
Controls how component CSS is scoped:
- **`Emulated`** (default) — Angular adds attributes (`_ngcontent-xxx`) so styles
  don't leak. Not true Shadow DOM but behaves like it.
- **`ShadowDom`** — real browser Shadow DOM; full isolation.
- **`None`** — styles become **global** (no scoping). Useful for theming, risky for leaks.

---

## 3. Modules vs Standalone

### Q10. What is an `NgModule` and why did it exist? (Angular 9 world)
`@NgModule` groups components/directives/pipes (`declarations`), other modules
(`imports`), services (`providers`), and the root component (`bootstrap`). It told
the compiler *what's available in templates* and configured DI.

```ts
@NgModule({
  declarations: [AppComponent, UserListComponent],
  imports: [BrowserModule, HttpClientModule, RouterModule.forRoot(routes)],
  providers: [UserService],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

### Q11. What are standalone components (Angular 14+, default in 19)?
Components that declare their **own dependencies via `imports`** and don't need an
`NgModule`. Less boilerplate, clearer dependency graph, better tree-shaking, simpler
lazy loading.
```ts
@Component({
  selector: 'app-user',
  standalone: true,                 // (implicit/default in v19)
  imports: [CommonModule, RouterLink, MatButtonModule],
  templateUrl: './user.component.html',
})
export class UserComponent {}
```
**Bootstrapping changed:**
```ts
// main.ts (standalone)
bootstrapApplication(AppComponent, {
  providers: [provideRouter(routes), provideHttpClient()],
});
```
**Why interviewers ask (given your migration):** "How did you move from modules to
standalone?" → incrementally: standalone components can be `imported` into existing
`NgModule`s, so you migrate leaf components first, then routes, then drop modules.

---

## 4. Dependency Injection

### Q12. How does Angular DI work?
Angular has a **hierarchical injector tree**. When a class asks for a dependency in
its constructor, Angular walks up the injector hierarchy to find a provider.

```ts
@Injectable({ providedIn: 'root' }) // tree-shakable singleton, app-wide
export class AuthService {}

@Component({ providers: [LocalService] }) // new instance per component
export class WidgetComponent {
  constructor(private auth: AuthService, private local: LocalService) {}
  // modern alternative:
  private auth2 = inject(AuthService);
}
```

### Q13. `providedIn: 'root'` vs providing in a component?
- **`root`** — one **singleton** for the whole app; tree-shaken away if unused.
- **Component `providers`** — a **new instance per component instance** (and its
  children). Useful for per-widget state.

### Q14. What injection tokens / resolution modifiers exist?
- **`InjectionToken`** — DI key for non-class values (config objects, strings).
- **`@Optional()`** — don't throw if missing (returns `null`).
- **`@Self()` / `@SkipSelf()`** — control where in the tree to look.
- **`@Host()`** — stop at the host component.

```ts
export const API_URL = new InjectionToken<string>('API_URL');
providers: [{ provide: API_URL, useValue: 'https://api.example.com' }];
constructor(@Inject(API_URL) private url: string) {}
```

### Q15. `useClass` / `useValue` / `useExisting` / `useFactory`?
```ts
providers: [
  { provide: Logger, useClass: ProdLogger },        // swap implementation
  { provide: API_URL, useValue: '/api' },            // constant
  { provide: OldApi, useExisting: NewApi },          // alias
  { provide: Http, useFactory: (c) => new Http(c), deps: [Config] }, // dynamic
];
```

---

## 5. Change Detection & Zone.js

### Q16. How does Angular change detection work?
Angular keeps a **tree of components**. On an async event (click, XHR, timer),
**Zone.js** notifies Angular, which runs CD **top-down**, comparing each binding's
current vs previous value and updating the DOM where changed.

### Q17. `Default` vs `OnPush` change detection?
- **`Default`** — checks the component on **every** CD cycle.
- **`OnPush`** — checks only when:
  1. an `@Input()` **reference** changes,
  2. an event fires **inside** the component,
  3. an observable bound via `async` emits, or
  4. you call `markForCheck()`.

```ts
@Component({ changeDetection: ChangeDetectionStrategy.OnPush })
```
**Why it matters (perf):** `OnPush` + immutable data drastically cuts CD work. This is
a common senior-level performance answer.

```ts
// With OnPush, mutate-in-place is NOT detected:
this.user.name = 'X';                 // ❌ same reference, no update
this.user = { ...this.user, name: 'X' }; // ✅ new reference, detected
```

**Real-world example:** a dashboard rendering a list of 500 stock-price cards. With
`Default`, *every* card re-checks its bindings on *every* mouse move or timer tick —
noticeable lag. Switch the card component to `OnPush` and feed it immutable data, and a
card only re-renders when *its own* `@Input()` price object reference changes. This is
exactly the kind of fix behind your Lighthouse/performance story — "I moved hot list
components to OnPush so change detection stopped re-checking thousands of unchanged
bindings."

### Q18. What is Zone.js and why is Angular moving away from it?
Zone.js **monkey-patches** async APIs (`setTimeout`, `addEventListener`, `Promise`,
XHR) so Angular knows when to run CD. Downsides: overhead, patches everything, hard to
reason about. **Angular 18+ offers zoneless change detection** (experimental →
stabilizing) where **signals** drive updates directly, removing Zone.js.

### Q19. How do you debug a "too many change detection cycles" / `ExpressionChangedAfterItHasBeenCheckedError`?
It means a value **changed after** Angular already checked it in the same cycle
(usually changing state in `ngAfterViewInit` or a getter with side effects). Fixes:
move the change to `ngOnInit`, use `setTimeout`/`Promise.resolve`, or restructure so
bindings are stable during a cycle. (Dev-mode-only error; double-checks catch bugs.)

---

## 6. Signals (Angular 16+, central in 19)

### Q20. What are signals and why were they added?
A **signal** is a reactive wrapper around a value that **notifies consumers when it
changes**, enabling **fine-grained, synchronous** reactivity **without Zone.js**.

```ts
const count = signal(0);          // writable signal
const double = computed(() => count() * 2); // derived, memoized
effect(() => console.log('count is', count())); // side effect on change

count.set(5);          // replace
count.update(c => c + 1); // derive from current
```
- **Why:** simpler mental model than RxJS for component state, better performance
  (only affected views update), and the path to **zoneless** Angular.

### Q21. Signals vs RxJS — when to use which?
- **Signals:** synchronous **UI state** (counters, form view-state, derived values).
- **RxJS:** **async streams / events over time** (HTTP, websockets, debounced search,
  complex orchestration). Convert between them with `toSignal()` / `toObservable()`.
```ts
readonly user = toSignal(this.http.get<User>('/me'), { initialValue: null });
```

### Q22. `signal` input/output and `model()` (v17.1+/19)?
```ts
// Signal-based inputs/outputs replace @Input/@Output decorators:
value = input.required<string>();   // required input as a signal
count = input(0);                   // optional with default
changed = output<string>();        // typed output
twoWay = model(0);                  // two-way bindable signal
```

---

## 7. RxJS & Observables

### Q23. Observable vs Promise?
| | Observable | Promise |
|--|-----------|---------|
| Values | **multiple** over time | single |
| Lazy | yes (runs on subscribe) | eager |
| Cancel | yes (`unsubscribe`) | no |
| Operators | rich (`map`, `filter`, …) | `.then` chains |

### Q24. Explain the most-used RxJS operators.
- **Transformation:** `map`, `switchMap`, `mergeMap`, `concatMap`, `exhaustMap`.
- **Filtering:** `filter`, `take`, `takeUntil`, `debounceTime`, `distinctUntilChanged`.
- **Combination:** `combineLatest`, `forkJoin`, `withLatestFrom`, `merge`.
- **Error:** `catchError`, `retry`, `retryWhen`.

### Q25. `switchMap` vs `mergeMap` vs `concatMap` vs `exhaustMap` — THE classic question.
- **`switchMap`** — cancels the previous inner observable. **Best for type-ahead
  search** (you only care about the latest).
- **`mergeMap`** — runs all in parallel, no order guarantee. Good for independent writes.
- **`concatMap`** — queues, preserves order, one at a time. Good for ordered writes.
- **`exhaustMap`** — ignores new emissions while one is in flight. **Best for login
  button spam** (ignore clicks until current request finishes).

```ts
this.searchControl.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(term => this.api.search(term)) // cancels stale requests
).subscribe(results => this.results = results);
```

**Real-world example for each (so you can tell a story, not just define):**
- **`switchMap`** — a movie **search box**. User types "aveng" then "avengers". The
  request for "aveng" is *cancelled* the moment "avengers" is typed, so stale results
  never flash on screen. (Your movie app's search would use this.)
- **`exhaustMap`** — a **"Book ticket" / login button**. If the user double-clicks, you
  *ignore* the second click while the first request is in flight, preventing double
  bookings/charges.
- **`concatMap`** — saving **drag-reorder of a playlist**: each reorder must hit the
  server *in order*, so you queue them one after another.
- **`mergeMap`** — firing **independent analytics events**: order doesn't matter, run
  them all in parallel.

### Q26. How do you avoid memory leaks with subscriptions?
1. **`async` pipe** (auto-unsubscribes) — preferred.
2. **`takeUntilDestroyed()`** (v16+) or `takeUntil(this.destroy$)`.
3. Manual `unsubscribe()` in `ngOnDestroy`.

```ts
constructor() {
  this.api.poll().pipe(takeUntilDestroyed()).subscribe(/*…*/);
}
```

### Q27. `Subject` vs `BehaviorSubject` vs `ReplaySubject`?
- **`Subject`** — no initial value; emits only to current subscribers.
- **`BehaviorSubject`** — holds **current value**; new subscribers get the latest
  immediately. Great for **state** (`isLoggedIn$`).
- **`ReplaySubject`** — replays the last N values to new subscribers.

### Q28. Hot vs Cold observables?
- **Cold** — produces data **per subscriber** (e.g., `http.get` fires once per
  subscribe). 
- **Hot** — shares one execution among subscribers (e.g., `Subject`, DOM events,
  or a cold source piped through `share()`/`shareReplay()`).

---

## 8. Routing & Guards

### Q29. How does Angular routing work? Lazy loading?
```ts
export const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'users/:id', component: UserComponent },
  // lazy-loaded standalone component:
  { path: 'admin', loadComponent: () => import('./admin/admin.component')
      .then(m => m.AdminComponent) },
  // lazy-loaded route group:
  { path: 'reports', loadChildren: () => import('./reports/routes')
      .then(m => m.REPORTS_ROUTES) },
  { path: '**', redirectTo: '' }, // wildcard
];
```
**Lazy loading** splits these into separate JS chunks loaded on demand → smaller
initial bundle, faster first load. *(Directly relevant to your Lighthouse work.)*

### Q30. Explain route guards. (You used `AuthGuard` at DXC.)
Guards decide if navigation can proceed. Modern Angular uses **functional guards**:
```ts
export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  const router = inject(Router);
  return auth.isLoggedIn() ? true
    : router.createUrlTree(['/login'], { queryParams: { redirect: state.url } });
};
```
Guard types: **`CanActivate`** (enter route), **`CanActivateChild`**,
**`CanDeactivate`** (leave — "unsaved changes?"), **`CanMatch`** (whether a route
config even matches — great for feature flags/role-based lazy modules),
**`Resolve`** (prefetch data before activating).

### Q31. `Resolver` — what and why?
Fetches data **before** the route activates so the component renders with data ready
(no flicker/empty state). Trade-off: delays navigation until data arrives.

### Q32. Route parameters: `snapshot` vs observable?
- `route.snapshot.paramMap.get('id')` — one-time read (fine if component is recreated).
- `route.paramMap.subscribe(...)` — needed when navigating **between** the same
  component (e.g., `/users/1` → `/users/2`) where the component is **reused**.

---

## 9. Forms

### Q33. Template-driven vs Reactive forms?
| | Template-driven | Reactive |
|--|----------------|----------|
| Where logic lives | template (`ngModel`) | component (`FormGroup`) |
| Best for | simple forms | complex/dynamic, testable forms |
| Validation | directives | functions in TS |
| Async/dynamic | harder | easy |

```ts
// Reactive
form = this.fb.group({
  email: ['', [Validators.required, Validators.email]],
  password: ['', [Validators.required, Validators.minLength(8)]],
});
get email() { return this.form.controls.email; }
```
```html
<input [formControl]="email" />
@if (email.touched && email.hasError('required')) { <span>Required</span> }
```

### Q34. How do custom validators work (sync + async)?
```ts
// sync
export function noSpaces(c: AbstractControl): ValidationErrors | null {
  return c.value?.includes(' ') ? { noSpaces: true } : null;
}
// async (e.g., unique email check)
export function uniqueEmail(api: Api): AsyncValidatorFn {
  return c => api.exists(c.value).pipe(map(exists => exists ? { taken: true } : null));
}
```

### Q35. Typed reactive forms (Angular 14+)?
Forms are now **strictly typed** by default — `form.value` infers types, preventing a
class of runtime bugs. Use `nonNullable` for cleaner types/reset behavior.

---

## 10. HTTP & Interceptors

### Q36. How does `HttpClient` work and how do you handle errors?
```ts
this.http.get<User[]>('/api/users').pipe(
  retry(2),
  catchError(err => {
    this.notify.error('Failed to load users');
    return of([]); // graceful fallback
  })
).subscribe(users => this.users = users);
```

### Q37. What is an HTTP interceptor? (You'll be asked since you used JWT.)
A middleware that intercepts every request/response — perfect for **auth tokens,
logging, error handling, caching, retry, loading spinners**. Modern functional form:
```ts
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(AuthService).accessToken;
  const authReq = token
    ? req.clone({ setHeaders: { Authorization: `Bearer ${token}` } })
    : req;
  return next(authReq).pipe(
    catchError(err => err.status === 401 ? handle401(req, next) : throwError(() => err))
  );
};
// register: provideHttpClient(withInterceptors([authInterceptor]))
```

### Q38. How do you implement a refresh-token flow in an interceptor?
On a `401`, pause new requests, call `/refresh` **once**, queue concurrent failed
requests, then retry them with the new token. Use a module-level `isRefreshing` flag +
a `BehaviorSubject` to coordinate the queue (classic concurrency gotcha — interviewers
love this). *(You implemented exactly this in the movie-booking app.)*

### Q39. How did you make login faster by caching API responses? (Resume!)
- Cache **idempotent GET** responses in memory / `localStorage` with a TTL via an
  interceptor; serve from cache on repeat.
- Cache **static JSON/JS assets** with HTTP cache headers / service worker.
- Result on Merlin.net: **7s → 3s**. Be ready to explain cache invalidation
  (TTL, versioned URLs, `Cache-Control`).

---

## 11. NgRx / State management

### Q40. When do you actually need NgRx?
When state is **shared across many components**, **complex**, needs **time-travel
debugging / predictability**, or **caching of server state**. For simple apps, a
service with a `BehaviorSubject` (or signals) is enough. *Don't over-engineer* — a
great senior answer.

### Q41. Explain the NgRx data flow.
**Component → dispatches `Action` → `Reducer` (pure, updates `Store`) →
`Selector` (derives slices) → Component.** Side effects (HTTP) live in **`Effects`**.
```
Action ──▶ Effect ──▶ (HTTP) ──▶ success Action ──▶ Reducer ──▶ Store ──▶ Selector ──▶ View
```
```ts
// action
export const loadUsers = createAction('[Users] Load');
export const loadUsersSuccess = createAction('[Users] Load Success', props<{ users: User[] }>());
// reducer
const reducer = createReducer(initialState,
  on(loadUsers, s => ({ ...s, loading: true })),
  on(loadUsersSuccess, (s, { users }) => ({ ...s, loading: false, users })));
// effect
loadUsers$ = createEffect(() => this.actions$.pipe(
  ofType(loadUsers),
  switchMap(() => this.api.getUsers().pipe(
    map(users => loadUsersSuccess({ users })),
    catchError(() => of(loadUsersFailure())))));
// selector
export const selectUsers = createSelector(selectUserState, s => s.users);
```

### Q42. Why must reducers be pure?
Predictability, testability, and time-travel debugging require **no side effects** and
**same input → same output**. Side effects go in Effects.

### Q43. NgRx ComponentStore / SignalStore?
- **ComponentStore** — local, lightweight store scoped to a component (no global boilerplate).
- **SignalStore (NgRx 17+)** — signal-based store; the modern direction.

---

## 12. Performance optimization (your specialty — be strong here)

### Q44. How do you optimize an Angular app? (Lighthouse story)
1. **Change detection:** `OnPush` + immutable data; `trackBy` in `*ngFor`.
2. **Bundle size:** lazy load routes, tree-shakable providers, drop unused deps,
   analyze with `source-map-explorer` / `webpack-bundle-analyzer`.
3. **Network:** cache API responses (your 7s→3s), HTTP caching, `gzip`/`brotli`, CDN,
   preload critical routes.
4. **Rendering:** `@defer` (v17+) for below-the-fold, virtual scrolling
   (`cdk-virtual-scroll`) for long lists, image lazy loading + `NgOptimizedImage`.
5. **SSR / hydration** (Angular Universal + non-destructive hydration v16+) for faster
   First Contentful Paint.
6. **Build:** AOT (default), production budgets, differential loading.

```html
<!-- trackBy avoids re-rendering the whole list -->
<div *ngFor="let item of items; trackBy: trackById">{{ item.name }}</div>
```
```ts
trackById = (_: number, item: Item) => item.id;
```

### Q45. What is `@defer` (deferrable views, v17)?
Declaratively **lazy-load a part of a template** with triggers and placeholders —
huge for performance without manual code-splitting:
```html
@defer (on viewport) {
  <app-heavy-chart [data]="data" />
} @placeholder { <div>Loading chart…</div> } @loading (after 100ms) { <spinner/> }
```

### Q46. What is `NgOptimizedImage`?
A directive (`ngSrc`) that enforces image best practices: lazy loading, responsive
`srcset`, priority hints for LCP images, preventing layout shift.

### Q47. SSR + hydration — what problem does it solve?
SSR renders HTML on the server for **faster FCP and SEO**; **non-destructive
hydration** (v16+) reuses the server DOM instead of re-rendering, avoiding flicker.

---

## 13. Micro-frontends (you did this at Wipro)

### Q48. What is a micro-frontend and how did you implement it?
Splitting a large app into **independently developed, built, and deployed** smaller
apps, composed at runtime. Common approach: **Webpack Module Federation** — a "shell"
loads remote bundles exposed by feature apps.
- **Pros:** team autonomy, independent deploys, tech isolation, smaller builds.
- **Cons:** shared-dependency/versioning complexity, larger total payload, cross-app
  state/styling coordination, routing integration.
- Be ready to discuss **sharing singletons** (e.g., one Angular/RxJS instance) and
  **avoiding style collisions** (`ViewEncapsulation`, prefixes).

---

## 14. Testing

### Q49. How do you unit test an Angular component? (Jasmine + Karma)
```ts
describe('UserComponent', () => {
  let fixture: ComponentFixture<UserComponent>;
  let apiSpy: jasmine.SpyObj<Api>;

  beforeEach(() => {
    apiSpy = jasmine.createSpyObj('Api', ['getUser']);
    TestBed.configureTestingModule({
      imports: [UserComponent], // standalone
      providers: [{ provide: Api, useValue: apiSpy }],
    });
    fixture = TestBed.createComponent(UserComponent);
  });

  it('loads the user on init', () => {
    apiSpy.getUser.and.returnValue(of({ id: 1, name: 'Ann' }));
    fixture.detectChanges();           // triggers ngOnInit
    expect(fixture.componentInstance.user?.name).toBe('Ann');
  });
});
```
- **`TestBed`** configures a testing module. **`ComponentFixture`** wraps the component.
- **`fakeAsync`/`tick`** for time-based async; **`HttpTestingController`** for HTTP.
- Mock dependencies with **spies** to isolate the unit.

### Q50. How do you test observables / async code?
```ts
it('debounces search', fakeAsync(() => {
  component.search('a');
  tick(300);                 // advance virtual time
  expect(apiSpy.search).toHaveBeenCalledWith('a');
}));
```

---

## 15. Angular 9 vs 19 — what actually changed (your migration narrative)

| Area | Angular 9 | Angular 19 |
|------|-----------|-----------|
| Engine | Ivy introduced (default) | Ivy mature |
| Modules | `NgModule` required | **Standalone by default**, modules optional |
| Bootstrap | `platformBrowserDynamic().bootstrapModule(AppModule)` | `bootstrapApplication(AppComponent, {...})` |
| Reactivity | RxJS + Zone.js only | **Signals**, optional **zoneless** CD |
| Control flow | `*ngIf`, `*ngFor`, `*ngSwitch` | built-in **`@if` / `@for` / `@switch`** |
| Lazy template | manual | **`@defer`** blocks |
| Inputs/Outputs | `@Input()/@Output()` | **`input()/output()/model()`** signal APIs |
| Hydration | destructive SSR | **non-destructive hydration** |
| Images | manual | **`NgOptimizedImage`** |
| HTTP/Router providers | modules (`HttpClientModule`) | `provideHttpClient()`, `provideRouter()` |

**How you'd describe the migration (STAR-ready):**
> *Situation:* Merlin.net PCN on Angular 9, slow, module-heavy.
> *Task:* modernize to 19, improve performance.
> *Action:* upgraded version-by-version (`ng update`), fixed breaking changes per
> changelog, migrated leaf components to **standalone**, replaced `*ngIf/*ngFor` with
> **`@if/@for`**, introduced **signals** for view state, added **`@defer`** + lazy
> routes, used **`NgOptimizedImage`**, cached API/asset responses.
> *Result:* **+10% Lighthouse**, login **7s→3s**, cleaner DI, smaller bundles.

### Q51. What's the safe upgrade process?
- Use **`ng update`** (one major at a time — 9→10→…→19), follow the **Angular Update
  Guide**, run tests between steps, update third-party libs, fix deprecations, verify
  with Lighthouse/e2e.

### Q52. New built-in control flow — why is it better than `*ngIf`?
`@if/@for/@switch` are **built into the compiler** (no `CommonModule` import), have
better type-narrowing, are **faster** (optimized instructions), and `@for` **requires
`track`** which avoids the old `trackBy` performance pitfall.
```html
@if (user(); as u) { <p>{{ u.name }}</p> } @else { <p>Guest</p> }
@for (item of items(); track item.id) { <li>{{ item.name }}</li> }
@empty { <li>No items</li> }
```

---

## 15b. More real-world questions (tie these to your experience)

### Q52a. How do you attach a JWT to every request? (HTTP interceptor)
A functional interceptor injects the token once, centrally — no repetition per service:
```ts
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(AuthService).token();
  const authReq = token ? req.clone({ setHeaders: { Authorization: `Bearer ${token}` } }) : req;
  return next(authReq);
};
// provideHttpClient(withInterceptors([authInterceptor]))
```
**Real-world:** this is exactly how a frontend talks to a JWT-secured Spring Boot API —
the interceptor adds `Authorization: Bearer …`, and a second interceptor can catch `401`
to trigger a refresh-token call. Great story to connect frontend + backend.

### Q52b. How does a route guard work? (you built an AuthGuard at DXC)
```ts
export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  return auth.isLoggedIn() ? true : inject(Router).createUrlTree(['/login']);
};
// { path: 'admin', canActivate: [authGuard] }
```
**Real-world:** redirect unauthenticated users to login, or use `canMatch` to **not even
download** a lazy admin bundle for non-admins (security + perf win).

### Q52c. How do you prevent memory leaks from subscriptions?
- Prefer the **`async` pipe** (auto-unsubscribes).
- Use **`takeUntilDestroyed()`** (v16+) for manual subscriptions.
- Or signals (no subscription to leak).
```ts
ngOnInit() {
  this.api.poll().pipe(takeUntilDestroyed(this.destroyRef)).subscribe(...);
}
```
**Real-world:** forgotten subscriptions on a dashboard that re-mounts repeatedly are a
classic memory-leak/perf bug — interviewers love this practical answer.

### Q52d. Content projection — what is `<ng-content>` and when do you use it?
It lets a component render arbitrary child markup passed by its parent — the basis of
reusable UI like cards, dialogs, and panels:
```html
<!-- card.component.html -->
<div class="card"><ng-content select="[header]"></ng-content><ng-content></ng-content></div>
```
**Real-world:** your shared component library (PrimeNG/Material-style) uses projection so
each team drops its own content into a consistent shell.

---

## 15c. Even more Q&A (deeper explanations)

### Q53. Reactive forms vs template-driven forms — which and why?
- **Template-driven** — logic lives in the template with `[(ngModel)]`; simple, good for
  tiny forms, but hard to test and scale.
- **Reactive (model-driven)** — the form model is defined in TypeScript with
  `FormGroup`/`FormControl`; **explicit, testable, strongly typed**, easy dynamic
  validation. Preferred for anything non-trivial.
```ts
form = this.fb.group({
  email: ['', [Validators.required, Validators.email]],
  password: ['', [Validators.required, Validators.minLength(8)]],
});
get email() { return this.form.controls.email; }
```
**Real-world:** a multi-step signup or a filter panel — reactive forms let you react to
`valueChanges` (e.g., `debounceTime` then call the API), add/remove controls
dynamically, and unit-test validation without touching the DOM.

### Q54. Hot vs cold observables — what's the difference?
- **Cold** — produces values **per subscriber**; each subscription gets its own run
  (e.g., `HttpClient.get()` fires a **new** HTTP request for every `subscribe`).
- **Hot** — shares one producer across subscribers (e.g., DOM events, a `Subject`).
```ts
const data$ = this.http.get('/api').pipe(shareReplay(1)); // make it hot + cached
```
**Real-world gotcha:** subscribing to the same cold `http.get()` in three places fires
**three** requests. `shareReplay(1)` multicasts one response to all — a very common
performance fix.

### Q55. `Subject` vs `BehaviorSubject` vs `ReplaySubject` vs `AsyncSubject`?
- **`Subject`** — multicast; subscribers get values emitted **after** they subscribe.
- **`BehaviorSubject`** — holds a **current value**; new subscribers immediately get the
  latest. (Great for state.)
- **`ReplaySubject`** — replays the last **N** values to new subscribers.
- **`AsyncSubject`** — emits only the **final** value, on complete.
```ts
private user$ = new BehaviorSubject<User | null>(null); // simple state store
```
**Real-world:** a lightweight auth/state service typically uses `BehaviorSubject` so any
component that subscribes immediately knows the current user.

### Q56. `forkJoin` vs `combineLatest` vs `zip`?
- **`forkJoin`** — waits for all to **complete**, emits the last value of each **once**
  (like `Promise.all`). Good for parallel one-shot HTTP calls.
- **`combineLatest`** — emits whenever **any** source emits, combining the latest of
  each. Good for reacting to multiple live inputs (filters + sort + search).
- **`zip`** — pairs emissions by index.
```ts
forkJoin({ user: this.api.user(), roles: this.api.roles() })
  .subscribe(({ user, roles }) => { ... }); // both loaded together
```

### Q57. Route resolvers vs guards — what's the difference?
- **Guard** (`CanActivate`/`CanMatch`) — decides **whether** you can enter a route.
- **Resolver** — **pre-fetches data** before the route activates, so the component opens
  with data ready (no loading flash).
```ts
export const userResolver: ResolveFn<User> = (route) =>
  inject(Api).getUser(route.paramMap.get('id')!);
```
**Trade-off:** resolvers delay navigation until data arrives — often a route-level
spinner or `@defer` gives a snappier feel. Be ready to discuss that trade-off.

### Q58. `computed()` vs `effect()` — and the rules for each.
- **`computed()`** — a **derived, cached** signal; recomputes only when a dependency
  changes; must be **pure** (no side effects).
- **`effect()`** — runs **side effects** (logging, syncing to localStorage, imperative
  DOM) when its dependencies change. Don't set signals you also read inside it (infinite
  loop risk).
```ts
count = signal(0);
double = computed(() => this.count() * 2);     // derived value
constructor() { effect(() => localStorage.setItem('c', `${this.count()}`)); } // side effect
```

### Q59. How do signals and RxJS interoperate (`toSignal` / `toObservable`)?
- **`toSignal(obs$)`** — consume an observable as a signal in templates (auto-unsub).
- **`toObservable(sig)`** — turn a signal into a stream to use RxJS operators
  (`debounceTime`, `switchMap`).
```ts
search = signal('');
results = toSignal(toObservable(this.search).pipe(
  debounceTime(300), switchMap(q => this.api.search(q))), { initialValue: [] });
```
**Real-world:** this bridges the two worlds — signals for simple view state, RxJS for
complex async pipelines — exactly the modern Angular pattern.

### Q60. `ngOnChanges` vs `ngDoCheck` — when does each fire?
- **`ngOnChanges`** — fires when an **`@Input()` reference** changes; gives you a
  `SimpleChanges` map with previous/current values.
- **`ngDoCheck`** — fires on **every** change-detection run; use for custom/deep change
  detection that `ngOnChanges` misses (e.g., a mutated array). Keep it **cheap** — it
  runs constantly.

### Q61. What is `markForCheck()` and when do you need it with `OnPush`?
With `OnPush`, a component only re-checks on input-reference change, events, or async
pipe. If state changes **outside** those (e.g., a callback mutating a signal-less
field), call `cdr.markForCheck()` to schedule a check.
```ts
constructor(private cdr: ChangeDetectorRef) {}
onData(d: Data) { this.data = d; this.cdr.markForCheck(); }
```
**Real-world:** an `OnPush` component updated from a WebSocket/3rd-party callback won't
refresh until you `markForCheck()` — a classic "why isn't my view updating?" bug.

### Q62. Lazy loading and preloading strategies?
Lazy routes load a feature bundle only when visited (smaller initial bundle).
**Preloading** then quietly fetches the rest in the background.
```ts
provideRouter(routes, withPreloading(PreloadAllModules));
// route: { path: 'admin', loadChildren: () => import('./admin/routes') }
```
**Real-world:** your module-based architecture story — lazy load admin/reporting areas so
first paint is fast, then `PreloadAllModules` (or a custom strategy) warms likely-next
routes for instant navigation.

---

## 16. Rapid-fire (one-liners)

- **`ng-template` vs `ng-container`?** `ng-template` defines a non-rendered template;
  `ng-container` is a logical, non-DOM grouping element.
- **Pure vs impure pipe?** Pure (default) runs only on reference change; impure runs
  every CD cycle (e.g., `async`). Impure can hurt performance.
- **`async` pipe benefits?** Auto subscribe/unsubscribe, works with `OnPush`, less code.
- **`ViewChild` `{ static: true | false }`?** `static:true` resolves before
  `ngAfterViewInit` (no `*ngIf`); `false` (default) after.
- **`HostListener` / `HostBinding`?** Bind events/properties of the host element.
- **`Renderer2` vs direct DOM?** `Renderer2` is platform-safe (SSR/web workers).
- **`enableProdMode()`?** Disables dev-mode checks (the double CD pass).
- **Standalone `APP_INITIALIZER`?** Run async init (config/feature flags) before bootstrap.
- **What is `inject()`?** Functional DI usable in field initializers, guards,
  interceptors, and `runInInjectionContext`.
- **Difference: `Subject.next()` vs signal `.set()`?** Stream emission vs synchronous
  state replacement that triggers fine-grained CD.

---

### Likely deep-dive chain interviewers follow
> "How does CD work?" → "What does `OnPush` change?" → "Why do signals make zoneless
> possible?" → "How do signals differ from `BehaviorSubject`?" → "Show me `switchMap`
> vs `exhaustMap` in a real feature."

Know that chain cold and you'll clear most Angular rounds.
