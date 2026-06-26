# HTML & CSS Interview Questions — Deep Dive

Often underestimated, but senior frontend roles probe layout, accessibility, and CSS
architecture hard. Be crisp and show you understand *why*, not just *how*.

---

## Part A — HTML

### Q1. What is semantic HTML and why does it matter?
Using elements for their **meaning**, not just appearance: `<header>`, `<nav>`,
`<main>`, `<article>`, `<section>`, `<aside>`, `<footer>`, `<figure>`, `<time>`.
**Benefits:** accessibility (screen readers), SEO, maintainability, default behavior.
```html
<!-- ❌ div soup -->
<div class="header"><div class="nav">…</div></div>
<!-- ✅ semantic -->
<header><nav aria-label="Primary">…</nav></header>
```

---

### Q2. `<div>` vs `<section>` vs `<article>`?
- **`<div>`** — generic, no meaning (use for styling hooks only).
- **`<section>`** — a thematic grouping, usually with a heading.
- **`<article>`** — self-contained, independently distributable content (blog post,
  card, comment).

---

### Q3. Block vs inline vs inline-block elements?
- **Block** (`div`, `p`, `section`) — full width, new line, respects width/height.
- **Inline** (`span`, `a`, `strong`) — flows in text, ignores width/height & vertical
  margins.
- **Inline-block** — flows inline but respects box dimensions.

---

### Q4. What is the DOCTYPE? Quirks mode?
`<!DOCTYPE html>` tells the browser to use **standards mode**. Without it, browsers fall
back to **quirks mode** (legacy box model & rendering bugs).

---

### Q5. `localStorage` vs `sessionStorage` vs cookies?
| | Capacity | Lifetime | Sent to server | Use |
|--|---------|----------|----------------|-----|
| Cookies | ~4KB | configurable | **yes (every request)** | auth tokens (httpOnly) |
| localStorage | ~5–10MB | until cleared | no | app prefs, cache |
| sessionStorage | ~5MB | per tab/session | no | per-tab temp state |
**Security note:** storing JWTs in `localStorage` is vulnerable to XSS; httpOnly
cookies are safer (but need CSRF protection).

---

### Q6. Accessibility (a11y) — what do you do?
- **Semantic HTML first** (buttons are `<button>`, not clickable `<div>`).
- **`alt` text** on images; empty `alt=""` for decorative.
- **ARIA** only when semantics aren't enough: `aria-label`, `aria-expanded`,
  `role`, `aria-live` for dynamic updates.
- **Keyboard navigation:** focus order, `:focus-visible`, no keyboard traps,
  `tabindex`.
- **Labels:** `<label for>` tied to inputs.
- **Color contrast** ≥ 4.5:1 (WCAG AA).
```html
<button aria-expanded="false" aria-controls="menu">Menu</button>
<img src="chart.png" alt="Sales rose 20% in Q3" />
```
**Why interviewers ask:** legal compliance + inclusive UX; signals senior maturity.

---

### Q7. `defer` vs `async` on `<script>`?
- **`async`** — download in parallel, execute **as soon as ready** (order not
  guaranteed). For independent scripts (analytics).
- **`defer`** — download in parallel, execute **in order, after HTML parsed**. For app
  scripts that touch the DOM.
- **Neither** — blocks parsing while downloading + executing.

---

### Q8. Other HTML topics
- **`data-*` attributes** — custom data on elements, read via `dataset`.
- **Forms:** `required`, `pattern`, `type=email`, `novalidate`, `<fieldset>`.
- **`<picture>` / `srcset`** — responsive images.
- **`loading="lazy"`** — native image lazy loading.
- **Meta viewport** — `<meta name="viewport" content="width=device-width, initial-scale=1">` for responsive.

---

## Part B — CSS

### Q9. Explain the box model.
Every element is a box: **content → padding → border → margin**.
```css
.box { box-sizing: border-box; } /* width includes padding + border */
```
- **`content-box`** (default) — width = content only; padding/border add to it.
- **`border-box`** — width includes padding & border (much easier to reason about;
  most resets set this globally).

---

