# Android (Jetpack Compose · Views)

Compose springs take a **damping ratio** (ζ) instead of a damping coefficient: ζ = damping / (2√stiffness). snappy 320/20 → ζ 0.56, settle → 1.0, layout 420/36 → 0.88.

## Tokens

```kotlin
object Motion {
    val Back = CubicBezierEasing(0.34f, 1.56f, 0.64f, 1f)
    val Out = CubicBezierEasing(0.33f, 1f, 0.68f, 1f)
    val In = CubicBezierEasing(0.32f, 0f, 0.67f, 0f)
    const val COLOR = 150; const val PRESS = 220; const val HOVER = 250; const val PAGE = 280
    const val POP = 350; const val OPEN = 420; const val CLOSE = 160; const val STAGGER = 28
    fun <T> snappy() = spring<T>(dampingRatio = 0.56f, stiffness = 320f)
    fun <T> settle() = spring<T>(dampingRatio = 1f, stiffness = 320f)
    fun <T> layout() = spring<T>(dampingRatio = 0.88f, stiffness = 420f)
}
```

## Recipes

```kotlin
// Press
val interaction = remember { MutableInteractionSource() }
val pressed by interaction.collectIsPressedAsState()
val scale by animateFloatAsState(if (pressed) 0.95f else 1f, Motion.snappy(), label = "press")
Button(onClick = onClick, interactionSource = interaction, modifier = Modifier.graphicsLayer { scaleX = scale; scaleY = scale }) { ... }

// Hover (desktop / ChromeOS / Compose Multiplatform): collectIsHoveredAsState() -> scale 1.05 or offset 3.dp
// For RTL, flip the offset sign using LocalLayoutDirection.current

// Color fade
val bg by animateColorAsState(if (selected) accent else surface, tween(Motion.COLOR), label = "bg")

// Enter / exit
AnimatedVisibility(
    visible = open,
    enter = fadeIn(tween(180, easing = Motion.Out)) + scaleIn(Motion.snappy(), initialScale = 0.95f, transformOrigin = TransformOrigin(0.5f, 0f)),
    exit = fadeOut(tween(120, easing = Motion.In)) + scaleOut(tween(120, easing = Motion.In), targetScale = 0.95f),
) { Menu() }
// Page content: fadeIn(tween(PAGE, easing = Out)) + slideInVertically(tween(PAGE, easing = Out)) { it / 30 }  — no bounce
// Toast: slideInHorizontally(Motion.snappy()) { it / 4 } + fadeIn()
// Stagger: per item, LaunchedEffect(Unit) { delay(index * STAGGER.toLong()); visible = true }
// Value swap: AnimatedContent(targetState = value, transitionSpec = { (slideInVertically { it / 3 } + fadeIn()) togetherWith fadeOut(tween(100)) })
// Chevron: animateFloatAsState(if (open) 180f else 0f, Motion.snappy()) -> Modifier.rotate()
// Accordion: Modifier.animateContentSize(tween(300, easing = Motion.Out))
// Tabs indicator: animateDpAsState for offset and width with Motion.snappy(), or TabRow's indicator with Modifier.tabIndicatorOffset
// Lazy lists: Modifier.animateItem(placementSpec = Motion.layout())
// Switch thumb: animateDpAsState(if (on) 20.dp else 0.dp, Motion.snappy())
```

For gestures, use `Animatable` and call `animateTo(target, Motion.snappy(), initialVelocity = velocity)` after `VelocityTracker`/`draggable` ends. For custom drawing, use `Canvas` with a `withFrameNanos` loop stepping the springs from `spring-engine.md`.

## Views (legacy)

```kotlin
view.animate().scaleX(0.95f).scaleY(0.95f).setDuration(220).setInterpolator(PathInterpolator(0.34f, 1.56f, 0.64f, 1f))
SpringAnimation(view, DynamicAnimation.SCALE_X, 1f).apply {
    spring.stiffness = 320f; spring.dampingRatio = 0.56f; start()
}
```

## Reduced motion

Android signals reduced motion through the animator duration scale. Read it with `Settings.Global.getFloat(context.contentResolver, Settings.Global.ANIMATOR_DURATION_SCALE, 1f)`; `0f` means animations are off. Compose's built-in animations already honour this scale. In your own frame loops and springs, snap to the target when the scale is 0.
