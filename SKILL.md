---
name: smoothy-animation-skill
description: Make an app's UI feel smooth, soft and "alive" with consistent micro-interactions — springy button presses, hover lifts, chips, color fades, toasts, page transitions, dropdowns, selects, tabs, modals, switches, tooltips, accordions, staggered lists, drag-and-drop feedback, and spring-driven custom/canvas widgets — plus reduced-motion support. Works on any platform: web (CSS, JS, React, Vue, Svelte, Framer Motion), mobile (React Native/Reanimated, Flutter, SwiftUI/UIKit, Jetpack Compose) and desktop (Avalonia, WPF, .NET MAUI, Electron/Tauri). Use when the user asks to add/improve UI animations, transitions, micro-interactions or "make it feel smooth/polished/bouncy", builds an animated component, or asks how to animate a UI element. Not for rendered videos or motion-graphics films (use the HyperFrames skills for those).
---

# Smoothy Animation

A small, opinionated motion system for app UIs. It came from studying [Filee](https://github.com/KnifeLemon/Filee), a desktop app that people describe as unusually smooth. Its whole feel comes from a few rules applied the same way everywhere, not from a big animation library.

## The 6 rules

1. **Animate only transform and opacity.** These are scale, translate, rotate and opacity. Colors and shadows may fade too. Never animate width, height, top, left, margin or padding to move things, because they force a re-layout and stutter. The exception is a collapse/expand, which uses the platform's size-animation primitive (see the recipes below).
2. **Small things bounce, big things glide.** A small element moving a short distance gets a soft overshoot curve (back-out). Big surfaces such as pages, panels and sheets get a plain ease-out with no overshoot. **The bounce strength must shrink as the distance grows.**
3. **Closing is faster than opening and never bounces.** Open: about 250–420 ms with a bounce. Close: about 120–160 ms with ease-in. The user is done with it, so get it out of the way.
4. **Colors never bounce.** A color that overshoots flashes a different color. Fade colors over about 150 ms linearly or with ease-out, separately from the transform.
5. **Anything that follows the pointer or finger uses a spring, not a duration.** Springs keep their velocity when the target changes mid-flight. A fixed-duration tween restarts and jerks.
6. **Respect reduced motion.** When the OS setting is on, drop movement (scale, slide, bounce) but keep state feedback (color, selection, opacity). Jump springs straight to their target.

## Motion tokens (use these numbers on every platform)

Create **one** tokens file or object in the project and reference it everywhere. Never scatter literal durations. If the project already has a design system with motion tokens, extend that instead.

| Token | Value | Used for |
|---|---|---|
| `ease.back` | back-out, overshoot ~10% → CSS `cubic-bezier(0.34, 1.56, 0.64, 1)` | small things appearing, pressing, hovering |
| `ease.out` | cubic-out → `cubic-bezier(0.33, 1, 0.68, 1)` | pages, panels, sheets, collapses |
| `ease.in` | cubic-in → `cubic-bezier(0.32, 0, 0.67, 0)` | closing / dismissing |
| `dur.color` | 150 ms | color, border, shadow fades |
| `dur.press` | 220 ms | button press and release |
| `dur.hover` | 250 ms | hover scale or nudge (chips, list items, cards) |
| `dur.page` | 280 ms | page / screen content entering |
| `dur.pop` | 350 ms | toast, dialog, popover entering |
| `dur.open` | 420 ms | radial menus and big staggered reveals (per item) |
| `dur.close` | 160 ms (menus 120 ms) | any dismissal |
| `stagger` | 28 ms | delay between consecutive items in a revealed list |
| `spring.snappy` | stiffness 320, damping 20, mass 1 (ζ ≈ 0.56, overshoots ~12%) | hover/press highlight, pointer-following |
| `spring.settle` | stiffness 320, damping 36 (critical, ζ ≈ 1) | returning to rest with **no** bounce |
| `spring.layout` | stiffness 420, damping 36 (ζ ≈ 0.88) | items rearranging, reflowing, drag-reorder |

To convert spring parameters for a platform API:
- Damping ratio: ζ = damping / (2·√(stiffness·mass)).
- Critical damping: damping = 2·√stiffness.
- Response (period): 2π / √stiffness. For 320 that is ≈ 0.35 s.

## Which tool for which job

- **The state changes and it should animate to the new state** (hover, pressed, selected, checked, open): use an implicit **transition** or animated value on that property.
- **Something appears once** (a toast is added, a page mounts, an item is inserted): use a **keyframe/enter animation** that runs on mount.
- **The target can change mid-animation** (pointer tracking, drag, a hover that moves between items, gestures): use a **spring**.
- **A custom-drawn widget** (canvas, radial menu, chart, game-like UI): run a **frame loop**. On each frame, step the springs with the real `dt`, redraw, and stop when everything has settled. See `references/spring-engine.md`.

## Element recipes

The values are platform-neutral. The platform reference files show the code.

| Element | Open / active | Rest / close | Notes |
|---|---|---|---|
| **Button press** | scale 0.95 | back to 1 with `ease.back`, `dur.press` | The release bounce is the "soft" feel |
| **Chip / tag hover** | scale 1.05 + faint accent background | 1 | Scale uses `ease.back` over `dur.hover`; the color uses `dur.color` |
| **List / sidebar item hover** | translate 3px toward the content | 0 | **Mirror in RTL** (-3px) |
| **Selection / theme / accent change** | color fade | — | `dur.color`, no bounce |
| **Drop zone (drag over)** | scale 1.015 + accent border and background | 1 | Web: count dragenter/dragleave so it doesn't flicker |
| **Card lift (dragging / selected)** | bigger shadow + accent border, translateY -3px | flat | shadow over `dur.color` |
| **Toast / snackbar** | from opacity 0, offset 40px from its screen edge, over `dur.pop` with `ease.back` | opacity 0 and offset again, 200 ms `ease.in`, then remove | Remove it only after the exit animation ends |
| **Page / screen content** | from opacity 0, translateY 10px, over `dur.page` with `ease.out` | — | **No bounce**, because it is big |
| **Dropdown / menu / popover** | from opacity 0, scale 0.95, translateY -6px; transform-origin at the trigger; 260 ms `ease.back`; items staggered 28 ms | 120 ms `ease.in`, no stagger | Also hide it from input and focus when closed |
| **Select** | the dropdown recipe + chevron rotates 180° with `ease.back` + ✓ pops from scale 0.4 + the new value slides up 8px | — | Close 120 ms *after* the pick, so the ✓ pop is visible |
| **Tabs** | **one** indicator slides (x and width) with 350 ms `ease.back`; new content slides in 24px from the side of the tab you went to | — | Mirror the direction in RTL |
| **Modal / dialog / sheet** | backdrop fades 200 ms with `ease.out`; panel from scale 0.9, 320 ms `ease.back` (a sheet: slide from its edge with `ease.out`, no bounce) | both 160 ms `ease.in` | Render the closed state for one frame before switching to open |
| **Switch / toggle** | the thumb slides with `ease.back` over 300 ms; the track color fades over `dur.color` | — | While pressed, the thumb stretches about 20% wide |
| **Tooltip** | wait 300 ms, then opacity in with a 4px rise and `ease.back` | hide in about 100 ms with **no** delay | Put the delay only on the "show" transition |
| **Accordion / collapse** | the height animates from 0 to auto with `ease.out` over 300 ms; the chevron rotates 90° with `ease.back` | same, reversed | Clip the content, not its container |
| **List reveal** | each item from opacity 0 and translateY -4px; item delay = i × 28 ms | no stagger on exit | Cap the total stagger at about 300 ms for long lists |
| **Pulse / "saved!" / badge bump** | a spring with target 0 that you kick with a velocity; scale = 1 + value × 0.08 | — | stiffness 260, damping 12 |
| **Hover highlight that follows the pointer** | spring to 1 with `spring.snappy` | spring to 0 with `spring.settle` | It bounces in and settles out, so there's no flash on leave |

## Workflow when applying this skill to a project

1. **Detect the platform and UI stack** from the repo: `package.json`, `pubspec.yaml`, `*.csproj`, `Package.swift`/`*.xcodeproj`, `build.gradle(.kts)`, and so on. Check for an existing animation library before adding one. Prefer what is already installed, then the platform's built-in tools.
2. **Read the matching reference file** (only that one):
   - Web (CSS, React, Vue, Svelte, Angular, Electron, Tauri) → `references/web.md`
   - React Native / Expo → `references/react-native.md`
   - Flutter → `references/flutter.md`
   - iOS / macOS (SwiftUI, UIKit) → `references/apple.md`
   - Android (Jetpack Compose, Views) → `references/android.md`
   - .NET desktop (Avalonia, WPF, .NET MAUI, WinUI) → `references/dotnet.md`
   - Custom-drawn widgets, games, canvas, or any platform without spring support → `references/spring-engine.md`
3. **Add the tokens** in the project's idiom: CSS variables, a TS/Dart/Swift/Kotlin constants file, or a XAML resource dictionary.
4. **Apply the recipes** to the elements the user asked about. If they asked for "make the app smooth" in general, start with buttons, list hover, color fades, page enter and dialogs. These five give most of the feel.
5. **Wire up reduced motion** once, globally.
6. **Verify** (see the checklist below).

## Anti-patterns to catch in review

| Symptom | Cause | Fix |
|---|---|---|
| Feels sluggish | the duration is over 400 ms for a UI response | Use the tokens; only staggered reveals reach 420 ms per item |
| A page or sheet "wobbles" | back-out on a big surface | `ease.out` |
| Stutter or jank | animating layout properties | transform and opacity |
| A color flashes | overshoot on a color | linear or ease-out color fade |
| Closing feels annoying | close mirrors open | close faster, with ease-in and no bounce |
| Jerks when the pointer moves fast | a fixed-duration tween restarted on every move | spring |
| A spring explodes on a slow device | one big integration step for a late frame | sub-step `dt` (≤ 1/120 s); see spring-engine |
| CPU busy while idle | the frame loop never stops | stop when every spring has settled |
| `transition: all` | animates unintended properties | list the properties explicitly |
| A hidden element still takes clicks or focus | faded with opacity only | also hide it from input once the close ends |
| The enter animation doesn't play | the element was shown and given its "open" state in the same frame | wait one frame, or use the platform's enter-transition API |

## Verification checklist

- [ ] One tokens file; no stray magic durations.
- [ ] Each animated property is transform, opacity, color or shadow (or a sanctioned collapse primitive).
- [ ] Open bounces and close is quicker; colors don't bounce; big surfaces don't bounce.
- [ ] Pointer- or gesture-driven motion uses springs; frame loops sub-step and stop when settled.
- [ ] Reduced motion: movement removed, state feedback kept, and end-of-animation callbacks still fire (use a near-zero duration, not "none").
- [ ] RTL: horizontal nudges and slide directions are mirrored.
- [ ] Tested on a low-end device or with CPU throttling, and nothing jumps or explodes.
