---
name: dart-standards
description: "Coding and testing standards for writing or reviewing Dart code."
---

[← Back](../SKILL.md) <br>

---

# Annexes

- [coding-standards](../coding-standards/SKILL.md)
- [Effective Dart](https://dart.dev/effective-dart)

---

# Table of content

- [1. Coding standards](#1-coding-standards)
- [2. Data boundaries](#2-data-boundaries)
- [3. Unit testing](#3-unit-testing)

---

# 1. Coding standards

In addition to [general coding standards](../coding-standards/SKILL.md):

1. Respect the project's SDK constraint; use supported null safety, pattern matching, records, and sealed classes where they clarify the model. Do not upgrade the SDK as a side effect.
2. Use `lower_snake_case` for files, `UpperCamelCase` for types, and `lowerCamelCase` for members. Keep declarations inferred when the type is clear; specify public API and collection element types.
3. Prefer immutable objects, `final` fields, `const` constructors, and defensive immutable collection copies. Define value equality where state comparisons or collections depend on it.
4. Use nullable types for legitimate absence; reserve `!` and `late` for proven invariants. Do not emulate Java `Optional` or use sentinel strings for missing values.
5. Return `Future<T>` for asynchronous work; await it or explicitly use `unawaited` with an error owner. Handle expected typed exceptions; preserve stack traces with `rethrow` when propagating.
6. Dispose streams, subscriptions, timers, and other owned resources. Make cancellation distinct from an operation failure.
7. Inject collaborators through constructors. Keep domain types independent of Flutter, HTTP clients, persistence, and state-management packages.

# 2. Data boundaries

1. Confine `dynamic` and JSON maps to serialization boundaries; expose typed requests, responses, and domain values internally.
2. Treat malformed required fields as decoding failures. Never replace invalid identifiers, dates, amounts, or currencies with `''`, the current time, zero, or a different currency.
3. Distinguish missing fields, explicit `null`, and empty values when the contract gives them different meanings, especially for partial updates.
4. Preserve exact monetary values using integer minor units with explicit currency scale or a decimal representation. Do not calculate money through `double`; match the wire contract using explicit converters.
5. Distinguish date-only values from instants. Serialize instants with the backend's required timezone and precision; localize only for display. Do not convert date-only values through UTC.
6. Map transport DTOs to domain values outside widgets. Generate serialization for repetitive DTOs when useful; do not hand-edit generated files.

# 3. Unit testing

1. Use `test`/`group` descriptions in given/when/then language. Instantiate subjects directly and fake only their external dependencies.
2. Keep reusable transport fixtures under `test/fixtures/<functional-area>/`; remove personal data and credentials.
3. Test meaningful boundaries: missing required fields, unknown enum values, null updates, exact decimal round trips, timezone boundaries, and expected exceptions. Inject time and identifiers when behavior depends on them.
4. Format and analyze changed code using the project's SDK. Run focused tests for changed behavior; run the project's required checks before completion.
