---
name: backend-baseline
description: "Backend baseline"
---

[← Back](../SKILL.md) <br>

---

# Annexes

_(without annexes)_

---

# Table of content

- [1. Screaming architecture](#1-screaming-architecture)
- [2. Error response structure](#2-error-response-structure)

---

# 1. Screaming architecture

```
io.github.miguelarmasbat:                     # e.g., Java class suffix
├───commons
│   ├───config
│   ├───constants
│   ├───enums | catalogs
│   ├───exceptions | errors                   # *Exception
│   ├───properties
│   ├───repository
│   └───utils
│
└───<domain>
  ├───<subdomain.level.1>
  │   └───<subdomain.level.2>
  │       ├───dto
  │       │   ├───request                   # *RequestDto
  │       │   └───response                  # *ResponseDto
  │       ├───enums | catalogs              # *Type | *Category | *Enum
  │       ├───exceptions|errors
  │       ├───mapper                        # *RequestMapper | *ResponseMapper
  │       ├───repository
  │       │   └───`<data-model>`            # *Repository
  │       │         └───entity | wrapper    # *Entity | *Document | *RequestWrapper | *ResponseWrapper
  │       ├───rest                          # *RestService
  │       └───service                       # *Service
  └───...
```

e.g.,

```
io.github.miguelarmasbat:
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
      ├───bank.receipts
      │   └───...
      └───categories
          └───...
```

# 2. Error response structure

```json
{
  "code": "1000",
  "origin": "OWN",
  "message": "Invalid field. traceParent must not be empty"
}
```

## 2.1. Error code

The error code must be an int of 4 digits and following the rules:

1. The first digit represents the top-level domain.
2. The second digit represents the second-level subdomain.
3. The first digit 1 is reserved for commons.
4. The last two digits represent a sequential identifier within the corresponding subdomain, starting at 00.
5. Error codes must be unique across the application.

## 2.2. Error origin

| Origin        | Description                                                               |
|---------------|---------------------------------------------------------------------------|
| `OWN`         | Own backend error.                                                        |
| `PARTNER`     | Error originates from another backend that uses the same error structure. |
| `THIRD_PARTY` | Error originates from another backend with a different error structure.   |

