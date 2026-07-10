---
name: frontend-flutter-rest-clients
description: "Repository-based REST integration, reproducible OpenAPI clients, transport logging, failures, and session handling for Flutter applications."
---

[← Back](../SKILL.md) <br>

---

# Annexes

- [frontend-flutter-baseline](../frontend-flutter-baseline/SKILL.md)
- [Dio documentation](https://pub.dev/packages/dio)
- [OpenAPI Generator dart-dio](https://openapi-generator.tech/docs/generators/dart-dio/)

---

# Table of content

- [1. Contract and client generation](#1-contract-and-client-generation)
- [2. API interoperability](#2-api-interoperability)
- [3. Shared transport and logging](#3-shared-transport-and-logging)
- [4. Failures and authentication](#4-failures-and-authentication)
- [5. Verification](#5-verification)

---

# 1. Contract and client generation

Apply [Flutter baseline](../frontend-flutter-baseline/SKILL.md).

1. Inventory consumed endpoints against the app-facing backend OpenAPI contract. Generate clients and DTOs for covered operations; do not handwrite equivalent HTTP calls. Contracts for a backend's own dependencies do not automatically describe APIs the app consumes.
2. Store an approved contract snapshot under `contracts/<backend>/openapi.yaml`, with its upstream revision or checksum and all referenced files. Do not depend on a developer-specific absolute backend path in CI.
3. Pin an exact OpenAPI Generator CLI version using the project tool wrapper, locked build tool, or container digest. Confirm compatibility with the contract's OpenAPI version, schema constructs, and installed Dart/Dio versions through a generation smoke check.
4. Keep generator configuration alongside the snapshot. Example options (confirm support using `config-help -g dart-dio` for the pinned version):

```yaml
pubName: backend_api
serializationLibrary: json_serializable
enumUnknownDefaultCase: true
disallowAdditionalPropertiesIfNotPresent: false
```

An unknown-enum fallback still requires explicit domain policy; never silently choose a known business value.

5. Keep a reproducible pipeline. Example, run from the app repository root with the pinned CLI:

```text
openapi-generator-cli validate -i contracts/<backend>/openapi.yaml
openapi-generator-cli generate -g dart-dio -i contracts/<backend>/openapi.yaml -c contracts/<backend>/generator.yaml -o packages/backend_api
```

Then run in `packages/backend_api` when required by the generated package:

```text
dart pub get
dart run build_runner build --delete-conflicting-outputs
dart analyze
```

6. Keep output exclusively generated. Choose checked-in output with a regeneration/diff CI check, or ignored output regenerated before app analysis/build. Track the app lockfile and resolution inputs consistently with that policy. The package `pubspec.yaml` is generated; manage dependencies through supported options/templates and add its path dependency in the app's `pubspec.yaml`.
7. On regeneration, replace only the verified generator-owned directory so removed operations do not leave stale code. Never delete neighboring handwritten adapters.
8. Inject the configured Dio into the generated API root using the constructor emitted by the pinned template; inspect that constructor instead of guessing its signature. Pass shared required header arguments through repository integration when generated signatures require them.
9. For operations absent from the contract, implement typed handwritten calls in the owning `<subdomain>/repository`, with DTOs in its sibling `dto` folder, using the same configured Dio. Verify method, path, payload, headers, and statuses against backend evidence; do not invent contracts. Replace manual integration when the operation becomes available in OpenAPI.
10. Consume generated and manual REST clients exclusively from repositories. Repositories convert DTOs through sibling `mapper` components and expose functional APIs. Services call repositories; controllers own presentation state. Keep one technology-agnostic `repository` folder, without `data/repositories` or `rest`/`local`/`files` subdivisions.
11. If generation fails on a schema feature, resolve upstream when authorized, choose a compatible pinned generator, or maintain a reviewed deterministic overlay with a documented difference. Keep isolated manual fallbacks only for demonstrated unsupported operations. Do not disable validation or silently weaken types.

# 2. API interoperability

Use the target API contract and authorized backend evidence. Record decisions in consuming project configuration or tests; do not transfer assumptions from another application.

| Contract concern | Repository integration rule |
|---|---|
| Headers and security | Match exact header names and security requirements. Supply required generated method arguments through repository integration, even when interceptors enrich the wire request. |
| Correlation | Configure the contract's header and identifier format. Validate the actual scheme; W3C trace context is only a default when declared or agreed. |
| Dates and times | Distinguish date-only values from instants; follow declared timezone and precision. |
| Numeric values | Preserve declared representation and precision. Monetary values need an exact domain representation and explicit converters. |
| Required, nullable, unknown | Verify schema semantics. Reject malformed required data and handle unknown values explicitly. |
| Pagination | Follow cursor, offset, or page semantics. Preserve opaque tokens and verify continuity and stopping conditions; do not repair server token inconsistencies in the client. |
| Success variants | Interpret documented statuses and bodies, including no-content responses, before returning domain results. |
| Files | Follow declared multipart names/media types; use bytes or streams for downloads. Keep access in repositories, delegate platform save/share to internal adapters, and sanitize external filenames. |
| Errors | Map actual envelope/status to common failures. Preserve available diagnostics separately from localized messages; do not require a universal error schema. |
| Session renewal | Implement only when supported by the authentication contract. Verify expiry, replay, persistence, and revocation semantics before automating recovery. |
| Multiple sources | Repository owns cache/network precedence, freshness, and fallback behavior. Distinguish stale from fresh data; never turn network failure into empty success. |

HTTP clients, DTOs, database models, file handles, and provider-specific exceptions stay behind the repository boundary. Representation conversions belong in sibling `mapper` folders. Generated source stays generator-owned; handwritten integration belongs in `<subdomain>/repository`, without technology-specific subdivisions or a surrounding `data` layer.

# 3. Shared transport and logging

1. Construct one configured Dio per backend/security context in `commons/network`; inject it into generated and manual clients. Centralize base URL, finite timeouts, authentication, correlation, and failure translation. Never send app credentials to third-party hosts.
2. Support cancellation through adapters. Retry only bounded transient failures for operations known to be safe; respect `Retry-After` where applicable. Mutations require a backend-supported idempotency mechanism before automatic replay.
3. Provide a common REST logger for successes, HTTP errors, timeouts, and cancellation. Record method, application-defined operation label, status, monotonic duration, and validated correlation ID. Install it after request-enriching interceptors. Define whether logs represent attempts or a full logical call when retries exist.
4. Implement the logger in `commons/network` and inject the common log sink. Configure its trace header from the contract. Attach static operation labels via `RequestOptions.extra`; never use a URL containing user data as the label.
5. Never log authorization, cookies, credentials, raw URLs, query values, sensitive payloads, files, or raw exceptions. Body logging requires explicit debug-only opt-in, an allowlist, redaction before truncation, and length/depth limits. The default logger omits bodies.
6. Logger failures must not affect networking. Forward each interceptor handler exactly once; do not consume streams, read multipart contents, mutate payloads, or bypass TLS verification.

# 4. Failures and authentication

1. Map transport errors once at the repository boundary into typed common failures: cancellation, timeout, network, unauthorized, forbidden, validation, not found, conflict, rate limit, server, and decoding. Preserve HTTP status and available backend identifiers/correlation as diagnostic metadata; do not assume a particular envelope.
2. Parse known error envelopes defensively and tolerate non-JSON proxy responses. Show localized safe messages by category/code; never render raw backend diagnostics or turn malformed successful responses into empty success values.
3. Read the current token at request time. Coordinate refresh as one shared in-flight operation, avoid intercepting refresh recursively, and bound replays to one. Replay a mutation only when backend semantics guarantee the rejected attempt was not applied, or use supported idempotency keys.
4. Distinguish expired/revoked credentials from transient refresh failure. Do not erase a recoverable session because the device is offline. Prevent refresh results from restoring a session after logout.
5. Persist rotated token pairs consistently; if partial persistence fails, do not expose a mixed session. A `403` does not automatically mean the token needs refresh.

# 5. Verification

1. Keep a reproducible generation command and verify it in CI. Regenerate twice from identical inputs and compare generator-owned output, excluding tool caches. Analyze both the generated package and app.
2. Test repository behavior with an in-memory HTTP adapter and sanitized fixtures: serialized headers/query/body, generated DTO decoding, domain/failure mapping, `204`, invalid bodies, cancellation, and binary/multipart paths used by the feature. Test cache/database/file coordination where repositories combine sources.
3. Test concurrent `401` responses, bounded refresh/replay, offline restoration, and logout during refresh when implementing renewal.
4. Verify logs omit secrets from query, headers, payloads, and exception text, and that a throwing log sink still forwards success and error handlers.
5. Exercise covered operations against contract fixtures, including nullable/required fields and consumed response variants. Do not claim the integration works solely because generated files exist.
