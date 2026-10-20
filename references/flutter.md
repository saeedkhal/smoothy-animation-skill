# Flutter

Use the implicit `Animated*` widgets for state changes, `AnimatedSwitcher` and `PageRouteBuilder` for enter/exit, and `AnimationController` with `SpringSimulation` for springs and gestures. The `flutter_animate` package is fine if the project already has it.

## Tokens

```dart
// motion.dart
import 'package:flutter/animation.dart';
import 'package:flutter/physics.dart';

abstract final class Motion {
  static const back = Cubic(0.34, 1.56, 0.64, 1); // ≈ Curves.easeOutBack (which is the 10% version)
  static const out = Cubic(0.33, 1, 0.68, 1);      // = Curves.easeOutCubic
  static const inn = Cubic(0.32, 0, 0.67, 0);      // = Curves.easeInCubic
  static const color = Duration(milliseconds: 150);
  static const press = Duration(milliseconds: 220);
  static const hover = Duration(milliseconds: 250);
  static const page = Duration(milliseconds: 280);
  static const pop = Duration(milliseconds: 350);
  static const open = Duration(milliseconds: 420);
  static const close = Duration(milliseconds: 160);
  static const stagger = Duration(milliseconds: 28);
  static final snappy = SpringDescription(mass: 1, stiffness: 320, damping: 20);
  static final settle = SpringDescription(mass: 1, stiffness: 320, damping: 36);
  static final layout = SpringDescription(mass: 1, stiffness: 420, damping: 36);

  /// Returns Duration.zero when the OS asks for reduced motion.
  static Duration of(BuildContext c, Duration d) =>
      MediaQuery.of(c).disableAnimations ? Duration.zero : d;
}
```

## Recipes

```dart
// Button press
class Squish extends StatefulWidget { /* child, onTap */ }
// state: bool down = false;
GestureDetector(
  onTapDown: (_) => setState(() => down = true),
  onTapUp: (_) => setState(() => down = false),
  onTapCancel: () => setState(() => down = false),
  onTap: widget.onTap,
  child: AnimatedScale(scale: down ? 0.95 : 1, duration: Motion.press, curve: Motion.back, child: widget.child),
);

// Hover (desktop / web): MouseRegion + AnimatedScale(1.05) / AnimatedSlide(Offset(3 / width, 0)); mirror the offset for RTL via Directionality.of(context)
// Color fade: AnimatedContainer(duration: Motion.color, color: selected ? accent : surface)
// Lift: AnimatedContainer(duration: Motion.color, decoration: BoxDecoration(boxShadow: lifted ? [bigShadow] : []))

// Page route: content rises 10px and fades, no bounce
PageRouteBuilder(
  transitionDuration: Motion.page,
  reverseTransitionDuration: Motion.close,
  pageBuilder: (_, __, ___) => page,
  transitionsBuilder: (_, a, __, child) {
    final c = CurvedAnimation(parent: a, curve: Motion.out, reverseCurve: Motion.inn);
    return FadeTransition(opacity: c, child: SlideTransition(
      position: Tween(begin: const Offset(0, 0.02), end: Offset.zero).animate(c), child: child));
  },
);

// Dialog: showGeneralDialog(transitionDuration: Motion.pop, transitionBuilder: ...ScaleTransition(0.9 -> 1, curve: Motion.back, reverseCurve: Motion.inn))
// Menu: wrap the overlay in ScaleTransition(alignment: Alignment.topCenter) + FadeTransition; items with Interval(i * 0.06, 1, curve: Motion.back) for stagger
// Swap value (select / counter): AnimatedSwitcher(duration: 300ms, switchInCurve: Motion.back, transitionBuilder: slide-up + fade, child: Text(value, key: ValueKey(value)))
// Chevron: AnimatedRotation(turns: open ? 0.5 : 0, duration: 300ms, curve: Motion.back)
// Accordion: AnimatedSize(duration: 300ms, curve: Motion.out, child: open ? body : const SizedBox.shrink())
// Tabs indicator: AnimatedPositioned(left: x, width: w, duration: 350ms, curve: Motion.back)
// Switch: AnimatedAlign(alignment: on ? end : start, duration: 300ms, curve: Motion.back) + AnimatedContainer track color in Motion.color
// Tooltip: Tooltip(waitDuration: Duration(milliseconds: 300), ...)
```

## Springs and gestures

```dart
final c = AnimationController.unbounded(vsync: this);
// on drag end, carry the gesture velocity into the spring:
c.animateWith(SpringSimulation(Motion.snappy, c.value, target, velocity));
```

For a custom-drawn widget, put a `CustomPainter(repaint: controller)` behind a `Ticker` and step the springs from `spring-engine.md` with the real elapsed time.

## Reduced motion

Check `MediaQuery.of(context).disableAnimations`. Pass durations through `Motion.of(context, …)`, and jump springs straight to their target.
