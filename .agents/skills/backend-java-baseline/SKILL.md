---
name: backend-java-baseline
description: "Backend Java baseline"
---

[← Back](../SKILL.md) <br>

---

# Annexes

- [backend-baseline](../backend-baseline/SKILL.md)

---

# Table of content

- [1. Error handling](#1-error-handling)

---

# 1. Error handling

## 1.1. GenericException

```java
package io.github.miguelarmasabt.error.exceptions;

// imports 

@Getter
public class GenericException extends RuntimeException {

   protected ErrorDto error;
   protected Response.Status httpStatus;

   public GenericException(String errorCode) {
      super(errorCode);
      this.error = ErrorMapper.own(errorCode);
   }

   public GenericException(String errorCode, String... messageArguments) {
      super(errorCode);
      this.error = ErrorMapper.own(errorCode, messageArguments);
   }

   public GenericException(String errorCode, Response.Status httpStatus) {
      super(errorCode);
      this.httpStatus = httpStatus;
      this.error = ErrorMapper.own(errorCode);
   }

   public GenericException(String errorCode, Response.Status httpStatus, String... messageArguments) {
      super(errorCode);
      this.httpStatus = httpStatus;
      this.error = ErrorMapper.own(errorCode, messageArguments);
   }

   public GenericException(String errorCode, Response.Status httpStatus, Throwable exception) {
      super(errorCode, exception);
      this.httpStatus = httpStatus;
      this.error = ErrorMapper.own(errorCode);
   }

   public GenericException(String errorCode, Response.Status httpStatus, Throwable exception, String... messageArguments) {
      super(errorCode, exception);
      this.httpStatus = httpStatus;
      this.error = ErrorMapper.own(errorCode, messageArguments);
   }
}
```

## 1.2. ErrorDto

This class is responsible for representing the JSON response error.

```java
package io.github.miguelarmasabt.error.dto;

// imports

@ToString
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class ErrorDto implements Serializable {

   @JsonProperty("origin")
   private ErrorOrigin origin;

   private String code;

   private String message;

   @JsonIgnore
   private String[] messageArguments;
}
```

e.g.,

## 1.3. ErrorOrigin

This enum represents [error origin](./../backend-baseline/SKILL.md#22-error-origin).

```java
package io.github.miguelarmasabt.error.dto;

public enum ErrorOrigin {
  OWN,
  PARTNER,
  THIRD_PARTY
}
```

## 1.4. How to create an exception?

1. If the exception is cross to the project, save it in `commons.exceptions` package.
2. If the exception is specific to a domain, save it in the specific `exceptions` package.
3. All exceptions must extend of `GenericException` and define the "code" and "HTTP status" attributes.
4. Error code must follow the [error code rules](./../backend-baseline/SKILL.md#21-error-code).

e.g.,

```java
package io.github.miguelarmasabt.personal.finances.expenses.csv.exceptions;

import io.github.miguelarmasabt.error.exceptions.GenericException;

import static jakarta.ws.rs.core.Response.Status.BAD_REQUEST;

public class InvalidCsvRowException extends GenericException {

  public InvalidCsvRowException(long rowIndex, int headersLength) {
    super(
        "2000",
        BAD_REQUEST,
        String.valueOf(rowIndex),
        String.valueOf(headersLength)
    );
  }

  public InvalidCsvRowException(long rowIndex, String field) {
    super(
        "2001",
        BAD_REQUEST,
        String.valueOf(rowIndex),
        field
    );
  }
}
```

## 1.5. Resource bundles

Error descriptions must be defined in localized resource bundles:
1. `ErrorMessages_en.properties` for English descriptions.
2. `ErrorMessages_es.properties` for Spanish descriptions.
3. Both resource bundles must contain the same error codes.
4. Use MessageFormat placeholders (`{0}`, `{1}`, etc.) for dynamic values.

e.g.,

```properties
#commons
1000=The ObjectId {0} is invalid
1001=The date {0} should be in the format dd-MM-yyyy or be before the current date

#expenses.csv
2000=Row {0} of the provided CSV file does not contain {1} columns
2001=Row {0} of the provided CSV file does not contain the field {1}

#expenses.bank.receipts
2100=The strategy for extracting expenses is not supported: {0},{1}

#expenses.categories
2200=The category pattern {0} is not supported

#auth.signin
3000=The account {0} already exists. Sign in with your password and link your Google account from your profile
3001=The user {0} is disabled
3002=The Google email {0} is not verified

#auth.refresh
3100=The refresh token is invalid or expired
```