### Q10. CSS specificity — how is it calculated?
Specificity is `(inline, IDs, classes/attrs/pseudo-classes, elements)`.
```
inline style     → 1,0,0,0
#id              → 0,1,0,0
.class / [attr] / :hover → 0,0,1,0
element / ::before → 0,0,0,1
```
Higher wins; ties → **last rule** wins. `!important` overrides (avoid it).
```css
#nav .link {}     /* 0,1,1,0 */
.menu .link {}    /* 0,0,2,0  → loses to the #id rule */
```

---

### Q11. `position` values?
- **`static`** — default, normal flow.
- **`relative`** — offset from its normal position; creates a positioning context.
- **`absolute`** — removed from flow, positioned to nearest **positioned ancestor**.
- **`fixed`** — positioned to the **viewport** (stays on scroll).
- **`sticky`** — `relative` until a threshold, then `fixed`.

---

### Q12. Flexbox — explain the main properties.
1-D layout (a row OR column).
```css
.container {
  display: flex;
  flex-direction: row;            /* row | column */
  justify-content: space-between; /* main axis alignment */
  align-items: center;            /* cross axis alignment */
  gap: 16px;
  flex-wrap: wrap;
}
.item { flex: 1 1 200px; }        /* grow shrink basis */
```
- **`justify-content`** = main axis; **`align-items`** = cross axis.
- **`flex: 1`** = `flex-grow:1; flex-shrink:1; flex-basis:0`.

---

### Q13. CSS Grid — when over Flexbox?
2-D layout (rows **and** columns at once). Use Grid for **page/section layouts**, Flex
for **1-D component alignment** (toolbars, button rows).
```css
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  grid-template-areas: "header header" "sidebar main";
  gap: 1rem;
}
.main { grid-area: main; }
```
**`auto-fit` + `minmax`** = responsive cards without media queries.

---

### Q14. How do you center a div? (the meme question)
```css
/* Flexbox */
.parent { display: flex; justify-content: center; align-items: center; }
/* Grid */
.parent { display: grid; place-items: center; }
/* Absolute */
.child { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); }
```

---

### Q15. Responsive design — how?
- **Mobile-first** media queries (`min-width`).
- **Relative units:** `rem`/`em`, `%`, `vw`/`vh`, `ch`.
- **Fluid layouts:** Flex/Grid, `clamp()`, `minmax()`.
- **Responsive images:** `srcset`, `<picture>`.
```css
.container { width: 100%; padding: clamp(1rem, 5vw, 3rem); }
@media (min-width: 768px) { .container { max-width: 720px; margin-inline: auto; } }
```

---

### Q16. `rem` vs `em` vs `px` vs `%` vs `vw/vh`?
- **`px`** — absolute.
- **`em`** — relative to **parent's** font-size (compounds).
- **`rem`** — relative to **root** font-size (predictable; preferred for spacing/type).
- **`%`** — relative to parent dimension.
- **`vw`/`vh`** — % of viewport width/height.

---

### Q17. What is the CSS cascade & inheritance?
**Cascade** resolves conflicts via **importance → specificity → source order**.
**Inheritance:** some properties (color, font) pass to children; layout properties
(margin, padding) don't. Force with `inherit`/`initial`/`unset`.

---

### Q18. Pseudo-classes vs pseudo-elements?
- **Pseudo-class** (`:hover`, `:focus`, `:nth-child`, `:first-of-type`) — a **state**.
- **Pseudo-element** (`::before`, `::after`, `::placeholder`) — a **virtual element**.
```css
a:hover { color: blue; }
.badge::after { content: '★'; }
```

---

### Q19. `display: none` vs `visibility: hidden` vs `opacity: 0`?
- **`display:none`** — removed from layout (no space, not focusable).
- **`visibility:hidden`** — hidden but **keeps its space**.
- **`opacity:0`** — invisible but **interactive** and takes space.

---

### Q20. What causes reflow vs repaint? (performance)
- **Reflow (layout)** — geometry changes (width, position, adding nodes) → expensive.
- **Repaint** — visual-only changes (color, visibility) → cheaper.
- **Composite-only** — `transform` / `opacity` are GPU-accelerated → cheapest.
**Tip:** animate `transform`/`opacity`, not `top`/`left`/`width`.

