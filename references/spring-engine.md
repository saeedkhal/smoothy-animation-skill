# Spring engine and frame loop (any language)

Use this for custom-drawn widgets (canvas, Skia, game UIs, radial menus, charts), for platforms without built-in springs, or when the built-in spring can't change its target mid-flight. It is ported from Filee's `DonutMenu` and has been tested.

## Spring (semi-implicit Euler with sub-steps)

```ts
class Spring {
  value = 0;
  velocity = 0;
  static maxStep = 1 / 120; // never integrate more than this per step

  /** Returns true while still moving. */
  step(target: number, dt: number, stiffness = 320, damping = 20): boolean {
    const steps = Math.max(1, Math.ceil(dt / Spring.maxStep));
    const h = dt / steps;
    for (let i = 0; i < steps; i++) {
      const force = stiffness * (target - this.value) - damping * this.velocity;
      this.velocity += force * h;   // velocity first...
      this.value += this.velocity * h; // ...then position (semi-implicit = stable)
    }
    const settled = Math.abs(target - this.value) < 0.001 && Math.abs(this.velocity) < 0.01;
    if (settled) { this.value = target; this.velocity = 0; }
    return !settled;
  }
}
```

**Why the sub-steps matter.** With stiffness 420 / damping 36, a single 50 ms step (one late frame on a busy machine) overshoots further on every frame and blows up to infinity. Splitting `dt` into steps of at most 1/120 s keeps it stable at any frame rate. This was verified numerically. Filee hit exactly this bug: its "slices flew apart on a busy machine".

Port it as is to C#, Dart, Swift, Kotlin, C++ and so on. Use a struct or value type with two doubles.

## Frame loop that sleeps when idle

```ts
let last: number | null = null;
let raf = 0;

function frame(now: number) {
  // Real elapsed time. Clamp it so a backgrounded tab doesn't jump a whole second.
  const dt = last === null ? 1 / 60 : Math.min(0.1, (now - last) / 1000);
  last = now;
  const busy = update(dt); // step every spring/tween, OR their "still moving" results
  draw();
  if (busy) raf = requestAnimationFrame(frame);
  else { raf = 0; last = null; } // everything settled: stop burning CPU
}

/** Call on any input or state change. */
function kick() { if (!raf) raf = requestAnimationFrame(frame); }
```

Platform equivalents for "request the next frame":

| Platform | API |
|---|---|
| Web | `requestAnimationFrame` |
| Avalonia | `TopLevel.GetTopLevel(this).RequestAnimationFrame(cb)` + `InvalidateVisual()` |
| WPF | `CompositionTarget.Rendering` (unsubscribe when settled) |
| Flutter | `Ticker` / `AnimationController.unbounded` + `CustomPainter(repaint: ...)` |
| SwiftUI | `TimelineView(.animation)` + `Canvas`; UIKit `CADisplayLink` |
| Compose | `withFrameNanos { }` inside a `LaunchedEffect` loop + `Canvas` |
| Android Views | `Choreographer.postFrameCallback` |
| React Native | Reanimated `useFrameCallback`, or Skia `useFrame` |

## Patterns built on it

**Hover that bounces in and settles out (no flash on leave):**

```ts
hover[i].step(i === active ? 1 : 0, dt, 320, i === active ? 20 : 36);
```

**Pulse ("saved!", badge bump):** keep the target at 0 and kick the velocity.

```ts
pulse.velocity = 9;                 // on event
pulse.step(0, dt, 260, 12);         // each frame
const scale = 1 + pulse.value * 0.08;
```

**Staggered reveal with a time-based ease.** Use this when the target never changes mid-flight. Closing is faster, nearly simultaneous, and doesn't bounce.

```ts
const easeOutBack = (t: number, c1 = 1.55) => 1 + (c1 + 1) * (t - 1) ** 3 + c1 * (t - 1) ** 2;
const easeInCubic = (t: number) => t * t * t;

clock += dt;
for (let i = 0; i < n; i++) {
  const delay = i * (closing ? 0.01 : 0.028);
  const dur = closing ? 0.16 : 0.42;
  const t = Math.min(1, Math.max(0, (clock - delay) / dur));
  open[i] = closing ? 1 - easeInCubic(t) : easeOutBack(t);
}
// draw item i with: scale = 0.55 + 0.45 * open[i]; rotate = (1 - min(open[i], 1)) * -0.35 rad; alpha = clamp(open[i], 0, 1)
```

**Easing curves for reference** (t in 0..1):

| Name | Formula | Peak overshoot |
|---|---|---|
| cubic out | `1 - (1 - t)^3` | 0 |
| cubic in | `t^3` | 0 |
| back out (c1) | `1 + (c1 + 1)(t - 1)^3 + c1(t - 1)^2` | c1 = 1.55 → 8.5 %, 1.70158 → 10 % |
| AHEasing back out (Avalonia/WPF default) | `p = 1 - t; 1 - p(p² - sin(πp))` | ≈ 38 %, use only for tiny distances |

## Reduced motion

When reduced motion is on, skip integration and snap each spring to its target: `value = target; velocity = 0`. Set the open values to their end state. Keep any timers that drive logic, such as hold-to-confirm, running.

## Canvas hygiene

- Size the backing store as CSS size × `devicePixelRatio`, then scale the context by the same ratio, so lines stay crisp.
- Read theme colors at draw time, and redraw on theme change and on resize.
- Use `touch-action: none` on interactive canvases, and use pointer events so mouse, touch and pen are all handled.
- Give the canvas an accessible label, plus a non-visual fallback for any information it conveys.
