# Angular Interview Questions & Answers (5 Years Experience)

---

## Table of Contents
1. [Components, Modules & Lifecycle Hooks](#1-components-modules--lifecycle-hooks)
2. [Data Binding & Directives](#2-data-binding--directives)
3. [Services & Dependency Injection](#3-services--dependency-injection)
4. [RxJS & Observables](#4-rxjs--observables)
5. [Routing & Lazy Loading](#5-routing--lazy-loading)
6. [Forms](#6-forms-template-driven--reactive)
7. [State Management](#7-state-management)
8. [Performance Optimization](#8-performance-optimization)
9. [Angular CLI & Project Structure](#9-angular-cli--project-structure)
10. [Security Best Practices](#10-security-best-practices)
11. [Scenario-Based & Real-World Problems](#11-scenario-based--real-world-problems)
12. [Coding Questions](#12-coding-questions)
13. [Behavioral & System Design](#13-behavioral--system-design-questions)
14. [Tips for Candidates](#14-tips-for-answering-effectively)

---

## 1. Components, Modules & Lifecycle Hooks

### Q1. What is the difference between a Component and a Directive? ⭐ *Commonly Asked*

**Answer:**
- A **Component** is a directive with a template. It controls a patch of screen (view).
- A **Directive** modifies the behavior or appearance of an existing DOM element.

| Feature | Component | Directive |
|---------|-----------|-----------|
| Template | Yes (`@Component`) | No (`@Directive`) |
| Selector | Element selector | Attribute/class selector |
| Per element | Only one component | Multiple directives |

**Follow-up:** *Can you have multiple components on the same element?*
No. Angular allows only one component per DOM element, but multiple directives.

---

### Q2. Explain Angular Module architecture. What is the difference between `declarations`, `imports`, `exports`, and `providers`?

**Answer:**

**Angular Module Architecture:**

Angular apps are organized into cohesive blocks of functionality using NgModules. The architecture follows a layered pattern:

```
┌─────────────────────────────────────────────────┐
│                  AppModule (Root)                │  ← Bootstraps the app
├─────────────────────────────────────────────────┤
│              CoreModule (Singleton)              │  ← Services, guards, interceptors (imported ONCE)
├─────────────────────────────────────────────────┤
│               SharedModule (Reusable)            │  ← Common components, pipes, directives
├──────────────┬──────────────┬───────────────────┤
│ FeatureModule│ FeatureModule│  FeatureModule     │  ← Lazy-loaded domain modules
│ (Dashboard)  │ (Users)      │  (Settings)        │
└──────────────┴──────────────┴───────────────────┘
```

| Module Type | Purpose | Import Strategy |
|-------------|---------|-----------------|
| **Root (AppModule)** | Bootstraps app, imports core | Once, at startup |
| **Core** | Singleton services, app-wide interceptors/guards | Once in AppModule |
| **Shared** | Reusable UI components, pipes, directives | In every feature module that needs them |
| **Feature** | Encapsulates a business domain | Lazy-loaded via router |

**NgModule Metadata Properties:**

```typescript
@NgModule({
  declarations: [MyComponent, MyDirective, MyPipe], // Belongs to THIS module
  imports: [CommonModule, SharedModule],             // Other modules needed
  exports: [MyComponent, MyPipe],                   // Available to importing modules
  providers: [MyService],                           // DI providers (module-scoped with lazy loading)
  bootstrap: [AppComponent]                         // Only in root module
})
export class FeatureModule {}
``

- `declarations` — Components, directives, pipes that belong to this module
- `imports` — Other NgModules whose exported classes are needed
- `exports` — Subset of declarations available to other modules
- `providers` — Services available for DI (if lazy-loaded, scoped to that module)
- `bootstrap` — The root component Angular creates and inserts into `index.html` (only in AppModule)

**How modules interact:**
1. **AppModule** imports CoreModule (once) and eagerly-loaded feature modules
2. **CoreModule** provides singleton services to the entire app
3. **SharedModule** is imported by any feature module needing common UI components
4. **Feature modules** are lazy-loaded via routes — they get their own child injector

**Follow-up:** *What happens if you declare a component in two modules?*
Angular throws a runtime error. A component can only belong to ONE module.

**Follow-up:** *How do you prevent CoreModule from being imported more than once?*
```typescript
export class CoreModule {
  constructor(@Optional() @SkipSelf() parentModule: CoreModule) {
    if (parentModule) {
      throw new Error('CoreModule is already loaded. Import it only in AppModule.');
    }
  }
}
```

---

### Q3. List all Angular lifecycle hooks in execution order and explain when you'd use each. ⭐ *Commonly Asked*

**Answer:**

| Hook | Timing | Use Case |
|------|--------|----------|
| `ngOnChanges` | Before `ngOnInit` and on every `@Input` change | React to input property changes |
| `ngOnInit` | Once, after first `ngOnChanges` | Initialize component, fetch data |
| `ngDoCheck` | Every change detection cycle | Custom change detection logic |
| `ngAfterContentInit` | After content projection (`<ng-content>`) | Access projected content |
| `ngAfterContentChecked` | After every check of projected content | React to projected content changes |
| `ngAfterViewInit` | After view and child views initialized | Access `@ViewChild` elements |
| `ngAfterViewChecked` | After every check of view | React to view changes |
| `ngOnDestroy` | Before component destruction | Cleanup subscriptions, timers |

**Follow-up:** *Why shouldn't you use `ngDoCheck` frequently?*
It fires on EVERY change detection cycle, which can cause performance issues. Use it only when Angular's default change detection isn't sufficient.

---

### Q4. What is `ng-content` and how does content projection work? Explain multi-slot projection.

**Answer:**
`ng-content` enables content projection (transclusion) — passing markup into a component from the parent.

**Multi-slot projection:**
```typescript
// card.component.html
<div class="card">
  <div class="header">
    <ng-content select="[card-header]"></ng-content>
  </div>
  <div class="body">
    <ng-content select="[card-body]"></ng-content>
  </div>
  <div class="footer">
    <ng-content></ng-content> <!-- Default slot -->
  </div>
</div>

// Usage
<app-card>
  <div card-header>Title</div>
  <div card-body>Content here</div>
  <p>This goes to default slot</p>
</app-card>
```

**Follow-up:** *What's the difference between `ng-content` and `ng-template`?*
- `ng-content` projects external content into the component
- `ng-template` defines a template that isn't rendered until explicitly used (with `*ngIf`, `ngTemplateOutlet`, etc.)

---

### Q5. What are standalone components in Angular 14+? How do they change the module system?

**Answer:**
Standalone components don't need to be declared in an NgModule. They manage their own dependencies directly.

```typescript
@Component({
  selector: 'app-user-card',
  standalone: true,
  imports: [CommonModule, RouterModule, UserAvatarComponent],
  template: `<div>{{ user.name }}</div>`
})
export class UserCardComponent {
  @Input() user!: User;
}
```

**Benefits:**
- Reduced boilerplate (no NgModule needed)
- Better tree-shaking
- Simplified lazy loading (can lazy-load a component directly)
- Easier to understand dependency graph

**Follow-up:** *Can standalone and non-standalone components coexist?*
Yes. You can import standalone components into NgModules and vice versa.

---

### Q6. Explain `ViewEncapsulation` modes and when would you change the default.

**Answer:**
```typescript
@Component({
  encapsulation: ViewEncapsulation.Emulated // Default
})
```

| Mode | Behavior |
|------|----------|
| `Emulated` (default) | Adds unique attribute selectors to scope CSS |
| `None` | No encapsulation, styles are global |
| `ShadowDom` | Uses native Shadow DOM |

**When to change:**
- `None` — When building theme/utility styles that must penetrate child components
- `ShadowDom` — When building web components with true isolation

**Follow-up:** *How does `::ng-deep` work and why is it deprecated?*
`::ng-deep` pierces view encapsulation to style child components. It's deprecated because it breaks encapsulation; use CSS custom properties or `ViewEncapsulation.None` on wrapper components instead.

---

## 2. Data Binding & Directives

### Q7. Explain all types of data binding in Angular. ⭐ *Commonly Asked*

**Answer:**

| Type | Syntax | Direction |
|------|--------|-----------|
| Interpolation | `{{ expression }}` | Component → View |
| Property binding | `[property]="expression"` | Component → View |
| Event binding | `(event)="handler()"` | View → Component |
| Two-way binding | `[(ngModel)]="property"` | Both directions |

**Two-way binding breakdown:**
```html
<!-- This: -->
<input [(ngModel)]="name">

<!-- Is sugar for: -->
<input [ngModel]="name" (ngModelChange)="name = $event">
```

---

### Q8. What's the difference between structural and attribute directives? Create a custom structural directive.

**Answer:**
- **Structural directives** change DOM layout (add/remove elements): `*ngIf`, `*ngFor`, `*ngSwitch`
- **Attribute directives** change appearance/behavior: `ngClass`, `ngStyle`, custom directives

**Custom structural directive (show if user has role):**
```typescript
@Directive({
  selector: '[appHasRole]'
})
export class HasRoleDirective implements OnInit {
  @Input() appHasRole!: string;

  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef,
    private authService: AuthService
  ) {}

  ngOnInit() {
    if (this.authService.hasRole(this.appHasRole)) {
      this.viewContainer.createEmbeddedView(this.templateRef);
    } else {
      this.viewContainer.clear();
    }
  }
}

// Usage
<div *appHasRole="'admin'">Admin Panel</div>
```

**Follow-up:** *How would you make it reactive (respond to role changes)?*
Subscribe to a role change observable and call `clear()`/`createEmbeddedView()` accordingly. Remember to unsubscribe in `ngOnDestroy`.

---

### Q9. What is `trackBy` in `*ngFor` and why is it important? ⭐ *Commonly Asked*

**Answer:**
`trackBy` tells Angular how to identify items in a list to minimize DOM re-rendering.

```html
<div *ngFor="let item of items; trackBy: trackById">
  {{ item.name }}
</div>
```

```typescript
trackById(index: number, item: Item): number {
  return item.id;
}
```

**Without trackBy:** Angular destroys and recreates ALL DOM elements when the array reference changes.
**With trackBy:** Angular only re-renders items that actually changed.

**Impact:** Dramatic performance improvement for large lists (100+ items).

---

## 3. Services & Dependency Injection

### Q10. Explain the Angular DI hierarchy and `providedIn` options. ⭐ *Commonly Asked*

**Answer:**

```typescript
// Tree-shakable, singleton at root level
@Injectable({ providedIn: 'root' })
export class GlobalService {}

// Scoped to a specific module
@Injectable({ providedIn: FeatureModule })
export class FeatureScopedService {}

// New instance per component
@Component({
  providers: [LocalService] // Each component instance gets its own
})
```

**DI Hierarchy (top to bottom):**
1. **Platform Injector** — Platform-wide services
2. **Root Injector** — `providedIn: 'root'` or `AppModule` providers
3. **Module Injector** — Lazy-loaded module providers
4. **Element Injector** — Component/directive providers

Angular resolves dependencies by walking UP the injector tree.

**Follow-up:** *What happens when a lazy-loaded module provides the same service as root?*
The lazy module gets its own instance (child injector), while eagerly-loaded modules share the root instance. This can cause bugs with multiple service instances.

---

### Q11. What is the difference between `useClass`, `useValue`, `useFactory`, and `useExisting`?

**Answer:**
```typescript
providers: [
  // useClass - provide a different class
  { provide: Logger, useClass: BetterLogger },

  // useValue - provide a static value
  { provide: API_URL, useValue: 'https://api.example.com' },

  // useFactory - dynamic creation with dependencies
  {
    provide: DataService,
    useFactory: (http: HttpClient, config: AppConfig) => {
      return config.useMock ? new MockDataService() : new RealDataService(http);
    },
    deps: [HttpClient, AppConfig]
  },

  // useExisting - alias one token to another
  { provide: OldService, useExisting: NewService }
]
```

**Follow-up:** *What are `InjectionToken`s and when do you use them?*
`InjectionToken` provides a unique token for non-class dependencies (primitives, interfaces, config objects) since TypeScript interfaces don't exist at runtime.

---

### Q12. Explain `@Optional()`, `@Self()`, `@SkipSelf()`, and `@Host()` decorators.

**Answer:**

| Decorator | Behavior |
|-----------|----------|
| `@Optional()` | Returns `null` if dependency not found (instead of error) |
| `@Self()` | Only look in the current element's injector |
| `@SkipSelf()` | Skip current injector, start from parent |
| `@Host()` | Stop searching at the host component |

```typescript
constructor(
  @Optional() private logger: LoggerService,          // null if not found
  @Self() private local: LocalService,                // only from this component
  @SkipSelf() private parent: ParentService,          // from parent injector
  @Host() private hostService: HostService            // up to host boundary
) {}
```

**Real-world use:** `@SkipSelf()` is commonly used in recursive components (tree structures) to get the parent instance of the same service.

---

## 4. RxJS & Observables

### Q13. What is the difference between `Subject`, `BehaviorSubject`, `ReplaySubject`, and `AsyncSubject`? ⭐ *Commonly Asked*

**Answer:**

| Type | Initial Value | Replay | Completes |
|------|--------------|--------|-----------|
| `Subject` | No | No (late subscribers miss past) | Manual |
| `BehaviorSubject` | Yes (required) | Last value to new subscribers | Manual |
| `ReplaySubject` | No | N values to new subscribers | Manual |
| `AsyncSubject` | No | Only last value on complete | Required |

```typescript
// BehaviorSubject - state management pattern
private userSubject = new BehaviorSubject<User | null>(null);
user$ = this.userSubject.asObservable();

// ReplaySubject - cache last N emissions
private actions = new ReplaySubject<Action>(5); // Buffer last 5

// AsyncSubject - only emits final value
private result = new AsyncSubject<Result>();
```

**Follow-up:** *When would you use `ReplaySubject` over `BehaviorSubject`?*
When you don't have a meaningful initial value, or need to replay multiple past values (e.g., caching last N API responses).

---

### Q14. Explain `switchMap`, `mergeMap`, `concatMap`, and `exhaustMap`. When would you use each? ⭐ *Commonly Asked*

**Answer:**

| Operator | Behavior | Use Case |
|----------|----------|----------|
| `switchMap` | Cancels previous inner observable | Search/typeahead (cancel previous search) |
| `mergeMap` | Runs all in parallel | Bulk operations where order doesn't matter |
| `concatMap` | Queues sequentially | Operations that must maintain order |
| `exhaustMap` | Ignores new until current completes | Login/submit buttons (prevent double-click) |

```typescript
// Typeahead search - cancel previous request
searchControl.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(term => this.searchService.search(term))
);

// Form submit - ignore rapid clicks
submitClick$.pipe(
  exhaustMap(() => this.http.post('/api/save', data))
);

// Sequential file uploads
files$.pipe(
  concatMap(file => this.uploadService.upload(file))
);
```

**Follow-up:** *What happens to in-flight HTTP requests with `switchMap`?*
They are unsubscribed (cancelled). Angular's `HttpClient` sends an abort signal to the browser, cancelling the XHR request.

---

### Q15. How do you handle memory leaks with Observables in Angular? ⭐ *Commonly Asked*

**Answer:**

**Approach 1: `async` pipe (preferred)**
```html
<div>{{ data$ | async }}</div>
```
Automatically subscribes and unsubscribes.

**Approach 2: `takeUntilDestroyed()` (Angular 16+)**
```typescript
export class MyComponent {
  private destroyRef = inject(DestroyRef);

  ngOnInit() {
    this.dataService.getData().pipe(
      takeUntilDestroyed(this.destroyRef)
    ).subscribe(data => this.process(data));
  }
}
```

**Approach 3: `takeUntil` with destroy subject**
```typescript
private destroy$ = new Subject<void>();

ngOnInit() {
  this.service.data$.pipe(
    takeUntil(this.destroy$)
  ).subscribe();
}

ngOnDestroy() {
  this.destroy$.next();
  this.destroy$.complete();
}
```

**Approach 4: Manual unsubscribe (for few subscriptions)**
```typescript
private sub: Subscription;

ngOnInit() {
  this.sub = this.service.data$.subscribe();
}

ngOnDestroy() {
  this.sub.unsubscribe();
}
```

**Follow-up:** *Do HTTP observables need unsubscription?*
HTTP observables complete after emitting, so they don't leak. However, unsubscribing cancels in-flight requests, which is useful for component destruction.

---

### Q16. What is the difference between `combineLatest`, `forkJoin`, `merge`, and `zip`?

**Answer:**

| Operator | Emits | Completes |
|----------|-------|-----------|
| `combineLatest` | When ANY source emits (after all emit at least once) | When all complete |
| `forkJoin` | Once, when ALL sources complete | After single emission |
| `merge` | When ANY source emits (no waiting) | When all complete |
| `zip` | When ALL sources have emitted (paired by index) | When any completes |

```typescript
// Load multiple APIs before showing page
forkJoin({
  user: this.http.get('/api/user'),
  settings: this.http.get('/api/settings')
}).subscribe(({ user, settings }) => { ... });

// Reactive derived state
combineLatest([this.filter$, this.data$]).pipe(
  map(([filter, data]) => data.filter(item => item.matches(filter)))
);
```

---

### Q17. Explain Higher-Order Observables and the `share`, `shareReplay` operators.

**Answer:**

**Problem:** Multiple subscribers to an HTTP observable trigger multiple requests.

```typescript
// BAD - each subscriber triggers a new HTTP call
data$ = this.http.get('/api/data');

// GOOD - share the result among subscribers
data$ = this.http.get('/api/data').pipe(
  shareReplay({ bufferSize: 1, refCount: true })
);
```

**`share` vs `shareReplay`:**
- `share` — Multicasts but doesn't replay to late subscribers
- `shareReplay(1)` — Multicasts AND replays last N values to late subscribers

**`refCount: true`** — Unsubscribes from source when all subscribers leave (prevents memory leaks).

**Follow-up:** *What's the risk of `shareReplay` without `refCount: true`?*
The subscription to the source observable is never cleaned up, causing memory leaks.

---

## 5. Routing & Lazy Loading

### Q18. Explain lazy loading in Angular. How does it work under the hood? ⭐ *Commonly Asked*

**Answer:**

```typescript
// app-routing.module.ts
const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule)
  },
  // Standalone component lazy loading (Angular 14+)
  {
    path: 'dashboard',
    loadComponent: () => import('./dashboard/dashboard.component').then(c => c.DashboardComponent)
  }
];
```

**How it works:**
1. Webpack/esbuild creates separate chunks for lazy-loaded modules
2. When user navigates to the route, Angular downloads the chunk
3. The module is compiled and its components become available
4. A child injector is created for the lazy module's providers

**Preloading strategies:**
```typescript
RouterModule.forRoot(routes, {
  preloadingStrategy: PreloadAllModules // Load all lazy modules after initial load
})
```

**Follow-up:** *How would you implement a custom preloading strategy?*
Implement `PreloadingStrategy` interface, check route data for a `preload` flag:
```typescript
export class SelectivePreload implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>) {
    return route.data?.['preload'] ? load() : of(null);
  }
}
```

---

### Q19. What are Route Guards? Explain each type with use cases.

**Answer:**

| Guard | Interface | Purpose |
|-------|-----------|---------|
| `canActivate` | `CanActivateFn` | Protect route entry |
| `canActivateChild` | `CanActivateChildFn` | Protect child routes |
| `canDeactivate` | `CanDeactivateFn` | Prevent leaving (unsaved changes) |
| `canMatch` | `CanMatchFn` | Control route matching |
| `resolve` | `ResolveFn` | Pre-fetch data before activation |

**Functional guard (modern Angular 15+):**
```typescript
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated()) {
    return true;
  }
  return router.createUrlTree(['/login'], {
    queryParams: { returnUrl: state.url }
  });
};

// Usage
{ path: 'admin', canActivate: [authGuard], component: AdminComponent }
```

**Unsaved changes guard:**
```typescript
export const unsavedChangesGuard: CanDeactivateFn<HasUnsavedChanges> = (component) => {
  if (component.hasUnsavedChanges()) {
    return confirm('You have unsaved changes. Leave?');
  }
  return true;
};
```

---

### Q20. Explain `ActivatedRoute` and how to read route parameters reactively.

**Answer:**
```typescript
export class ProductComponent implements OnInit {
  product$: Observable<Product>;

  constructor(
    private route: ActivatedRoute,
    private productService: ProductService
  ) {}

  ngOnInit() {
    // Reactive approach - handles parameter changes without component recreation
    this.product$ = this.route.paramMap.pipe(
      map(params => params.get('id')!),
      switchMap(id => this.productService.getById(id))
    );

    // Query parameters
    this.route.queryParamMap.pipe(
      map(params => params.get('filter'))
    );

    // Snapshot (non-reactive, use only when component is recreated)
    const id = this.route.snapshot.paramMap.get('id');
  }
}
```

**Follow-up:** *When would snapshot fail?*
When the same component handles different route params (e.g., navigating from `/product/1` to `/product/2`). Angular reuses the component, so `snapshot` still shows the old value.

---

## 6. Forms (Template-driven & Reactive)

### Q21. Compare Template-driven and Reactive forms. When would you choose each? ⭐ *Commonly Asked*

**Answer:**

| Feature | Template-driven | Reactive |
|---------|----------------|----------|
| Setup | FormsModule, `ngModel` | ReactiveFormsModule, `FormGroup` |
| Logic location | Template | Component class |
| Data flow | Async (via directives) | Sync (immediate access) |
| Validation | Directive-based | Function-based |
| Testing | Harder (need DOM) | Easier (no DOM needed) |
| Dynamic forms | Difficult | Easy (`FormArray`) |
| Scalability | Simple forms | Complex forms |

**Choose Template-driven for:** Login forms, simple contact forms, prototypes
**Choose Reactive for:** Dynamic forms, complex validation, forms with arrays, when testability matters

---

### Q22. How do you create custom validators (sync and async) in Reactive Forms?

**Answer:**

**Sync validator:**
```typescript
// Custom validator function
function noWhitespace(control: AbstractControl): ValidationErrors | null {
  if (control.value && control.value.trim().length === 0) {
    return { whitespace: true };
  }
  return null;
}

// Cross-field validator
function passwordMatch(group: AbstractControl): ValidationErrors | null {
  const password = group.get('password')?.value;
  const confirm = group.get('confirmPassword')?.value;
  return password === confirm ? null : { passwordMismatch: true };
}
```

**Async validator (e.g., check username availability):**
```typescript
function uniqueUsername(userService: UserService): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    return userService.checkUsername(control.value).pipe(
      map(exists => exists ? { usernameTaken: true } : null),
      catchError(() => of(null))
    );
  };
}

// Usage
this.form = this.fb.group({
  username: ['', [Validators.required], [uniqueUsername(this.userService)]],
  passwords: this.fb.group({
    password: ['', [Validators.required, Validators.minLength(8)]],
    confirmPassword: ['']
  }, { validators: passwordMatch })
});
```

**Follow-up:** *How do you show validation errors in the template?*
```html
<div *ngIf="form.get('username')?.errors?.['usernameTaken'] && form.get('username')?.touched">
  Username is already taken
</div>
```

---

### Q23. How do you build dynamic forms with `FormArray`?

**Answer:**
```typescript
export class DynamicFormComponent {
  form = this.fb.group({
    name: ['', Validators.required],
    addresses: this.fb.array([])
  });

  get addresses(): FormArray {
    return this.form.get('addresses') as FormArray;
  }

  addAddress() {
    this.addresses.push(this.fb.group({
      street: ['', Validators.required],
      city: [''],
      zip: ['', [Validators.required, Validators.pattern(/^\d{5}$/)]]
    }));
  }

  removeAddress(index: number) {
    this.addresses.removeAt(index);
  }
}
```

```html
<div formArrayName="addresses">
  <div *ngFor="let addr of addresses.controls; let i = index" [formGroupName]="i">
    <input formControlName="street" placeholder="Street">
    <input formControlName="city" placeholder="City">
    <input formControlName="zip" placeholder="ZIP">
    <button (click)="removeAddress(i)">Remove</button>
  </div>
</div>
<button (click)="addAddress()">Add Address</button>
```

---

## 7. State Management

### Q24. When would you introduce NgRx? What problems does it solve? ⭐ *Commonly Asked*

**Answer:**

**Introduce NgRx when:**
- Multiple components need the same state
- State needs to be preserved across route navigations
- Complex async flows with side effects
- Need for undo/redo, time-travel debugging
- Team needs predictable, traceable state mutations

**Don't use NgRx when:**
- Simple app with few shared states
- Component-local state suffices
- Over-engineering risk outweighs benefits

**NgRx Core Concepts:**
```
User Action → Component dispatches Action → Reducer produces new State → Selector reads State → Component renders
                                           → Effects handle side effects (API calls)
```

**Follow-up:** *What are alternatives to NgRx?*
- **NGXS** — Less boilerplate, class-based
- **Akita** — Entity-based state management
- **Component Store** — NgRx's lightweight local state solution
- **Signals** (Angular 16+) — Built-in reactive primitives
- **RxJS + Services** — Simple BehaviorSubject-based stores

---

### Q25. Implement a simple feature store using NgRx.

**Answer:**
```typescript
// actions
export const loadProducts = createAction('[Products] Load');
export const loadProductsSuccess = createAction(
  '[Products] Load Success',
  props<{ products: Product[] }>()
);
export const loadProductsFailure = createAction(
  '[Products] Load Failure',
  props<{ error: string }>()
);

// reducer
export interface ProductState {
  products: Product[];
  loading: boolean;
  error: string | null;
}

const initialState: ProductState = { products: [], loading: false, error: null };

export const productReducer = createReducer(
  initialState,
  on(loadProducts, state => ({ ...state, loading: true, error: null })),
  on(loadProductsSuccess, (state, { products }) => ({ ...state, products, loading: false })),
  on(loadProductsFailure, (state, { error }) => ({ ...state, error, loading: false }))
);

// selectors
export const selectProductState = createFeatureSelector<ProductState>('products');
export const selectAllProducts = createSelector(selectProductState, s => s.products);
export const selectLoading = createSelector(selectProductState, s => s.loading);

// effects
@Injectable()
export class ProductEffects {
  loadProducts$ = createEffect(() =>
    this.actions$.pipe(
      ofType(loadProducts),
      exhaustMap(() =>
        this.productService.getAll().pipe(
          map(products => loadProductsSuccess({ products })),
          catchError(error => of(loadProductsFailure({ error: error.message })))
        )
      )
    )
  );

  constructor(private actions$: Actions, private productService: ProductService) {}
}
```

---

### Q26. Explain Angular Signals and how they compare to RxJS Observables (Angular 16+).

**Answer:**

```typescript
// Signals - synchronous, pull-based reactive primitives
export class CounterComponent {
  count = signal(0);
  doubleCount = computed(() => this.count() * 2);

  increment() {
    this.count.update(v => v + 1);
    // or this.count.set(this.count() + 1);
  }
}
```

| Feature | Signals | Observables |
|---------|---------|-------------|
| Reading value | Synchronous (`signal()`) | Asynchronous (subscribe) |
| Change detection | Granular (no Zone.js needed) | Zone-based or `async` pipe |
| Composition | `computed()` | `combineLatest`, `map`, etc. |
| Side effects | `effect()` | `subscribe()` |
| Async operations | Convert with `toSignal()` | Native |

**Interop:**
```typescript
// Observable to Signal
userSignal = toSignal(this.userService.user$, { initialValue: null });

// Signal to Observable
user$ = toObservable(this.userSignal);
```

---

## 8. Performance Optimization

### Q27. What is `ChangeDetectionStrategy.OnPush` and how does it improve performance? ⭐ *Commonly Asked*

**Answer:**

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
```

**Default strategy:** Checks component on EVERY change detection cycle (any event, timer, HTTP response).

**OnPush strategy:** Only checks when:
1. `@Input()` reference changes (not mutations)
2. Event originates from the component or its children
3. Async pipe emits
4. `ChangeDetectorRef.markForCheck()` is called manually
5. Signal value changes (Angular 16+)

**Key requirement:** Treat data as **immutable**:
```typescript
// BAD - won't trigger OnPush
this.items.push(newItem);

// GOOD - new reference triggers OnPush
this.items = [...this.items, newItem];
```

**Follow-up:** *How do you debug when OnPush isn't updating?*
Use `ChangeDetectorRef.markForCheck()` or ensure inputs are new references. Check for mutable operations on arrays/objects.

---

### Q28. List all Angular performance optimization techniques you know.

**Answer:**

**Bundle & Load Optimization:**
- Lazy loading routes and components
- Preloading strategies for critical modules
- Tree-shaking unused code
- Differential loading (ES2015+ for modern browsers)
- Code splitting with dynamic `import()`

**Runtime Performance:**
- `ChangeDetectionStrategy.OnPush` on all presentational components
- `trackBy` in `*ngFor`
- Pure pipes instead of method calls in templates
- `@defer` blocks (Angular 17+) for below-fold content
- Virtual scrolling (`cdk-virtual-scroll-viewport`) for large lists
- `runOutsideAngular()` for non-UI operations
- Debounce/throttle frequent events
- Avoid complex computations in templates

**Memory:**
- Unsubscribe from observables
- Use `takeUntilDestroyed()`
- Avoid circular references
- Detach change detector for hidden components

**Build:**
- AOT compilation (default since Angular 9)
- Production mode (`enableProdMode()`)
- Source map explorer to analyze bundle
- Budgets in `angular.json`

---

### Q29. Explain `@defer` blocks in Angular 17+ and their triggers.

**Answer:**
```html
<!-- Lazy-load heavy components -->
@defer (on viewport) {
  <app-heavy-chart [data]="chartData"></app-heavy-chart>
} @placeholder {
  <div class="skeleton-loader"></div>
} @loading (minimum 500ms) {
  <app-spinner></app-spinner>
} @error {
  <p>Failed to load chart</p>
}
```

**Trigger types:**
| Trigger | Description |
|---------|-------------|
| `on idle` | When browser is idle |
| `on viewport` | When placeholder enters viewport |
| `on interaction` | On click/focus/input on placeholder |
| `on hover` | On mouseenter |
| `on timer(2000ms)` | After specified delay |
| `on immediate` | Immediately after non-deferred content |
| `when condition` | When expression becomes true |

**Benefits:** Reduces initial bundle size without route-level lazy loading.

---

### Q30. How would you optimize a large list rendering (10,000+ items)?

**Answer:**

**Solution: Virtual Scrolling with CDK**
```typescript
import { ScrollingModule } from '@angular/cdk/scrolling';

@Component({
  template: `
    <cdk-virtual-scroll-viewport itemSize="48" class="list-viewport">
      <div *cdkVirtualFor="let item of items; trackBy: trackById" class="list-item">
        {{ item.name }}
      </div>
    </cdk-virtual-scroll-viewport>
  `,
  styles: [`.list-viewport { height: 600px; }`]
})
export class LargeListComponent {
  items: Item[] = []; // 10,000+ items
  trackById = (i: number, item: Item) => item.id;
}
```

**Additional optimizations:**
- `OnPush` change detection
- Pagination with infinite scroll
- Web Workers for heavy filtering/sorting
- `requestAnimationFrame` for smooth scrolling
- Memoized pure pipes for item transformations

---

## 9. Angular CLI & Project Structure

### Q31. How would you structure a large-scale Angular application?

**Answer:**

```
src/app/
├── core/                    # Singleton services, guards, interceptors
│   ├── auth/
│   ├── interceptors/
│   ├── guards/
│   └── core.module.ts       # Imported ONCE in AppModule
├── shared/                  # Reusable components, pipes, directives
│   ├── components/
│   ├── directives/
│   ├── pipes/
│   └── shared.module.ts     # Imported in feature modules
├── features/                # Feature modules (lazy-loaded)
│   ├── dashboard/
│   │   ├── components/
│   │   ├── services/
│   │   ├── models/
│   │   ├── store/           # Feature state (if using NgRx)
│   │   ├── dashboard-routing.module.ts
│   │   └── dashboard.module.ts
│   ├── users/
│   └── settings/
├── layout/                  # Shell components (header, sidebar, footer)
└── app.module.ts
```

**Principles:**
- Core module imported once (use `throwIfAlreadyLoaded` guard)
- Shared module re-exported by feature modules
- Each feature is independently lazy-loadable
- Smart (container) vs Dumb (presentational) component pattern
- Index barrels (`index.ts`) for clean imports

---

### Q32. Explain Angular workspace concepts: projects, libraries, and monorepo structure.

**Answer:**

```bash
# Generate a library
ng generate library ui-components

# Generate an application
ng generate application admin-portal
```

**`angular.json` structure:**
```json
{
  "projects": {
    "main-app": { "projectType": "application" },
    "admin-app": { "projectType": "application" },
    "ui-components": { "projectType": "library" },
    "shared-utils": { "projectType": "library" }
  }
}
```

**Library benefits:**
- Shared code across multiple apps
- Independent versioning and publishing
- Enforced API boundaries via `public-api.ts`
- Tree-shaking at the entry point level

**Monorepo tools:** Nx extends Angular CLI with dependency graph, affected commands, computation caching.

---

## 10. Security Best Practices

### Q33. How does Angular protect against XSS attacks? ⭐ *Commonly Asked*

**Answer:**

**Built-in protections:**
1. **Auto-sanitization** — Angular sanitizes values in templates by default
2. **Trusted Types** — `[innerHTML]` is sanitized, dangerous tags/attributes removed
3. **Context-aware escaping** — Different sanitization for HTML, URL, style, resource URL

```typescript
// Angular auto-sanitizes this:
<div [innerHTML]="userContent"></div>  // Scripts and dangerous attrs removed

// Bypass sanitization (USE WITH EXTREME CAUTION):
constructor(private sanitizer: DomSanitizer) {}

trustedHtml = this.sanitizer.bypassSecurityTrustHtml(htmlContent);
trustedUrl = this.sanitizer.bypassSecurityTrustResourceUrl(url);
```

**Best practices:**
- Never use `bypassSecurityTrust*` with user input
- Avoid `document.createElement` / direct DOM manipulation
- Use Angular's template binding over `innerHTML` when possible
- Enable Content Security Policy (CSP) headers

**Follow-up:** *What is the difference between sanitization and validation?*
- **Sanitization** removes dangerous content but keeps the rest
- **Validation** rejects the entire input if it doesn't match rules

---

### Q34. How do you secure Angular HTTP communications?

**Answer:**

```typescript
// 1. CSRF/XSRF protection (built-in)
// Angular reads XSRF-TOKEN cookie and sends X-XSRF-TOKEN header automatically
imports: [
  HttpClientXsrfModule.withOptions({
    cookieName: 'XSRF-TOKEN',
    headerName: 'X-XSRF-TOKEN'
  })
]

// 2. Auth interceptor with token refresh
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getToken();

  if (token) {
    req = req.clone({
      setHeaders: { Authorization: `Bearer ${token}` }
    });
  }

  return next(req).pipe(
    catchError(error => {
      if (error.status === 401) {
        return authService.refreshToken().pipe(
          switchMap(newToken => {
            req = req.clone({
              setHeaders: { Authorization: `Bearer ${newToken}` }
            });
            return next(req);
          })
        );
      }
      throw error;
    })
  );
};

// 3. Always use HTTPS
// 4. Validate API responses
// 5. Don't store sensitive data in localStorage (use httpOnly cookies)
```

---

### Q35. What are Angular's security contexts and how do you handle trusted URLs?

**Answer:**

Angular has 4 security contexts:
1. **HTML** — `[innerHTML]`
2. **Style** — `[style]`
3. **URL** — `[href]`, `[src]`
4. **Resource URL** — `<iframe [src]>`, `<object [data]>`

```typescript
// Safe approach for dynamic URLs
@Pipe({ name: 'safeResource' })
export class SafeResourcePipe implements PipeTransform {
  constructor(private sanitizer: DomSanitizer) {}

  transform(url: string): SafeResourceUrl {
    // IMPORTANT: Only use with URLs you control/validate
    if (this.isAllowedDomain(url)) {
      return this.sanitizer.bypassSecurityTrustResourceUrl(url);
    }
    throw new Error('Untrusted URL');
  }

  private isAllowedDomain(url: string): boolean {
    const allowed = ['https://trusted-cdn.com', 'https://api.myapp.com'];
    return allowed.some(domain => url.startsWith(domain));
  }
}
```

---

## 11. Scenario-Based & Real-World Problems

### Q36. You notice your Angular app has become slow. How do you diagnose and fix it? ⭐ *Commonly Asked*

**Answer:**

**Diagnosis Steps:**
1. **Angular DevTools** — Check change detection cycles, component tree
2. **Chrome Performance tab** — Look for long tasks, layout thrashing
3. **Lighthouse audit** — Core Web Vitals (LCP, FID, CLS)
4. **Bundle analysis** — `ng build --stats-json` + webpack-bundle-analyzer
5. **Network tab** — Check request waterfall, payload sizes

**Common causes & fixes:**

| Problem | Diagnosis | Fix |
|---------|-----------|-----|
| Too many CD cycles | DevTools profiler | OnPush, detach CD |
| Large bundle | Bundle analyzer | Lazy loading, tree-shaking |
| Memory leaks | Chrome Memory tab | Unsubscribe, WeakRef |
| Excessive re-renders | Performance profiler | trackBy, pure pipes |
| Heavy computations in template | Highlight updates | Move to computed/pipe |
| Large list rendering | Scroll jank | Virtual scrolling |
| Blocking main thread | Long tasks | Web Workers |

---

### Q37. How would you implement a real-time notification system in Angular?

**Answer:**

```typescript
@Injectable({ providedIn: 'root' })
export class NotificationService {
  private notifications$ = new BehaviorSubject<Notification[]>([]);

  connect(): Observable<Notification> {
    // WebSocket connection
    const ws = webSocket<Notification>('wss://api.example.com/notifications');

    return ws.pipe(
      tap(notification => {
        const current = this.notifications$.value;
        this.notifications$.next([notification, ...current]);
      }),
      retryWhen(errors => errors.pipe(
        scan((retryCount) => retryCount + 1, 0),
        delayWhen(retryCount => timer(Math.min(1000 * 2 ** retryCount, 30000))), // Exponential backoff
        take(10) // Max 10 retries
      )),
      share()
    );
  }

  markAsRead(id: string): Observable<void> {
    return this.http.patch<void>(`/api/notifications/${id}/read`, {}).pipe(
      tap(() => {
        const updated = this.notifications$.value.map(n =>
          n.id === id ? { ...n, read: true } : n
        );
        this.notifications$.next(updated);
      })
    );
  }
}
```

**Key considerations:**
- Exponential backoff for reconnection
- Optimistic UI updates
- Unread count badge with `computed` or `distinctUntilChanged`
- Service Worker for background notifications
- Token refresh for WebSocket auth

---

### Q38. How do you handle error handling globally in an Angular application?

**Answer:**

```typescript
// 1. Global Error Handler
@Injectable()
export class GlobalErrorHandler implements ErrorHandler {
  constructor(private injector: Injector) {}

  handleError(error: any): void {
    const logger = this.injector.get(LoggingService);
    const notifier = this.injector.get(NotificationService);

    if (error instanceof HttpErrorResponse) {
      // Server error
      notifier.showError(`Server Error: ${error.status}`);
      logger.logServerError(error);
    } else {
      // Client error
      notifier.showError('An unexpected error occurred');
      logger.logClientError(error);
    }

    console.error(error); // Always log to console in dev
  }
}

// 2. HTTP Error Interceptor
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    retry({ count: 2, delay: 1000 }), // Retry transient errors
    catchError((error: HttpErrorResponse) => {
      if (error.status === 0) {
        // Network error
        return throwError(() => new Error('Network unavailable'));
      }
      if (error.status === 403) {
        inject(Router).navigate(['/unauthorized']);
      }
      return throwError(() => error);
    })
  );
};

// 3. Provide in module
providers: [
  { provide: ErrorHandler, useClass: GlobalErrorHandler }
]
```

---

### Q39. You're building a multi-tenant SaaS app. How do you handle dynamic theming?

**Answer:**

```typescript
// 1. CSS Custom Properties approach
@Injectable({ providedIn: 'root' })
export class ThemeService {
  applyTheme(tenant: TenantConfig): void {
    const root = document.documentElement;
    root.style.setProperty('--primary-color', tenant.primaryColor);
    root.style.setProperty('--accent-color', tenant.accentColor);
    root.style.setProperty('--font-family', tenant.fontFamily);
    root.style.setProperty('--logo-url', `url(${tenant.logoUrl})`);
  }
}

// 2. styles.scss
:root {
  --primary-color: #1976d2;
  --accent-color: #ff4081;
}

.btn-primary {
  background-color: var(--primary-color);
}

// 3. Load theme on app init
export function initializeTheme(themeService: ThemeService, tenantService: TenantService) {
  return () => tenantService.getTenant().pipe(
    tap(tenant => themeService.applyTheme(tenant))
  ).toPromise();
}

providers: [
  { provide: APP_INITIALIZER, useFactory: initializeTheme, deps: [ThemeService, TenantService], multi: true }
]
```

---

### Q40. How would you implement micro-frontends with Angular?

**Answer:**

**Approach 1: Module Federation (Webpack 5)**
```typescript
// Remote app (micro-frontend) - webpack.config.js
new ModuleFederationPlugin({
  name: 'mfeApp',
  filename: 'remoteEntry.js',
  exposes: {
    './Module': './src/app/feature/feature.module.ts'
  },
  shared: { '@angular/core': { singleton: true }, ... }
});

// Shell app - routes
{
  path: 'feature',
  loadChildren: () => loadRemoteModule({
    remoteEntry: 'http://localhost:4201/remoteEntry.js',
    remoteName: 'mfeApp',
    exposedModule: './Module'
  }).then(m => m.FeatureModule)
}
```

**Approach 2: Web Components (Angular Elements)**
```typescript
@NgModule({
  declarations: [WidgetComponent],
  entryComponents: [WidgetComponent]
})
export class WidgetModule {
  constructor(private injector: Injector) {
    const element = createCustomElement(WidgetComponent, { injector });
    customElements.define('my-widget', element);
  }
}
```

**Considerations:**
- Shared dependencies (singleton Angular)
- Cross-MFE communication (Custom Events, shared state)
- Independent deployment pipelines
- Version compatibility

---

## 12. Coding Questions

### Q41. Implement a debounced search with loading state and error handling.

**Answer:**
```typescript
@Component({
  selector: 'app-search',
  template: `
    <input [formControl]="searchControl" placeholder="Search...">
    <div *ngIf="loading" class="spinner">Loading...</div>
    <div *ngIf="error" class="error">{{ error }}</div>
    <ul>
      <li *ngFor="let result of results$ | async">{{ result.name }}</li>
    </ul>
  `
})
export class SearchComponent implements OnInit {
  searchControl = new FormControl('');
  results$!: Observable<SearchResult[]>;
  loading = false;
  error: string | null = null;

  constructor(private searchService: SearchService) {}

  ngOnInit() {
    this.results$ = this.searchControl.valueChanges.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      filter(term => term!.length >= 2),
      tap(() => { this.loading = true; this.error = null; }),
      switchMap(term =>
        this.searchService.search(term!).pipe(
          catchError(err => {
            this.error = 'Search failed. Please try again.';
            return of([]);
          }),
          finalize(() => this.loading = false)
        )
      )
    );
  }
}
```

---

### Q42. Create a reusable modal/dialog service without third-party libraries.

**Answer:**
```typescript
// modal.service.ts
@Injectable({ providedIn: 'root' })
export class ModalService {
  private modalRef: ComponentRef<ModalContainerComponent> | null = null;

  constructor(
    private appRef: ApplicationRef,
    private injector: Injector
  ) {}

  open<T>(component: Type<T>, config?: ModalConfig): ModalRef<T> {
    // Create modal container
    const containerRef = createComponent(ModalContainerComponent, {
      environmentInjector: this.appRef.injector
    });

    // Create content component
    const contentRef = createComponent(component, {
      environmentInjector: this.appRef.injector,
      hostElement: containerRef.instance.contentHost.nativeElement
    });

    // Attach to DOM
    this.appRef.attachView(containerRef.hostView);
    document.body.appendChild(containerRef.location.nativeElement);

    // Return handle
    const modalRef = new ModalRef(containerRef, contentRef, this.appRef);
    containerRef.instance.close.subscribe(() => modalRef.dismiss());

    return modalRef;
  }
}

// Usage
const ref = this.modalService.open(ConfirmDialogComponent);
ref.afterClosed$.subscribe(result => {
  if (result === 'confirm') { /* proceed */ }
});
```

---

### Q43. Implement an HTTP caching interceptor with TTL.

**Answer:**
```typescript
interface CacheEntry {
  response: HttpResponse<any>;
  expiry: number;
}

@Injectable({ providedIn: 'root' })
export class HttpCacheService {
  private cache = new Map<string, CacheEntry>();

  get(url: string): HttpResponse<any> | null {
    const entry = this.cache.get(url);
    if (!entry) return null;
    if (Date.now() > entry.expiry) {
      this.cache.delete(url);
      return null;
    }
    return entry.response;
  }

  set(url: string, response: HttpResponse<any>, ttlMs: number): void {
    this.cache.set(url, { response, expiry: Date.now() + ttlMs });
  }

  invalidate(pattern?: string): void {
    if (pattern) {
      for (const key of this.cache.keys()) {
        if (key.includes(pattern)) this.cache.delete(key);
      }
    } else {
      this.cache.clear();
    }
  }
}

// Interceptor
export const cachingInterceptor: HttpInterceptorFn = (req, next) => {
  const cache = inject(HttpCacheService);

  // Only cache GET requests
  if (req.method !== 'GET') {
    // Invalidate cache on mutations
    cache.invalidate(req.url.split('?')[0]);
    return next(req);
  }

  // Check cache
  const cached = cache.get(req.urlWithParams);
  if (cached) {
    return of(cached.clone());
  }

  return next(req).pipe(
    filter(event => event instanceof HttpResponse),
    tap(response => {
      if (response instanceof HttpResponse && response.status === 200) {
        const ttl = parseCacheControl(response.headers) || 60000; // Default 1 min
        cache.set(req.urlWithParams, response, ttl);
      }
    })
  );
};
```

---

### Q44. Write a custom pipe that highlights search terms in text.

**Answer:**
```typescript
@Pipe({ name: 'highlight' })
export class HighlightPipe implements PipeTransform {
  transform(text: string, search: string): string {
    if (!search || !text) return text;

    // Escape special regex characters to prevent ReDoS
    const escaped = search.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
    const regex = new RegExp(`(${escaped})`, 'gi');
    return text.replace(regex, '<mark class="highlight">$1</mark>');
  }
}

// Usage (with innerHTML binding for sanitized output)
<span [innerHTML]="item.name | highlight: searchTerm"></span>
```

**Follow-up:** *Is this pipe pure or impure? Why does it matter?*
It's pure (default). Pure pipes only re-evaluate when input reference changes, which is optimal. An impure pipe runs on every change detection cycle.

---

### Q45. Implement a generic typed HTTP service with error handling.

**Answer:**
```typescript
@Injectable({ providedIn: 'root' })
export class ApiService {
  private baseUrl = inject(APP_CONFIG).apiUrl;

  constructor(private http: HttpClient) {}

  get<T>(endpoint: string, params?: HttpParams): Observable<T> {
    return this.http.get<T>(`${this.baseUrl}/${endpoint}`, { params }).pipe(
      this.handleError<T>('GET', endpoint)
    );
  }

  post<T>(endpoint: string, body: unknown): Observable<T> {
    return this.http.post<T>(`${this.baseUrl}/${endpoint}`, body).pipe(
      this.handleError<T>('POST', endpoint)
    );
  }

  put<T>(endpoint: string, body: unknown): Observable<T> {
    return this.http.put<T>(`${this.baseUrl}/${endpoint}`, body).pipe(
      this.handleError<T>('PUT', endpoint)
    );
  }

  delete<T>(endpoint: string): Observable<T> {
    return this.http.delete<T>(`${this.baseUrl}/${endpoint}`).pipe(
      this.handleError<T>('DELETE', endpoint)
    );
  }

  private handleError<T>(method: string, endpoint: string) {
    return (source: Observable<T>) => source.pipe(
      retry({ count: 2, delay: (error, retryCount) => {
        // Only retry on 5xx or network errors
        if (error.status >= 500 || error.status === 0) {
          return timer(1000 * retryCount);
        }
        return throwError(() => error);
      }}),
      catchError((error: HttpErrorResponse) => {
        console.error(`[${method}] ${endpoint} failed:`, error);
        return throwError(() => ({
          status: error.status,
          message: error.error?.message || 'An error occurred',
          endpoint
        }));
      })
    );
  }
}
```

---

## 13. Behavioral & System Design Questions

### Q46. Tell me about a time you improved the performance of an Angular application significantly.

**Expected Answer Structure (STAR method):**
- **Situation:** "Our dashboard loaded in 8+ seconds with 50+ components"
- **Task:** "Reduce initial load time below 3 seconds"
- **Action:** "Profiled with Lighthouse, implemented lazy loading for 6 feature modules, added OnPush to 40+ components, replaced method calls with pure pipes, added virtual scrolling for data tables"
- **Result:** "Load time dropped to 2.1 seconds, bundle size reduced by 60%"

---

### Q47. How would you design a component library that's shared across 5 teams?

**Answer:**

**Architecture:**
- Separate Angular library project (`ng generate library`)
- Published to private npm registry
- Semantic versioning with changelog
- Storybook for documentation and visual testing
- Chromatic for visual regression testing

**Design principles:**
- Headless/unstyled base components + theme layer
- CSS custom properties for theming
- Strict API contracts with `@Input()` / `@Output()`
- Accessibility (WCAG 2.1 AA) built-in
- Tree-shakable (each component independently importable)

**Governance:**
- Design system team owns the library
- RFC process for new components
- Breaking changes only in major versions
- Automated migration schematics for breaking changes

---

### Q48. How would you architect a real-time collaborative editing feature (like Google Docs)?

**Answer:**

**High-level architecture:**
```
┌─────────────┐     WebSocket      ┌──────────────┐     ┌─────────────┐
│  Angular UI  │ ◄──────────────► │  Sync Server  │ ◄──► │  Database   │
│  (CRDT lib)  │                   │  (OT/CRDT)   │     │  (Postgres) │
└─────────────┘                   └──────────────┘     └─────────────┘
```

**Frontend approach:**
- **CRDT** (Conflict-free Replicated Data Types) via Yjs or Automerge
- WebSocket connection with reconnection/offline queue
- Local-first editing (no latency for user)
- Cursor presence indicators (other users' positions)
- Operational Transform or CRDT for conflict resolution

**Angular implementation concerns:**
- `NgZone.runOutsideAngular()` for high-frequency sync events
- Detached change detection on the editor component
- Web Worker for CRDT merge operations
- IndexedDB for offline persistence
- RxJS for managing WebSocket lifecycle

---

### Q49. You join a team with a poorly structured Angular app (everything in AppModule, no lazy loading, 15MB bundle). What's your plan?

**Answer:**

**Phase 1: Assessment (Week 1)**
- Bundle analysis (source-map-explorer)
- Identify largest contributors
- Map component dependency graph
- Set measurable targets (e.g., < 500KB initial bundle)

**Phase 2: Quick Wins (Week 2-3)**
- Enable production optimizations
- Remove dead code and unused imports
- Add build budgets in `angular.json`
- Split into Core/Shared/Feature module structure

**Phase 3: Modularization (Week 4-8)**
- Create feature modules one at a time
- Implement lazy loading for non-critical routes
- Move shared components to SharedModule
- Add `OnPush` to leaf components

**Phase 4: Ongoing**
- CI checks for bundle size regression
- Architecture decision records (ADRs)
- Linting rules to enforce module boundaries (Nx)
- Gradual migration to standalone components

**Key principle:** Incremental migration, never big-bang rewrite. Each PR should be independently deployable.

---

### Q50. How do you handle internationalization (i18n) at scale?

**Answer:**

**Angular built-in i18n:**
```html
<h1 i18n="@@pageTitle">Welcome</h1>
<p i18n="@@greeting">Hello, {{ name }}</p>
```
- Generates separate bundles per locale
- AOT-compiled translations (best performance)
- Requires rebuild per language

**Runtime i18n (ngx-translate or Transloco):**
```typescript
// More flexible, runtime switching
<h1>{{ 'PAGE_TITLE' | translate }}</h1>

// Transloco approach
<h1 transloco="pageTitle"></h1>
```

**At scale considerations:**
- Translation management system (Crowdin, Lokalise)
- CI pipeline extracts new keys automatically
- Lazy-load translations per feature module
- ICU message format for plurals/gender
- RTL support with CSS logical properties
- Date/number formatting with Angular's `LOCALE_ID`

---

## 14. Tips for Answering Effectively

### General Tips:
1. **Structure your answers:** Start with the "what", then "why", then "how"
2. **Give real examples:** Reference actual projects where you applied the concept
3. **Acknowledge tradeoffs:** Every decision has pros and cons — show you understand both
4. **Don't bluff:** If you don't know, say "I haven't used that directly, but my understanding is..."
5. **Ask clarifying questions:** Show you think about context before answering

### For Coding Questions:
1. Think out loud — explain your approach before coding
2. Start with the simplest solution, then optimize
3. Handle edge cases and error scenarios
4. Mention testing — "I would unit test this by..."

### For System Design:
1. Clarify requirements and constraints first
2. Start high-level, then zoom into specifics
3. Discuss scalability, maintainability, and performance
4. Reference specific Angular patterns (services, interceptors, guards)

### Red Flags Interviewers Watch For:
- Not knowing the difference between `OnPush` and Default change detection
- Not handling observable unsubscription
- Mixing template-driven and reactive forms concepts
- Not understanding lazy loading implications on DI
- Inability to debug performance issues systematically

### Green Flags:
- Mentioning Angular DevTools and profiling
- Understanding when NOT to use complex patterns (NgRx for simple apps)
- Considering accessibility in component design
- Discussing testing strategy alongside implementation
- Awareness of Angular's evolution (standalone, signals, defer)

---

## Bonus: Quick-Fire Questions (Expect These!)

| Question | Key Answer |
|----------|-----------|
| AOT vs JIT? | AOT compiles at build time (faster, smaller). JIT at runtime (dev convenience) |
| What is Zone.js? | Library that patches async APIs to trigger change detection |
| `ng-template` vs `ng-container`? | Template = reusable, not rendered. Container = grouping without extra DOM |
| What is tree-shaking? | Dead code elimination by bundler based on ES module imports |
| Difference between `Promise` and `Observable`? | Observable: lazy, cancellable, multiple values, operators |
| What are Angular Elements? | Angular components packaged as custom elements (Web Components) |
| What is `APP_INITIALIZER`? | Token to run async init logic before app bootstraps |
| How to share data between sibling components? | Shared service with Subject, parent intermediary, or state management |
| What are interceptors used for? | Auth headers, logging, caching, error handling, loading spinners |
| Explain `renderer2` vs direct DOM access | Renderer2 is platform-agnostic, works with SSR. Direct DOM doesn't |

---

*Document generated for interview preparation. Covers Angular 14-17+ features.*
