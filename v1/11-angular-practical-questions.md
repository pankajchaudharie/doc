# Practical Angular Questions — Real Tasks Companies Ask

Not theory — these are the **"build this / fix this / how would you implement…"** tasks
that come up in Angular interviews and live coding rounds. Each has **working code**, the
**reasoning**, and **how to talk about it** using your real experience (Abbott/Merlin.net,
Wipro micro-frontends, the movie-booking app). Tailored for Angular 9 → 19.

> In a live round they'll often share a StackBlitz. Narrate your choices ("I'll use a
> reactive form so validation is testable", "OnPush here for performance") — that's what
> separates a senior from a junior.

---

## Part A — Components & communication

### Q1. Pass data parent → child and child → parent
```ts
// child.component.ts
@Component({ selector: 'app-child', standalone: true, template: `
  <p>{{ title }}</p>
  <button (click)="liked.emit(title)">Like</button>
`})
export class ChildComponent {
  @Input() title = '';
  @Output() liked = new EventEmitter<string>();
}
// parent template
<app-child [title]="movieName" (liked)="onLiked($event)" />
```
**Angular 19 signal style:**
```ts
title = input<string>('');        // signal input
liked = output<string>();         // signal output
```
**Talk track:** `@Input` flows data down, `@Output` + `EventEmitter` sends events up.
Modern Angular replaces them with `input()`/`output()` signal APIs.

### Q2. Share data between unrelated components (a service with state)
```ts
@Injectable({ providedIn: 'root' })
export class CartService {
  private items = signal<Movie[]>([]);
  readonly count = computed(() => this.items().length);
  add(movie: Movie) { this.items.update(list => [...list, movie]); }
}
```
**Talk track:** a singleton service is the standard way to share state. Pre-signals you'd
use a `BehaviorSubject`; now a `signal` is cleaner and template-friendly. **Real-world:**
exactly how a "selected seats" or cart state would work in the movie app.

### Q3. Build a reusable, configurable component (content projection)
```ts
@Component({ selector: 'app-card', standalone: true, template: `
  <div class="card">
    <header><ng-content select="[card-title]"></ng-content></header>
    <div class="body"><ng-content></ng-content></div>
  </div>
`})
export class CardComponent {}
// usage
<app-card><h2 card-title>Avengers</h2><p>Action • 2h 30m</p></app-card>
```
**Talk track:** `ng-content` projects caller markup into a consistent shell — the basis of
a design-system/component library (ties to your reusable-components resume point).

---

## Part B — Forms (a guaranteed practical task)

### Q4. Build a reactive login form with validation
```ts
@Component({ /* ... */, imports: [ReactiveFormsModule] })
export class LoginComponent {
  private fb = inject(FormBuilder);
  form = this.fb.group({
    email: ['', [Validators.required, Validators.email]],
    password: ['', [Validators.required, Validators.minLength(8)]],
  });
  submit() {
    if (this.form.invalid) { this.form.markAllAsTouched(); return; }
    console.log(this.form.value);   // { email, password }
  }
}
```
```html
<form [formGroup]="form" (ngSubmit)="submit()">
  <input formControlName="email" />
  @if (form.controls.email.touched && form.controls.email.invalid) {
    <small>Enter a valid email</small>
  }
  <button [disabled]="form.invalid">Login</button>
</form>
```
**Why reactive over template-driven:** explicit model, easy to unit test, dynamic
validation. **Real-world:** this is your movie-app login talking to the JWT backend.

### Q5. Custom validator (e.g., password match)
```ts
function passwordMatch(group: AbstractControl): ValidationErrors | null {
  const pw = group.get('password')?.value;
  const confirm = group.get('confirm')?.value;
  return pw === confirm ? null : { mismatch: true };
}
this.form = this.fb.group({
  password: [''], confirm: ['']
}, { validators: passwordMatch });
```
**Talk track:** validators are pure functions returning an error object or `null` — easy
to test and reuse.

