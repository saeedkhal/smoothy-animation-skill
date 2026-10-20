# .NET desktop (Avalonia · WPF · .NET MAUI · WinUI)

This is where the system came from: Filee is an Avalonia app, and every Avalonia snippet below mirrors its real code (`Styles/Soft.axaml`, `Controls/DonutMenu.cs`).

**Important:** the built-in `BackEaseOut` in Avalonia, and WPF's `BackEase` with Amplitude 1, overshoot about **38%**. That's fine for tiny moves like scale 1 → 0.95 or a 3px nudge. For anything that travels further, use a gentler curve: `SplineEasing` with the token bezier, or WPF `BackEase Amplitude="0.3"`.

## Avalonia

### Tokens (resource dictionary)

```xml
<Styles.Resources>
  <ResourceDictionary>
    <BackEaseOut x:Key="SoftEase" />                                       <!-- tiny moves -->
    <SplineEasing x:Key="EaseBack" X1="0.34" Y1="1.56" X2="0.64" Y2="1" /> <!-- bigger moves, ~10% -->
    <SplineEasing x:Key="EaseOut" X1="0.33" Y1="1" X2="0.68" Y2="1" />
    <SplineEasing x:Key="EaseIn" X1="0.32" Y1="0" X2="0.67" Y2="0" />
  </ResourceDictionary>
</Styles.Resources>
```

### Recipes

```xml
<!-- Press -->
<Style Selector="Button, ToggleButton, RepeatButton">
  <Setter Property="Transitions">
    <Transitions>
      <TransformOperationsTransition Property="RenderTransform" Duration="0:0:0.22" Easing="{StaticResource SoftEase}" />
      <BrushTransition Property="Background" Duration="0:0:0.15" />
    </Transitions>
  </Setter>
</Style>
<Style Selector="Button:pressed, ToggleButton:pressed">
  <Setter Property="RenderTransform" Value="scale(0.95)" />
</Style>

<!-- Chip hover -->
<Style Selector="Border.chip:pointerover">
  <Setter Property="RenderTransform" Value="scale(1.05)" />
</Style>

<!-- Nav item nudge (use -3px when FlowDirection is RightToLeft) -->
<Style Selector="ListBox.nav ListBoxItem:pointerover">
  <Setter Property="RenderTransform" Value="translateX(3px)" />
</Style>

<!-- Card lift -->
<Style Selector="Border.card">
  <Setter Property="Transitions"><Transitions><BoxShadowsTransition Property="BoxShadow" Duration="0:0:0.15" /></Transitions></Setter>
</Style>
<Style Selector="Border.card.dragging"><Setter Property="BoxShadow" Value="0 8 24 0 #33000000" /></Style>

<!-- Enter animations. Keyframes may not animate RenderTransform (Filee's comment says so), so slide via Margin -->
<Style Selector="ContentControl.page > :is(Control)">
  <Style.Animations>
    <Animation Duration="0:0:0.28" Easing="CubicEaseOut" FillMode="Forward">
      <KeyFrame Cue="0%"><Setter Property="Opacity" Value="0" /><Setter Property="Margin" Value="0,10,0,-10" /></KeyFrame>
      <KeyFrame Cue="100%"><Setter Property="Opacity" Value="1" /><Setter Property="Margin" Value="0" /></KeyFrame>
    </Animation>
  </Style.Animations>
</Style>
<!-- Toast: same pattern, Duration 0.35, Easing BackEaseOut, Margin "40,0,-40,0" -> 0 -->

<!-- Reduced motion: a class on the Window, set from the OS setting or a user toggle -->
<Style Selector="Window.reduced-motion :is(Button), Window.reduced-motion :is(ToggleButton), Window.reduced-motion ListBoxItem">
  <Setter Property="Transitions" Value="{x:Null}" />
</Style>
```

Other built-in tools:
- **Page switches:** `TransitioningContentControl` with `PageTransition`, for example `<CrossFade Duration="0:0:0.25" />`, `<PageSlide Duration="0:0:0.28" />` or `CompositePageTransition`.
- **Expander:** has `ContentTransition`.
- **Tooltips:** `ToolTip.ShowDelay="300"`.
- **Dropdowns:** animate the popup's border on `ComboBox:dropdownopen /template/ Border#PopupBorder`. Template part names vary by version, so check the theme's template.

Custom-drawn controls: override `Render(DrawingContext)`, request frames with `TopLevel.GetTopLevel(this)!.RequestAnimationFrame(OnFrame)`, call `InvalidateVisual()`, and step the springs from `spring-engine.md` as a `struct Spring`. Use `context.PushTransform(matrix)` and `context.PushOpacity(alpha)` per item.

## WPF

```xml
<Window.Resources>
  <BackEase x:Key="SoftEase" EasingMode="EaseOut" Amplitude="0.3" />
  <CubicEase x:Key="EaseOut" EasingMode="EaseOut" />
  <CubicEase x:Key="EaseIn" EasingMode="EaseIn" />
</Window.Resources>

<Style TargetType="Button">
  <Setter Property="RenderTransformOrigin" Value="0.5,0.5" />
  <Setter Property="RenderTransform"><Setter.Value><ScaleTransform /></Setter.Value></Setter>
  <Style.Triggers>
    <Trigger Property="IsPressed" Value="True">
      <Trigger.EnterActions><BeginStoryboard><Storyboard>
        <DoubleAnimation Storyboard.TargetProperty="RenderTransform.ScaleX" To="0.95" Duration="0:0:0.08" />
        <DoubleAnimation Storyboard.TargetProperty="RenderTransform.ScaleY" To="0.95" Duration="0:0:0.08" />
      </Storyboard></BeginStoryboard></Trigger.EnterActions>
      <Trigger.ExitActions><BeginStoryboard><Storyboard>
        <DoubleAnimation Storyboard.TargetProperty="RenderTransform.ScaleX" To="1" Duration="0:0:0.22" EasingFunction="{StaticResource SoftEase}" />
        <DoubleAnimation Storyboard.TargetProperty="RenderTransform.ScaleY" To="1" Duration="0:0:0.22" EasingFunction="{StaticResource SoftEase}" />
      </Storyboard></BeginStoryboard></Trigger.ExitActions>
    </Trigger>
  </Style.Triggers>
</Style>
```

Use `ColorAnimation` (0.15 s) on `Background.Color` for color fades. Use `CompositionTarget.Rendering` as the frame loop for springs. For reduced motion, read `SystemParameters.ClientAreaAnimation`; `false` means animations are off.

## .NET MAUI

```csharp
await view.ScaleTo(0.95, 80, Easing.CubicOut);
await view.ScaleTo(1, 220, Easing.SpringOut);           // built-in overshoot
await Task.WhenAll(page.FadeTo(1, 280, Easing.CubicOut), page.TranslateTo(0, 0, 280, Easing.CubicOut)); // page enter from TranslationY = 10
// Custom curve: new Easing(t => 1 + 2.55 * Math.Pow(t - 1, 3) + 1.55 * Math.Pow(t - 1, 2))
// Spring loop: new Animation(callback, ...) or IDispatcherTimer at 16 ms stepping spring-engine's Spring
```

## WinUI 3

Implicit animations are built in: `ScaleTransition`, `BrushTransition` and `RepositionThemeTransition`. For springs, use Composition's `SpringVector3NaturalMotionAnimation` with `DampingRatio = 0.56f`. Tune `Period` by eye against the web demo; check the docs for exactly how it is defined, because it doesn't map 1:1 to stiffness. Read `UISettings.AnimationsEnabled` for reduced motion.