**Real-world example:** a slide-in side drawer. Animating `left: -300px → 0` triggers a
**reflow on every frame** → janky on mobile. Switching to
`transform: translateX(-300px) → translateX(0)` runs on the GPU compositor → buttery 60fps.
This kind of swap is exactly the sort of measurable win behind a Lighthouse performance
improvement.

---

### Q21. `z-index` and stacking contexts.
`z-index` only works on **positioned** elements (or flex/grid children). A new
**stacking context** is formed by `position` + `z-index`, `opacity < 1`, `transform`,
`filter`, etc. A child can never escape its parent's stacking context — a common bug.

---

### Q22. CSS variables (custom properties)?
```css
:root { --primary: #1976d2; --space: 8px; }
.button { background: var(--primary); padding: var(--space); }
/* runtime themeable, cascade-aware, unlike SCSS variables */
```
Differ from SCSS variables: **live at runtime**, inherit, and can be changed via JS.

---

## Part C — SCSS / CSS architecture

### Q23. What does SCSS add?
Variables, **nesting**, **mixins**, **functions**, `@extend`, partials/`@use`, loops,
conditionals — compiled to CSS.
```scss
$primary: #1976d2;
@mixin flex-center { display: flex; justify-content: center; align-items: center; }
.card {
  @include flex-center;
  background: lighten($primary, 40%);
  &:hover { background: $primary; }   // & = parent selector
  .title { font-weight: 700; }
}
```

### Q24. Mixin vs `@extend` vs function?
- **Mixin** — reusable block of declarations (can take args); duplicates output.
- **`@extend`** — shares a selector (groups them); can bloat/couple selectors.
- **Function** — returns a **value** (`@function`), used in property values.

### Q25. What is BEM and why?
**Block__Element--Modifier** naming for **flat, predictable, low-specificity** CSS —
avoids deep nesting and specificity wars.
```html
<div class="card card--featured">
  <h2 class="card__title">…</h2>
  <button class="card__btn card__btn--primary">Buy</button>
</div>
```

### Q26. How do you avoid global CSS leaks at scale?
- **BEM / naming conventions**, **CSS Modules**, or framework scoping (Angular
  `ViewEncapsulation`), utility-first (Tailwind), `@layer`/cascade layers, and avoiding
  high-specificity/`!important`.

---

## Part D — More questions (with real-world examples)

### Q27. How do you make a site responsive without a CSS framework?
Mobile-first media queries + fluid units + intrinsic layouts:
```css
.grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(240px, 1fr)); gap: 1rem; }
```
`auto-fit + minmax` reflows a card grid from 1 column on phones to 4 on desktop **with
zero media queries** — a clean, modern answer that impresses.

### Q28. A button looks fine but is unusable on mobile — what do you check?
- **Tap target size** ≥ 44×44px (Apple/WCAG guidance).
- **`:focus-visible`** styles so keyboard/AT users see focus.
- **`viewport` meta** present, no fixed pixel widths causing horizontal scroll.
- **Contrast** ≥ 4.5:1. This "debug a real UI" question shows practical a11y sense.

### Q29. What's the difference between `<button>` and a clickable `<div>`?
A `<button>` is **focusable, keyboard-activatable (Enter/Space), and announced as a
button** by screen readers — all for free. A clickable `<div>` needs `tabindex="0"`,
`role="button"`, and a keydown handler to even approach parity. **Always reach for the
real element first** — this is the single most common a11y mistake in real codebases.

### Q30. How do you load a web font without blocking render / causing layout shift?
- `font-display: swap` so text shows immediately in a fallback, then swaps.
- `<link rel="preload" as="font" crossorigin>` for the critical font.
- Match the fallback's metrics (`size-adjust`) to reduce **CLS** (Cumulative Layout
  Shift) — a Core Web Vital interviewers love.