### Q6. React to form changes (search-as-you-type)
```ts
this.search.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(term => this.api.searchMovies(term))
).subscribe(results => this.results.set(results));
```
**Talk track:** `debounceTime` waits for a pause, `switchMap` cancels the stale request.
**Real-world:** the movie search box — name-drop `switchMap` cancellation to sound senior.

---

## Part C — HTTP & services

### Q7. Call a REST API and show data
```ts
@Injectable({ providedIn: 'root' })
export class MovieService {
  private http = inject(HttpClient);
  getMovies(): Observable<Movie[]> {
    return this.http.get<Movie[]>('/api/v1/movies');
  }
}
// component
movies = toSignal(this.movieService.getMovies(), { initialValue: [] });
```
```html
@for (m of movies(); track m.id) { <li>{{ m.title }}</li> }
@empty { <li>No movies</li> }
```
**Talk track:** services own data access; `toSignal` consumes the observable directly in
the template (auto-unsubscribe).

### Q8. Attach a JWT to every request (HTTP interceptor)
```ts
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(AuthService).token();
  const authReq = token
    ? req.clone({ setHeaders: { Authorization: `Bearer ${token}` } })
    : req;
  return next(authReq);
};
// provideHttpClient(withInterceptors([authInterceptor]))
```
**Real-world:** this is how the Angular app talks to the JWT-secured Spring Boot API — add
a second interceptor to catch `401` and refresh the token.

### Q9. Handle HTTP errors gracefully
```ts
this.http.get<Movie[]>('/api/v1/movies').pipe(
  retry(1),
  catchError(err => {
    this.toast.error('Could not load movies');
    return of([]);              // graceful fallback so the UI doesn't break
  })
).subscribe(list => this.movies.set(list));
```
**Talk track:** `catchError` returns a safe fallback stream; `retry` handles transient
failures. A centralized `ErrorHandler` covers uncaught errors app-wide.

---

## Part D — Routing

### Q10. Set up routes with a lazy-loaded feature
```ts
export const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'movies/:id', component: MovieDetailComponent },
  { path: 'admin', canMatch: [adminGuard],
    loadChildren: () => import('./admin/admin.routes').then(m => m.ADMIN_ROUTES) },
  { path: '**', component: NotFoundComponent },
];
```
**Talk track:** lazy loading keeps the initial bundle small (your performance story);
`canMatch` won't even download the admin bundle for non-admins.

### Q11. Protect a route with a guard
```ts
export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  return auth.isLoggedIn() ? true : inject(Router).createUrlTree(['/login']);
};
```
**Real-world:** your DXC AuthGuard — redirect unauthenticated users to login.

### Q12. Read a route param and load data
```ts
export class MovieDetailComponent {
  private route = inject(ActivatedRoute);
  private api = inject(MovieService);
  movie = toSignal(
    this.route.paramMap.pipe(
      switchMap(p => this.api.getMovie(p.get('id')!))
    )
  );
}
```
**Talk track:** reacting to `paramMap` means navigating from `/movies/1` to `/movies/2`
reloads data without re-creating the component.

---

## Part E — Performance (your specialty — lead with these)

### Q13. Optimize a slow list rendering
```ts
@Component({ changeDetection: ChangeDetectionStrategy.OnPush, /* ... */ })
```
```html
@for (movie of movies(); track movie.id) {     <!-- track avoids re-rendering all rows -->
  <app-movie-card [movie]="movie" />
}
```
**Talk track:** `OnPush` + immutable data + `track` is the combo. Without `track`, Angular
re-creates every DOM node when the list changes. **Real-world:** tie to your "+10%
Lighthouse / cut change-detection work" story.

### Q14. Lazy-render below-the-fold content (`@defer`, v17+)
```html
@defer (on viewport) {
  <app-reviews [movieId]="id" />
} @placeholder { <div>Scroll to load reviews…</div> } @loading { <app-spinner /> }
```
**Talk track:** declaratively defers heavy components until needed — improves First
Contentful Paint with zero manual code-splitting.

### Q15. Prevent memory leaks from subscriptions
```ts
ngOnInit() {
  this.socket.messages()
    .pipe(takeUntilDestroyed(this.destroyRef))
    .subscribe(msg => this.handle(msg));
}
```
**Talk track:** prefer the `async` pipe / signals (no manual subscription); when you must
subscribe, `takeUntilDestroyed` auto-cleans up. **Real-world:** a dashboard that re-mounts
leaks without this — a classic perf bug you can speak to.

