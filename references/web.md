# Web (CSS · JS · React · Vue · Svelte · Electron · Tauri)

Electron and Tauri apps are web UIs, so everything here applies to them.

## Tokens

```css
:root {
  --ease-back: cubic-bezier(0.34, 1.56, 0.64, 1);
  --ease-out: cubic-bezier(0.33, 1, 0.68, 1);
  --ease-in: cubic-bezier(0.32, 0, 0.67, 0);
  --dur-color: 150ms;
  --dur-press: 220ms;
  --dur-hover: 250ms;
  --dur-page: 280ms;
  --dur-pop: 350ms;
  --dur-open: 420ms;
  --dur-close: 160ms;
  --stagger: 28ms;
  --dir: 1; /* flips nudges in RTL */
}
:dir(rtl) { --dir: -1; }
```

If the project uses Tailwind, add these to `theme.extend.transitionTimingFunction` / `transitionDuration` (or `@theme` in v4) instead of raw CSS.

## Recipes

```css
/* Button press */
.btn { transition: transform var(--dur-press) var(--ease-back), background-color var(--dur-color), border-color var(--dur-color); }
.btn:active { transform: scale(0.95); }

/* Chip hover: scale bounces, color does not */
.chip { transition: transform var(--dur-hover) var(--ease-back), background-color var(--dur-color); }
.chip:hover { transform: scale(1.05); background: color-mix(in srgb, var(--accent) 10%, transparent); }

/* Sidebar / list item nudge, RTL-aware */
.nav-item { transition: transform var(--dur-hover) var(--ease-back), background-color var(--dur-color); }
.nav-item:hover { transform: translateX(calc(3px * var(--dir))); }

/* Card lift */
.card { transition: box-shadow var(--dur-color), border-color var(--dur-color), transform var(--dur-hover) var(--ease-back); }
.card.lifted { transform: translateY(-3px); box-shadow: 0 8px 24px -6px rgb(0 0 0 / 0.25); }

/* Drop zone */
.drop { transition: background-color 200ms, border-color 200ms, transform 300ms var(--ease-back); }
.drop.over { transform: scale(1.015); border-color: var(--accent); }

/* Toast enter / exit (remove on animationend of the exit) */
.toast { animation: toast-in var(--dur-pop) var(--ease-back) both; }
.toast.leaving { animation: toast-out 200ms var(--ease-in) both; }
@keyframes toast-in { from { opacity: 0; transform: translateX(40px); } }
@keyframes toast-out { to { opacity: 0; transform: translateX(40px) scale(0.96); } }

/* Page content enter: no bounce */
.page-enter { animation: page-in var(--dur-page) var(--ease-out) both; }
@keyframes page-in { from { opacity: 0; transform: translateY(10px); } }

/* Dropdown / popover: open bounces + stagger, close is fast */
.menu {
  transform-origin: top; /* the side the trigger is on */
  opacity: 0; transform: translateY(-6px) scale(0.95); visibility: hidden;
  transition: opacity 120ms var(--ease-in), transform 120ms var(--ease-in), visibility 0s 120ms;
}
.open > .menu {
  opacity: 1; transform: none; visibility: visible;
  transition: opacity 180ms var(--ease-out), transform 260ms var(--ease-back), visibility 0s;
}
.menu > li { opacity: 0; transform: translateY(-4px); transition: opacity 100ms, transform 100ms; }
.open > .menu > li {
  opacity: 1; transform: none;
  transition: opacity 200ms var(--ease-out), transform 300ms var(--ease-back);
  transition-delay: calc(var(--i) * var(--stagger)); /* set style="--i: 0..n" */
}

/* Select extras */
.chevron { transition: transform 300ms var(--ease-back); }
.open .chevron { transform: rotate(180deg); }
.check { opacity: 0; transform: scale(0.4); transition: opacity 150ms, transform 300ms var(--ease-back); }
[aria-selected="true"] > .check { opacity: 1; transform: none; }
.value.swap { animation: swap 300ms var(--ease-back); }
@keyframes swap { from { opacity: 0; transform: translateY(8px); } }

/* Tabs: one sliding indicator */
.tab-ink { position: absolute; left: var(--x); width: var(--w); transition: left 350ms var(--ease-back), width 350ms var(--ease-back); }

/* Modal */
.backdrop { opacity: 0; transition: opacity var(--dur-close) var(--ease-in); }
.backdrop.open { opacity: 1; transition: opacity 200ms var(--ease-out); }
.dialog { opacity: 0; transform: scale(0.9); transition: transform var(--dur-close) var(--ease-in), opacity var(--dur-close) var(--ease-in); }
.backdrop.open .dialog { opacity: 1; transform: none; transition: transform 320ms var(--ease-back), opacity 200ms var(--ease-out); }

/* Switch */
.switch { transition: background-color var(--dur-color); }
.switch .thumb { transition: transform 300ms var(--ease-back), width 150ms; }
.switch:active .thumb { width: 24px; }                  /* 20px at rest */
.switch[aria-checked="true"] .thumb { transform: translateX(20px); }
.switch[aria-checked="true"]:active .thumb { transform: translateX(16px); }

/* Tooltip: delayed show, instant hide */
.tip { opacity: 0; transform: translateY(4px); transition: opacity 100ms, transform 100ms; }
.has-tip:hover .tip, .has-tip:focus-within .tip {
  opacity: 1; transform: none;
  transition: opacity 180ms var(--ease-out) 300ms, transform 250ms var(--ease-back) 300ms;
}

/* Accordion: grid 0fr -> 1fr, clip on the child */
.acc-body { display: grid; grid-template-rows: 0fr; transition: grid-template-rows 300ms var(--ease-out); }
.open > .acc-body { grid-template-rows: 1fr; }
.acc-body > * { overflow: hidden; min-height: 0; }
.open .acc-chevron { transform: rotate(90deg); }
.acc-chevron { transition: transform 300ms var(--ease-back); }

/* Reduced motion: near-zero, not none, so animationend/transitionend still fire */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    transition-delay: 0s !important;
  }
}
```

