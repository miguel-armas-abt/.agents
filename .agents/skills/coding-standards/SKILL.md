---
name: coding-standards
description: "General coding standards"
---

[← Back](../SKILL.md) <br>

---

# Annexes

_(without annexes)_

---

# Table of content

- [1. Coding standards](#1-coding-standards)

---

# 1. Coding standards

1. All variables, parameters, methods and components must have meaningful and descriptive names.
2. Type-revealing suffixes in variable and parameter names must be avoided when the declared type already provides that information. e.g., clientDto, requestWrapper, userEntity.
3. If a component has a single dependency of given type (service, repository, mapper, so on), the attribute should be named unambiguously according to its role (service, repository, mapper), unless there is ambiguity.
4. Mandatorily delete components that lack functional or architectural value (e.g., orphan components, lacking content, without consumers, or legacy remnants). All component must have valid and verifiable responsibility.
5. Including comments is prohibited, unless they serve to clarify complex business logic.
6. SOLID principles should guide component design and dependency management.
7. DRY must be applied to avoid duplicated business rules and knowledge. However, abstractions must not be introduced solely to eliminate superficial code duplication.
8. Externalized configuration should be used for values that need to vary across environments.