---

## Part F — RxJS practical scenarios

### Q16. Load two things in parallel, then render
```ts
forkJoin({
  movie: this.api.getMovie(id),
  reviews: this.api.getReviews(id),
}).subscribe(({ movie, reviews }) => { /* both ready together */ });
```

### Q17. Cancel a previous request (typeahead)
```ts
this.query$.pipe(switchMap(q => this.api.search(q))).subscribe(/* ... */);
```
**Why `switchMap`:** cancels the in-flight request when a new query arrives — no stale
results flicker.

### Q18. Avoid duplicate HTTP calls (share one response)
```ts
readonly config$ = this.http.get<Config>('/api/config').pipe(shareReplay(1));
```
**Talk track:** without `shareReplay`, each subscriber fires a **new** request (cold
observable). `shareReplay(1)` multicasts one response to all — a common real fix.

---

## Part G — "How would you…" design questions

### Q19. How would you structure a large Angular app?
- **Feature-based folders** (`/features/movies`, `/features/auth`), a **core** module for
  singletons (auth, interceptors), a **shared** module/components for reusable UI.
- **Lazy-load** each feature route. **Standalone** components (Angular 14+).
- State: services + signals for simple state; **NgRx** only when state is complex and
  shared widely. **Real-world:** your module-based architecture at Merlin.net.

### Q20. How did you implement micro-frontends? (Wipro)
- **Webpack Module Federation** — a shell app loads remote feature bundles at runtime.
- **Pros:** independent deploys, team autonomy. **Cons:** shared-dependency versioning,
  larger total payload, cross-app styling/state coordination.
- Mention **sharing singletons** (one Angular/RxJS instance) and **avoiding style
  collisions** (`ViewEncapsulation`, prefixes).

### Q21. How do you test a component?
```ts
it('loads movies on init', () => {
  const apiSpy = jasmine.createSpyObj('MovieService', ['getMovies']);
  apiSpy.getMovies.and.returnValue(of([{ id: 1, title: 'X' }]));
  TestBed.configureTestingModule({
    imports: [MovieListComponent],
    providers: [{ provide: MovieService, useValue: apiSpy }],
  });
  const fixture = TestBed.createComponent(MovieListComponent);
  fixture.detectChanges();                    // triggers ngOnInit
  expect(fixture.componentInstance.movies().length).toBe(1);
});
```
**Talk track:** mock dependencies with spies, `detectChanges()` runs lifecycle hooks,
assert on the result. Use `HttpTestingController` for HTTP, `fakeAsync`/`tick` for timers.

### Q22. How would you migrate Angular 9 → 19? (your headline story)
- Upgrade **one major at a time** (`ng update`), fix breaking changes per changelog.
- Migrate components to **standalone**, replace `*ngIf/*ngFor` with **`@if/@for`**,
  introduce **signals**, add **`@defer`** + lazy routes, use **`NgOptimizedImage`**.
- Measure with **Lighthouse** before/after. **Result:** +10% performance, login 7s→3s.

---

## Part H — More practical tasks (round 2)

### Q23. Build a custom pipe (e.g., truncate text)
```ts
@Pipe({ name: 'truncate', standalone: true })
export class TruncatePipe implements PipeTransform {
  transform(value: string, limit = 20, trail = '…'): string {
    return value.length > limit ? value.slice(0, limit) + trail : value;
  }
}
// usage: {{ movie.synopsis | truncate:50 }}
```
**Talk track:** pipes are pure by default (re-run only when the input reference changes) —
great for performance. Mark `pure: false` only if you must react to internal mutations.

