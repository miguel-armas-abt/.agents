---
name: backend-quarkus-baseline
description: "Backend Quarkus baseline"
---

[← Back](../SKILL.md) <br>

---

# Annexes

- [backend-baseline](./../backend-baseline/SKILL.md)
- [backend-java-baseline](./../backend-java-baseline/SKILL.md)
- [java-standards](./../java-standards/SKILL.md)

---

# Table of content

- [1. Coding standards](#1-coding-standards)
- [2. Parent](#2-parent)
- [3. Contract first - Server](#3-contract-first---server)
- [4. Contract first - Client](#4-contract-first---client)


---

# 1. Coding standards

In addition to [java coding standards](./../java-standards/SKILL.md#1-coding-standards):

1. The use of `jakarta.ws.rs.core.HttpHeaders` or similar is prohibited. All headers must reach the backend via method parameters in the REST service layer.
2. The use of `lombok.extern.slf4j.Slf4j` is prohibited; use `org.jboss.logging.Logger` with a `log` variable that is `private`, `static` and `final`.

---

# 2. Parent

All Quarkus backend applications must be created from a common parent artifact.

```xml
<parent>
    <groupId>io.github.miguelarmasabt</groupId>
    <artifactId>commons-quarkus-parent</artifactId>
    <version>1.0.0</version>
</parent>

<dependencies>
   <!-- rest of dependencies -->
   <dependency>
       <groupId>io.github.miguelarmasabt</groupId>
       <artifactId>commons-quarkus</artifactId>
       <version>1.0.0</version>
   </dependency>
</dependencies>
```

# 3. Contract first - Server

> **IMPORTANT**: Only applies if the project is contract first server.

1. OpenAPI definition must be localized in `src/main/resources/META-INF/openapi.yaml`.
2. Use `io.quarkiverse.openapi.generator/quarkus-openapi-generator-server` dependency.
3. Use the following structure for `quarkus.openapi.generator.server`:

```yaml
quarkus:
  openapi:
    generator:
      server:
        use: apicurio
        spec: openapi.yaml
        input-base-dir: src/main/resources/META-INF
        base-package: ${info.main-package}.rest.server
        use-reactive: false
        use-bean-validation: true
        skip-if-unchanged: false
```

4. Methods in rest service layer must return reactive types. e.g., `CompletionStage<>`, `RestMulti`, so on. 
   - If the operation returns a DTO directly or doesn't return any response body, include `x-codegen-async: true` int the OpenAPI operation.
   - If the operation must return `jakarta.ws.rs.core.Response`, also include `content.application/json.x-codegen-returnType: jakarta.ws.rs.core.Response`.
   - If the operation must return files, include for example `content.text/csv.x-codegen-returnType: org.jboss.resteasy.reactive.RestMulti<io.vertx.core.buffer.Buffer>`

# 4. Contract first - Client

> **IMPORTANT**: Only applies if the project is contract first client.

1. REST client OpenAPI definition must be localized in `src/main/openapi/*.yaml`.
2. REST clients must be registered under property `quarkus.openapi-generator.codegen.spec`.
3. The `base-package` property must follow the pattern `base-package: ${info.commons-package}.repository.*`
4. The `openApiSpecId` value in `application.yml` must be match with the name of its OpenAPI file, replacing all hyphens and dots with a hyphen. E.g:

```yaml
quarkus:
  openapi-generator:
    codegen:
      spec:
        gmail_yaml:
          base-package: ${info.commons-package}.repository.google
          config-key: gmail
          api-name-suffix: Repository
          mutiny: true
          serializable-model: true
          use-bean-validation: false
          enable-security-generation: false
          additional-api-type-annotations: '${info.rest-client.register-client-headers}(${info.commons-package}.repository.google.headers.GmailHeadersFactory.class)'

```

5. REST clients must be defined in `application-local.yml` under property `quarkus.rest-client`. e.g.,

```yaml
quarkus:
  rest-client:
    exchange-rate:
      url: ${EXCHANGE_RATE_ENDPOINT}
      verify-host: false
      connect-timeout: 1500
      read-timeout: 13000
      headers:
        Subscription-Key: ${EXCHANGE_RATE_SUBSCRIPTION_KEY}
```

6. Headers that are non-static must be propagated using the `org.eclipse.microprofile.rest.client.propagateHeaders` property.


