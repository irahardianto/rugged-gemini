---
name: mobile-design
description: >-
  Distinctive, production-grade mobile interfaces for Flutter/RN. Platform-native
  patterns, adaptive layouts, fluid motion. iOS/Android conventions.
---

# Mobile Design Skill

Distinctive, production-grade mobile interfaces. Native to platform while maintaining unique aesthetic.

## Design Thinking

Before coding:
- **Platform:** iOS (Cupertino), Android (Material 3), or cross-platform?
- **Tone:** luxury, playful, utilitarian, editorial, organic, bold, soft...
- **Constraints:** low-end device perf, offline, accessibility
- **Differentiation:** signature interaction

**CRITICAL:** Respect platform conventions (nav patterns, gestures, system UI) while injecting personality through typography, color, motion, layout.

## Guidelines

### Platform
- **iOS:** bottom tab bar, large title nav, swipe-back, Cupertino widgets
- **Android:** Material 3, nav rail/drawer, FAB, top app bar
- **Cross:** adaptive widgets rendering platform-appropriate UI

### Typography
- Readable at mobile sizes (16px+ body). Respect dynamic type (never hardcode sizes).
- `TextTheme` in Flutter. Characterful display + legible body.

### Color & Theme
- ALWAYS support light + dark. `ColorScheme`/`ThemeData`. OLED + LCD contrast.
- Color purposefully: primary actions, status, brand.

### Motion
- Meaningful (communicate state, not decorate). ≤300ms interactions, ≤500ms transitions.
- `Hero` for shared elements. Stagger list items. Respect `reduceMotion` — always static fallback.

### Layout
- One-handed: critical actions in bottom 60% (thumb zone). Safe area insets.
- Responsive: compact (<600dp), medium (600-840dp), expanded (>840dp).
- Touch targets: min 48×48dp.

### Performance
- `const` constructors. `ListView.builder` for lists. `cached_network_image`.
- Profile on real devices.

## Flutter Patterns

```dart
// ✅ Adaptive platform widget
Widget buildButton(BuildContext context) {
  return Platform.isIOS
    ? CupertinoButton(child: text, onPressed: onTap)
    : ElevatedButton(child: text, onPressed: onTap);
}

// ✅ Responsive breakpoints
Widget build(BuildContext context) {
  final width = MediaQuery.of(context).size.width;
  if (width >= 840) return _expandedLayout();
  if (width >= 600) return _mediumLayout();
  return _compactLayout();
}

// ❌ Hardcoded sizes
Container(width: 375, height: 812) // iPhone X only!
```

## Compliance
- Project Structure @.gemini/rules/project-structure.md (Flutter layout)
- Testing Strategy @.gemini/rules/testing-strategy.md (widget/integration tests)
- Security Principles @.gemini/rules/security-principles.md (secure storage)
- Accessibility — load `accessibility-principles` skill
- Architectural Patterns @.gemini/rules/architectural-pattern.md (Riverpod 3)

Mobile constraints: battery, network variability, one-handed use. Beautiful UI that drains battery or stutters = failure. Every pixel and interaction intentional.