### Q24. Build a custom attribute directive (highlight on hover)
```ts
@Directive({ selector: '[appHighlight]', standalone: true })
export class HighlightDirective {
  private el = inject(ElementRef);
  @Input() appHighlight = 'yellow';
  @HostListener('mouseenter') onEnter() { this.set(this.appHighlight); }
  @HostListener('mouseleave') onLeave() { this.set(''); }
  private set(color: string) { this.el.nativeElement.style.background = color; }
}
// usage: <li appHighlight="lightblue">…</li>
```
**Why asked:** tests `ElementRef`, `@HostListener`, and `@Input` on a directive — the
building block of reusable DOM behavior.

### Q25. Load a component dynamically
```ts
@ViewChild('host', { read: ViewContainerRef }) host!: ViewContainerRef;

show() {
  this.host.clear();
  const ref = this.host.createComponent(MovieCardComponent);
  ref.setInput('movie', this.movie);          // pass an @Input
}
```
**Real-world:** modals, tooltips, a dashboard of widgets chosen at runtime.

### Q26. Two-way binding on a custom component
```ts
@Component({ selector: 'app-rating', /* ... */ })
export class RatingComponent {
  value = model(0);                 // Angular 17.2+ two-way signal
}
// usage: <app-rating [(value)]="movieRating" />
```
**Talk track:** `model()` creates a writable signal that supports the `[(...)]` banana-in-a-box
syntax — the modern replacement for paired `@Input` + `@Output`.

### Q27. Show a loading spinner during an HTTP call
```ts
loading = signal(false);
load() {
  this.loading.set(true);
  this.api.getMovies().pipe(finalize(() => this.loading.set(false)))
    .subscribe(list => this.movies.set(list));
}
```
```html
@if (loading()) { <app-spinner /> } @else { @for (m of movies(); track m.id) { … } }
```
**Talk track:** `finalize` runs on both success and error — the correct place to stop a
spinner.

### Q28. Debounce a button to prevent double-submit
```ts
private click$ = new Subject<void>();
ngOnInit() {
  this.click$.pipe(throttleTime(1000), takeUntilDestroyed(this.destroyRef))
    .subscribe(() => this.book());
}
onBook() { this.click$.next(); }
```
**Real-world:** prevents double-booking a seat if the user taps twice — a concrete bug fix.

### Q29. Fix `ExpressionChangedAfterItHasBeenCheckedError`
**Cause:** you changed a bound value *after* change detection ran (often in `ngAfterViewInit`).
**Fixes:**
- Move the change to `ngOnInit`, or
- Defer it: `Promise.resolve().then(() => this.value = x)` / `setTimeout`, or
- Use `ChangeDetectorRef.detectChanges()` deliberately, or signals (which schedule correctly).
**Talk track:** it's a dev-mode safety check that the view is consistent in one pass —
shows you understand Angular's change-detection lifecycle.

### Q30. Cache an HTTP response so repeated calls don't refetch
```ts
private cache$?: Observable<Config>;
getConfig(): Observable<Config> {
  return this.cache$ ??= this.http.get<Config>('/api/config').pipe(shareReplay(1));
}
```
**Real-world:** your Merlin.net caching win — cut redundant API calls and load time.

### Q31. Conditionally apply classes / styles
```html
<span [class.active]="isActive()" [class.sold-out]="seats() === 0"
      [style.color]="rating() > 4 ? 'gold' : 'gray'">{{ title }}</span>
<!-- or [ngClass] for an object map -->
<div [ngClass]="{ premium: isPremium(), disabled: !available() }"></div>
```

### Q32. Format and localize values in the template
```html
{{ price | currency:'USD' }}          <!-- $250.00 -->
{{ showtime | date:'short' }}         <!-- 6/27/26, 7:30 PM -->
{{ rating | number:'1.1-1' }}         <!-- 4.5 -->
{{ booked | percent }}                <!-- 75% -->
```
**Talk track:** built-in pipes handle i18n-aware formatting — no manual string building.

---

### Live-coding tips
1. **State assumptions** and the approach before typing.
2. **Default to reactive forms, OnPush, and signals** — signals modern Angular maturity.
3. **Handle loading/error states**, not just the happy path.
4. **Unsubscribe** (async pipe / `takeUntilDestroyed`) — interviewers watch for leaks.
5. **Tie answers to real experience** — performance, migration, micro-frontends, AuthGuard.