## JS helpers

```ts
// Restart a CSS enter animation (e.g. on page/tab change)
el.classList.remove("page-enter");
void el.offsetWidth; // force reflow
el.classList.add("page-enter");

// Show an element that was display:none, then animate: wait 2 frames
el.hidden = false;
requestAnimationFrame(() => requestAnimationFrame(() => el.classList.add("open")));
// Hide after the close transition ends
el.classList.remove("open");
el.addEventListener("transitionend", () => (el.hidden = true), { once: true });

// Drop zone without flicker
let depth = 0;
zone.addEventListener("dragenter", (e) => { e.preventDefault(); depth++; zone.classList.add("over"); });
zone.addEventListener("dragover", (e) => e.preventDefault());
zone.addEventListener("dragleave", () => { if (--depth <= 0) { depth = 0; zone.classList.remove("over"); } });
zone.addEventListener("drop", () => { depth = 0; zone.classList.remove("over"); });

// Tabs indicator
ink.style.setProperty("--x", `${tab.offsetLeft}px`);
ink.style.setProperty("--w", `${tab.offsetWidth}px`);

const reduceMotion = () => matchMedia("(prefers-reduced-motion: reduce)").matches;
```

The native `<dialog>` element and the Popover API can use the same classes. For entry animations on elements coming from `display: none`, `@starting-style` plus `transition-behavior: allow-discrete` is the modern pure-CSS option. Check browser support for the project's targets.

## Exact springs in pure CSS

`cubic-bezier` can only overshoot once. To get a real spring curve in CSS, sample the spring from `spring-engine.md` into a `linear()` easing:

```ts
function springLinear(stiffness = 320, damping = 20, samples = 40) {
  // simulate until settled, then emit linear(v0, v1, ...) and the duration
  const h = 1 / 240; let x = 0, v = 0, t = 0; const pts: number[] = [];
  while (t < 3) { pts.push(x); const f = stiffness * (1 - x) - damping * v; v += f * h; x += v * h; t += h;
    if (Math.abs(1 - x) < 0.001 && Math.abs(v) < 0.01) break; }
  const step = Math.max(1, Math.floor(pts.length / samples));
  const vals = pts.filter((_, i) => i % step === 0).concat(1).map((n) => +n.toFixed(3));
  return { easing: `linear(${vals.join(", ")})`, duration: Math.round(t * 1000) };
}
```

## Libraries (use only if already in the project, or the user wants one)

**Motion / Framer Motion (React):**

```tsx
const tokens = {
  press: { type: "spring", stiffness: 320, damping: 20 },
  settle: { type: "spring", stiffness: 320, damping: 36 },
  page: { duration: 0.28, ease: [0.33, 1, 0.68, 1] },
  close: { duration: 0.16, ease: [0.32, 0, 0.67, 0] },
} as const;

<motion.button whileTap={{ scale: 0.95 }} whileHover={{ scale: 1.05 }} transition={tokens.press} />

<AnimatePresence>
  {open && (
    <motion.ul
      initial={{ opacity: 0, scale: 0.95, y: -6 }}
      animate={{ opacity: 1, scale: 1, y: 0, transition: { ...tokens.press, staggerChildren: 0.028 } }}
      exit={{ opacity: 0, scale: 0.95, y: -6, transition: tokens.close }}
      style={{ transformOrigin: "top" }}
    />
  )}
</AnimatePresence>
// Tabs indicator: render one <motion.span layoutId="tab-ink" /> inside the active tab
// Reduced motion: <MotionConfig reducedMotion="user">
```

**Vue:** use `<Transition name="menu">` with `.menu-enter-from`, `.menu-enter-active`, `.menu-leave-active` and `.menu-leave-to`, built from the same values as the CSS above. Give the leave-active class the 120 ms `ease-in`. Use `<TransitionGroup>` for list stagger, setting `--i` per item.

**Svelte:** `transition:scale={{ start: 0.95, duration: 260, easing: backOut }}` from `svelte/easing`, with `out:fade={{ duration: 120 }}`. For pointer-following values, use `spring` from `svelte/motion` (`stiffness`/`damping` are normalized 0–1 there, so tune them by eye).

**GSAP:** `ease: "back.out(1.55)"`, `"power3.out"`, `"power3.in"`; `stagger: 0.028`.