### Q31. Explain `box-sizing: border-box` with a concrete bug it fixes.
Without it, `width: 100%; padding: 16px` makes the element **wider than its parent**
(content is 100% *plus* 32px of padding) → it overflows and breaks the layout. With
`border-box`, the padding is *included* in the 100%. That's why nearly every CSS reset
sets `*, *::before, *::after { box-sizing: border-box; }`.

### Q32. What are container queries and how do they differ from media queries?
**Media queries** respond to the **viewport**; **container queries** respond to a
**parent element's** size. This makes a component truly reusable — it adapts to wherever
it's placed, not just the screen width.
```css
.card-wrap { container-type: inline-size; }
@container (min-width: 400px) { .card { display: grid; grid-template-columns: 1fr 2fr; } }
```
**Real-world:** the same product card sits in a wide main column (2-column layout) and a
narrow sidebar (stacked) — with container queries it "just knows," no global breakpoints.

### Q33. What does the `:has()` selector enable (the "parent selector")?
`:has()` styles an element based on its **descendants/siblings** — something CSS could
never do before.
```css
.card:has(img) { padding-top: 0; }          /* card that contains an image */
label:has(+ input:invalid) { color: red; }  /* label before an invalid input */
```
**Real-world:** highlight a form row when its input is invalid, or change a layout only
when a slot has content — logic that used to require JavaScript.

### Q34. How do you build dark mode in CSS?
Combine `prefers-color-scheme` with **CSS variables** so you swap a small set of tokens,
not every rule:
```css
:root { --bg: #fff; --fg: #111; }
@media (prefers-color-scheme: dark) { :root { --bg: #111; --fg: #eee; } }
body { background: var(--bg); color: var(--fg); }
/* a [data-theme="dark"] override lets users toggle manually too */
```
**Real-world:** because variables cascade at runtime, a theme toggle flips one attribute
on `<html>` and the whole app re-themes instantly — no rebuild, no duplicate stylesheets.

### Q35. What are CSS cascade layers (`@layer`) and why use them?
`@layer` lets you define **explicit priority order** for groups of rules, so specificity
wars stop. Later layers win regardless of selector specificity.
```css
@layer reset, base, components, utilities;   /* utilities always beat components */
@layer components { .btn { color: blue; } }
@layer utilities { .text-red { color: red; } } /* wins even if less specific */
```
**Real-world:** integrating a third-party CSS framework without `!important` battles —
put vendor styles in a low layer and your overrides in a higher one.

### Q36. How do you keep an element's aspect ratio (e.g., 16:9 video)?
Modern way is one line:
```css
.video { aspect-ratio: 16 / 9; width: 100%; }
```
**Real-world:** responsive video/thumbnail embeds that never cause layout shift (good for
**CLS**) — far cleaner than the old padding-top percentage hack.

### Q37. How do you manage focus for accessibility in a single-page app?
When a route changes or a modal opens, sighted users see the change but screen-reader and
keyboard users don't — you must **move focus** programmatically.
- On route change, focus the new page's `<h1>` (or a skip-link target).
- When a modal opens, **trap focus** inside it and restore focus to the trigger on close.
- Use `:focus-visible` so focus rings show for keyboard users without bothering mouse
  users.
**Real-world:** this is one of the most common accessibility gaps in Angular/React SPAs
and a strong senior-level answer.

---

### Rapid-fire
- **`min/max/clamp()`?** Fluid sizing: `clamp(min, preferred, max)`.
- **`gap` in Flexbox?** Supported in modern browsers — spacing without margins.
- **`object-fit`?** How an image fills its box (`cover`, `contain`).
- **`overflow` values?** `visible|hidden|scroll|auto|clip`.
- **`transition` vs `animation`?** Transition = state A→B; animation = keyframes/loops.
- **`:nth-child(2n)`?** Selects even children.
- **Critical CSS?** Inline above-the-fold CSS for faster FCP.
- **`will-change`?** Hint to the browser to promote a layer (use sparingly).

---

### Likely deep-dive chain
> "Box model?" → "`content-box` vs `border-box`?" → "Specificity of this selector?" →
> "Flexbox vs Grid?" → "What triggers reflow and how do you avoid it?"
