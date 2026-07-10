---
name: frontend-flutter-baseline
description: "Screaming architecture and state lifecycle conventions for adding or reviewing Flutter features."
---

[← Back](../SKILL.md) <br>

---

# Annexes

- [dart-standards](../dart-standards/SKILL.md)
- [Flutter architecture recommendations](https://docs.flutter.dev/app-architecture/recommendations)
- For REST integration, use [frontend-flutter-rest-clients](../frontend-flutter-rest-clients/SKILL.md).
- For views and themes, use [frontend-flutter-ui-standards](../frontend-flutter-ui-standards/SKILL.md).

---

# Table of content

- [1. Screaming architecture](#1-screaming-architecture)
- [2. State and lifecycle](#2-state-and-lifecycle)
- [3. Configuration and verification](#3-configuration-and-verification)

---

# 1. Screaming architecture

Apply [Dart standards](../dart-standards/SKILL.md). Organize by functional responsibility first, then by frontend responsibility:

```text
lib:
├──app                         # Bootstrap, dependency composition, router, app shell
├──commons
│   ├──config
│   ├──errors                  # AppFailure and transport-independent failure types
│   ├──network                 # Client factory, interceptors, connectivity policies
│   ├──logging
│   ├──persistence             # Shared storage adapters
│   ├──theme                   # Tokens and component themes
│   ├──l10n
│   └──widgets                 # Reusable, domain-independent UI
└──<domain>
    ├──<subdomain.level.1>  
    │   └──<subdomain.level.2>     # Only when useful
    │       ├──dto                 # Handwritten external requests/responses, when needed
    │       ├──service             # Business rules and functional orchestration
    │       ├──mapper              # External representations ↔ functional values
    │       ├──repository          # REST, database, cache, files; grouped by responsibility
    │       └──presentation
    │           ├──controllers     # View models / state owners
    │           ├──pages
    │           └──widgets
    └───...
packages/<backend>_api          # Generated transport package; see REST skill
test                            # Mirrors lib/ by functional area
```

e.g.,

```
lib:
├──app
├───commons
│   └───...
│
└───personal.finances
  ├───auth
  │   ├───refresh
  │   │   └───...
  │   └───signin
  │       └───...
  │
  └───expenses
      ├───crud
      │   └───...
      └───categories
          └───...
```

1. Create only folders and layers with actual consumers. Existing `features/<area>` remains a valid functional grouping; prefer `commons` for new shared artifacts and migrate an existing `core` only within authorized refactoring.
2. The dependency flow is `widget → controller → service → repository → external source`. Controllers own presentation state; services own business rules and functional orchestration; repositories own external data access; mappers convert representations. Models and business rules remain independent of Flutter and transport/storage packages. Create a service only when it adds business value; simple operations may use a repository directly from the controller.
3. Keep business widgets and policies inside their owning area. Promote code to `commons` when it is cross-cutting or has real independent consumers. `commons` must not import functional areas.
4. Share public service/repository APIs or functional models across areas; avoid importing another area's controllers or private transport models. Keep dependency direction acyclic. Repository APIs express functional operations and values, independently of their underlying technology.
5. Keep generated clients/DTOs in a transport package and consume them from `<subdomain>/repository`. Keep handwritten integration code in that same repository layer and handwritten DTOs in `dto`; conversions belong in the sibling `mapper` folder. Services, controllers, and widgets must not call REST clients, database drivers, or file APIs directly.

# 2. State and lifecycle

1. Retain the established state-management solution. For Riverpod, use providers for dependency wiring and `Notifier`/`AsyncNotifier` for presentation state. Keep provider declarations beside their owning component or in app composition; providers are a framework mechanism, not an architectural layer. Keep services, repositories, and mappers independent of Riverpod.
2. Represent loading, data, empty, and failure states explicitly. Model refresh, pagination, and mutation progress separately; retain usable data when a later operation fails.
3. Keep `build` methods free of network calls, navigation side effects, and repeated initialization. Initialize asynchronous controllers through the state-management lifecycle.
4. Cancel requests on disposal or invalidate their results using request generations. A slow old search, page load, or session restoration must not overwrite newer filters, logout, or a different user session.
5. Serialize duplicate submissions and page loads. Treat cursors as opaque; reset them when filters change, deduplicate appended records by stable ID, and stop on absent or repeated cursors.
6. Dispose text/scroll controllers and listeners. After awaits, verify widget/controller liveness before updating state or navigating. Present one-time effects through listeners or consumed events.
7. Centralize route names and authentication redirects. Distinguish session initialization from logged out state; validate deep-link parameters and retain the intended destination when appropriate. Server authorization remains authoritative.
8. Scope user-specific providers and caches to the session; invalidate them on logout/account changes. Ignore late results from the previous session.

# 3. Configuration and verification

1. Externalize base URLs and public environment settings; validate them during bootstrap. Compile-time defines and bundled assets are not secret storage. Store credentials with a platform-appropriate secure storage adapter.
2. Own shared resources at composition scope; close HTTP clients and dispose containers when their owner ends. Handle uncaught errors through a common sanitized logging boundary.
3. Test controller state transitions, stale responses, pagination/filter races, disposal, and session changes using controlled futures and provider overrides. Keep HTTP and plugins out of widget tests.
4. Test loading/error/empty/data and mutation behavior with widgets; add integration tests for critical journeys such as login and create/update. Use `dart format`, `flutter analyze`, and relevant `flutter test` checks; build affected targets when platform configuration changes.
