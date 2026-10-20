# React Native / Expo

Prefer **react-native-reanimated** (it runs on the UI thread). If it isn't installed, use the built-in `Animated` API with `useNativeDriver: true` (transform and opacity only). Add Reanimated only if the user agrees.

## Tokens

```ts
// motion.ts
import { Easing, ReduceMotion } from "react-native-reanimated";

export const ease = {
  back: Easing.bezier(0.34, 1.56, 0.64, 1),
  out: Easing.bezier(0.33, 1, 0.68, 1),
  in: Easing.bezier(0.32, 0, 0.67, 0),
};
export const dur = { color: 150, press: 220, hover: 250, page: 280, pop: 350, open: 420, close: 160, stagger: 28 };
export const spring = {
  snappy: { stiffness: 320, damping: 20, mass: 1, reduceMotion: ReduceMotion.System },
  settle: { stiffness: 320, damping: 36, mass: 1, reduceMotion: ReduceMotion.System },
  layout: { stiffness: 420, damping: 36, mass: 1, reduceMotion: ReduceMotion.System },
};
```

`Easing.out(Easing.back(1.55))` gives the exact back-out curve if you prefer it to the bezier approximation.

## Recipes

```tsx
// Pressable with a springy press (mobile has no hover; press IS the main micro-interaction)
const scale = useSharedValue(1);
const style = useAnimatedStyle(() => ({ transform: [{ scale: scale.value }] }));
<Pressable
  onPressIn={() => (scale.value = withSpring(0.95, spring.snappy))}
  onPressOut={() => (scale.value = withSpring(1, spring.snappy))}
>
  <Animated.View style={style}>{children}</Animated.View>
</Pressable>

// Enter / exit with layout animations
<Animated.View
  entering={FadeInDown.duration(dur.page).easing(ease.out)}      // page content: no bounce
  exiting={FadeOut.duration(dur.close).easing(ease.in)}
/>
<Animated.View entering={ZoomIn.springify().stiffness(320).damping(20)} exiting={ZoomOut.duration(120)} /> // menu / dialog
// Staggered list: entering={FadeInUp.delay(index * dur.stagger).springify().stiffness(320).damping(20)}
// Toast: entering={SlideInRight.springify().stiffness(320).damping(20)} exiting={SlideOutRight.duration(200).easing(ease.in)}
// Reorder / layout changes: layout={LinearTransition.springify().stiffness(420).damping(36)}

// Color fade
const progress = useSharedValue(0);
const bg = useAnimatedStyle(() => ({ backgroundColor: interpolateColor(progress.value, [0, 1], [off, on]) }));
progress.value = withTiming(checked ? 1 : 0, { duration: dur.color });

// Switch thumb
thumbX.value = withSpring(checked ? 20 : 0, spring.snappy);

// Tabs indicator: measure each tab with onLayout, then
inkX.value = withSpring(tabX, spring.snappy); inkW.value = withSpring(tabW, spring.snappy);

// Bottom sheet / modal: backdrop opacity withTiming(1, {duration: 200, easing: ease.out});
// sheet translateY withTiming(0, {duration: dur.page, easing: ease.out}) (big surface: no bounce); close with ease.in in dur.close.
// A centered dialog may use scale 0.9 -> 1 with spring.snappy.

// Pulse
scale.value = withSequence(withTiming(1.08, { duration: 90 }), withSpring(1, { stiffness: 260, damping: 12 }));
```

## Gestures

For drag or swipe, set the value directly during the gesture (`translateX.value = e.translationX`). On release, use `withSpring(target, { ...spring.snappy, velocity: e.velocityX })` so the gesture's velocity carries into the animation. Use react-native-gesture-handler.

## Reduced motion

`ReduceMotion.System` in the configs handles Reanimated animations. For your own logic, `useReducedMotion()` from Reanimated or `AccessibilityInfo.isReduceMotionEnabled()` gives you the setting.

## Haptics pair well

A light haptic on press-in or on toggle (`expo-haptics` `impactAsync(Light)`) makes the bounce feel physical. Keep it subtle and only for confirmations.
