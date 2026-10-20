# smoothy-animation-skill

A [Claude Code](https://claude.com/claude-code) skill that makes any app feel smooth: springy buttons, chips, dropdowns, tabs, page transitions, toasts and more. It works on **web, mobile and desktop** from one shared set of motion tokens.

The motion system is distilled from [Filee](https://github.com/KnifeLemon/Filee), an Avalonia desktop app with very polished micro-interactions, then generalised to every major UI platform.

## Install

**Personal (all your projects):**

```bash
git clone https://github.com/saeedkhal/smoothy-animation-skill ~/.claude/skills/smoothy-animation-skill
```

**One project only (shared with your team):**

```bash
git clone https://github.com/saeedkhal/smoothy-animation-skill .claude/skills/smoothy-animation-skill
```

Restart Claude Code if it is already running.

## Use

Type `/smoothy-animation-skill`, or just ask, for example:

- "make this app feel smooth"
- "add a nice animation to this dropdown"
- "the tabs feel stiff, fix them"

Claude detects your platform, adds a single motion-tokens file, applies the recipes, adds reduced-motion support and verifies the result.

## What's inside

| File | Covers |
|---|---|
| `SKILL.md` | The 6 rules, motion tokens, element recipes, workflow, anti-patterns, checklist |
| `references/web.md` | CSS, JS, React, Framer Motion, Vue, Svelte, GSAP, Electron, Tauri |
| `references/react-native.md` | Reanimated, Expo, gestures, haptics |
| `references/flutter.md` | Implicit animations, routes, spring physics |
| `references/apple.md` | SwiftUI, UIKit, AppKit |
| `references/android.md` | Jetpack Compose, Views |
| `references/dotnet.md` | Avalonia, WPF, .NET MAUI, WinUI |
| `references/spring-engine.md` | A stable spring + frame loop for canvas / custom-drawn widgets on any platform |

### The 6 rules

1. Animate only transform and opacity.
2. Small things bounce, big things glide.
3. Closing is faster than opening and never bounces.
4. Colors never bounce.
5. Anything that follows the pointer uses a spring.
6. Always respect reduced motion.

### Core tokens

| Token | Value |
|---|---|
| ease.back | `cubic-bezier(0.34, 1.56, 0.64, 1)` (~10% overshoot) |
| ease.out | `cubic-bezier(0.33, 1, 0.68, 1)` |
| ease.in | `cubic-bezier(0.32, 0, 0.67, 0)` |
| durations | color 150 · press 220 · hover 250 · page 280 · pop 350 · open 420 · close 160 ms |
| stagger | 28 ms |
| springs | snappy 320/20 · settle 320/36 · layout 420/36 (stiffness/damping, mass 1) |

## Notes

The web and spring-engine code has been tested in a browser. The snippets for the other platforms follow their documented APIs but may need small adjustments for your framework version. Issues and PRs welcome.

## License

[MIT](LICENSE)
