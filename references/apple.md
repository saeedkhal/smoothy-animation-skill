# iOS / macOS (SwiftUI · UIKit · AppKit)

SwiftUI springs are first-class. Use them for almost everything, and use timing curves only for big surfaces, colors and closing.

## Tokens

```swift
import SwiftUI

enum Motion {
    // Springs. These are the same physics as the token table (mass 1).
    static let snappy = Animation.interpolatingSpring(mass: 1, stiffness: 320, damping: 20)
    static let settle = Animation.interpolatingSpring(mass: 1, stiffness: 320, damping: 36)
    static let layout = Animation.interpolatingSpring(mass: 1, stiffness: 420, damping: 36)
    // Same feel expressed as response/damping fraction: .spring(response: 0.35, dampingFraction: 0.56)

    static let back = Animation.timingCurve(0.34, 1.56, 0.64, 1, duration: 0.22)
    static let page = Animation.timingCurve(0.33, 1, 0.68, 1, duration: 0.28)
    static let close = Animation.timingCurve(0.32, 0, 0.67, 0, duration: 0.16)
    static let color = Animation.easeOut(duration: 0.15)
    static let stagger = 0.028
}
```

## Recipes

```swift
// Press: a ButtonStyle so every button gets it
struct Squish: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .scaleEffect(configuration.isPressed ? 0.95 : 1)
            .animation(Motion.snappy, value: configuration.isPressed)
    }
}

// Hover (macOS / iPadOS pointer)
.scaleEffect(hovering ? 1.05 : 1).animation(Motion.snappy, value: hovering).onHover { hovering = $0 }
// List nudge: .offset(x: hovering ? 3 : 0) // SwiftUI mirrors .leading/.trailing layouts, but offsets are not mirrored: flip manually for RTL using @Environment(\.layoutDirection)

// Color fade
.background(selected ? Color.accentColor.opacity(0.18) : .clear).animation(Motion.color, value: selected)

// Enter / exit
.transition(.asymmetric(
    insertion: .scale(scale: 0.95, anchor: .top).combined(with: .opacity).animation(Motion.snappy),
    removal: .opacity.animation(Motion.close)))
// Page content: .transition(.opacity.combined(with: .offset(y: 10))) with Motion.page
// Toast: .transition(.move(edge: .trailing).combined(with: .opacity))
// Stagger: .animation(Motion.snappy.delay(Double(index) * Motion.stagger), value: shown)

// Tabs indicator: one Capsule with .matchedGeometryEffect(id: "ink", in: ns), selected change wrapped in withAnimation(Motion.snappy)
// Chevron: .rotationEffect(.degrees(open ? 180 : 0)).animation(Motion.snappy, value: open)
// Value swap: Text(value).id(value).transition(.push(from: .bottom)) or .contentTransition(.numericText())
// Pulse: .symbolEffect(.bounce, value: count) for SF Symbols, or .phaseAnimator / keyframeAnimator for custom
// Sheets and alerts: keep the system presentations (they already follow the platform's motion)
```

Use `withAnimation(Motion.snappy) { state.toggle() }` to animate a state change from an action.

## UIKit

```swift
UIView.animate(withDuration: 0.35, delay: 0, usingSpringWithDamping: 0.56, initialSpringVelocity: 0, options: [.allowUserInteraction]) {
    view.transform = .identity
}
// Exact physics: UISpringTimingParameters(mass: 1, stiffness: 320, damping: 20, initialVelocity: .zero) with UIViewPropertyAnimator
// Close: UIViewPropertyAnimator(duration: 0.16, controlPoint1: CGPoint(x: 0.32, y: 0), controlPoint2: CGPoint(x: 0.67, y: 0))
```

AppKit: `NSAnimationContext.runAnimationGroup` with `context.timingFunction = CAMediaTimingFunction(controlPoints: 0.34, 1.56, 0.64, 1)`, or Core Animation `CASpringAnimation` with `stiffness = 320` and `damping = 20`.

## Reduced motion

Read `@Environment(\.accessibilityReduceMotion) var reduceMotion` (UIKit: `UIAccessibility.isReduceMotionEnabled`). When it's on, use `reduceMotion ? .none : Motion.snappy`, and replace scale/move transitions with `.opacity`.

## Haptics

Use `.sensoryFeedback(.impact(weight: .light), trigger: value)` (iOS 17+) or `UIImpactFeedbackGenerator(style: .light)` on toggles and confirmations.
