---
name: frontend-flutter-ui-standards
description: "Theming, responsive layout, and accessibility standards for building or reviewing Flutter views."
---

[← Back](../SKILL.md) <br>

---

# Annexes

- [frontend-flutter-baseline](../frontend-flutter-baseline/SKILL.md)
- [Flutter themes](https://docs.flutter.dev/cookbook/design/themes)
- [Adaptive design](https://docs.flutter.dev/ui/adaptive-responsive/best-practices)
- [Accessibility](https://docs.flutter.dev/ui/accessibility)

---

# Table of content

- [1. Design system](#1-design-system)
- [2. Views and interaction](#2-views-and-interaction)
- [3. Verification](#3-verification)

---

# 1. Design system

Apply [Flutter baseline](../frontend-flutter-baseline/SKILL.md).

1. Retain the project's visual language. For a new app without a selected system, use Flutter Material 3 as a practical default; use Cupertino/adaptive components when native iOS behavior is a product requirement. Treat brand, visual style, and component provider as separate choices.
2. Centralize colors, typography, spacing, radii, elevation, and motion in `commons/theme`. Use `ThemeData`, semantic `ColorScheme` roles, `TextTheme`, and component themes; use `ThemeExtension` for custom tokens. Avoid per-screen palettes and duplicated literal styles.
3. Start with a small spacing scale such as 4/8/12/16/24/32 logical pixels; choose a consistent density and readable content width. Express reusable business components within their feature and domain-independent components in `commons/widgets`.
4. Derive light/dark themes from shared semantic roles. Verify on-surface contrast and disabled/error states; do not encode status using color alone.
5. Select third-party component systems only for an identified gap. Check SDK/platform support, accessibility, theming, maintenance, license, and dependency cost. Record the choice; do not combine unrelated systems screen by screen.

# 2. Views and interaction

1. Use constraints (`LayoutBuilder`, `MediaQuery.sizeOf`) to choose layouts; define breakpoints where content stops fitting, rather than by device names. Respect safe areas, keyboard insets, orientation changes, text scaling, and maximum content widths.
2. Prefer standard accessible controls and small composable widgets. Use lazy list/grid builders for long collections; keep stable item keys. Avoid nested unconstrained scrollables and expensive parsing or I/O in `build`.
3. Define loading, empty, failure with recovery, and populated views. Show local progress for pagination and mutations; disable duplicate actions and preserve form input when a request fails.
4. Use `Form` validation, appropriate input types, focus traversal, and submit actions. Validate for user feedback while treating backend validation as authoritative. Show field-specific errors when the contract supports them; make destructive actions confirmable or undoable according to impact.
5. Localize visible strings through Flutter localization resources; format currency, numbers, and dates for the selected locale. Keep wire-format parsing separate from localized input/display. Do not assume all currencies have two fractional digits.
6. Provide semantic labels/tooltips for icon actions, meaningful reading order, keyboard focus, and screen-reader announcements where needed. Aim for at least 48×48 logical-pixel touch targets; preserve usability with large text and reduced motion.
7. Use selectors or appropriately scoped consumers to limit rebuilds when useful. Measure slow screens with DevTools before adding caching or rendering optimizations.

# 3. Verification

1. Widget-test interactions and all meaningful view states, including validation errors, retained input, and disabled duplicate submissions.
2. Check compact and wide layouts, light/dark themes, long localized text, large text scaling, keyboard-visible forms, and accessibility guidelines. Avoid asserting incidental widget nesting.
3. Use golden tests for stable reusable visual components when they add value; use device/emulator checks for scrolling, keyboard, screen readers, and platform behavior that widget tests cannot establish.
