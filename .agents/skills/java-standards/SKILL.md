---
name: java-standards
description: "Java programming standards"
---

[← Back](../SKILL.md) <br>

---

# Annexes

_(without annexes)_

---

# Table of content

- [1. Coding conventions](#1-coding-standards)
- [2. Unit testing](#2-unit-testing)

---

# 1. Coding standards

In addition to [general coding standards](./../coding-standards/SKILL.md#1-coding-standards):

1. Java 25 language
   - Records
   - Pattern Matching (e.g., `instanceof`, `switch`)
2. Java's functional programming features:
   - Optional
   - Streams and primitive streams
   - Functional interfaces (e.g., `Predicate`, `Function`, `Consumer`, `Supplier`, `UnaryOperator`, `BinaryOperator`, `BiPredicate`, `BiFunction`, `BiConsumer`, etc.)
   - Method references
   - Collectors
3. Avoid using `var` for object declarations.
4. Use `java.util.Objects` for basic `null` checks and comparisons. Use Optional for complex cases.
5. Take advantage of `Mapstruct` and `Lombok` features.
6. Shared properties between profiles must be in `application.yaml`.
7. Use `@RequiredArgsConstructor` to inject DI instead of CDI annotations.
8. Returning `null` explícitly is prohibited. Wrap the variable in an `Optional` or use a reactive operator.

---

# 2. Unit testing

1. Use the `given`-`when`-`then` naming convention for test method names.

```java
void givenValidUserId_WhenFindUser_ThenReturnUser() {...}
void givenInvalidUserId_WhenFindUser_ThenThrowNotFoundUserException() {...}
void whenGetCurrentTimestamp_ThenReturnCurrentTimestamp() {...}
```

2. Use `@DisplayName` to describe the test behavior consistently with the test method name.

```java
@Test
@DisplayName("Given a valid user ID, when finding the user, then return the user")
void givenValidUserId_WhenFindUser_ThenReturnUser() {...}
```
3. Separate the test method into `Arrange`, `Act`, and `Assert` sections using comments.
   - For simple tests, use only the section names.
   - For more complex tests, add a brief description after each section name.

```java
//Arrange

//Act

//Assert 
```

```java
// Arrange: Create a user with an expired subscription.

// Act: Renew the subscription.

// Assert: Verify the subscription is active and the expiration date is updated.
```

4. Use `@ParameterizedTest` with `@CsvSource` when the same test logic can be executed with multiple input cases.
   - Explicitly define the `delimiter` attribute to avoid conflicts with data values.

```java
@ParameterizedTest
@CsvSource(
    value = {
        "john@example.com | true",
        "invalid-email | false",
        "' ' | false"
    },
    delimiter = '|'
)
void givenEmail_WhenValidate_ThenReturnValidEmailAssertion(String email, boolean expected) {...}
```

5. Use the dependency injection container (e.g., Spring or CDI) only for integration tests, framework behavior or configuration validation.
6. Avoid using `@InjectMock`, `@MockBean`, or similar annotations when manual "dependency wiring" is sufficient.
7. Instantiate the class under test directly using its constructor.
8. Use real MapStruct mappers in unit tests by default. Instantiate them with `Mappers.getMapper(MyMapper.class)`
9. Mock only the direct dependencies of the class under test.
10. Centralize mock creation in a static `Mock` helper class to improve readability and reuse.
11. Centralize test fixtures in JSON files under `src/test/resources/fixtures` (e.g., `src/test/resources/fixtures/expenses/ExpenseCategoryResponseDto.json`).
12. Reference JSON fixtures using their full path to make them easier to locate (e.g., `JSON_SERIALIZER.readListFromFile("fixtures/financings/PaymentInstallmentResponseDto.json", PaymentInstallmentResponseDto.class)`). Avoid using of base path constants; each reference should be complete to IDE allow redirect to the file (ctrl + click).
13. Convert JSON test fixtures into java objects and set the attributes required by each test case.

```java
import io.github.miguelarmasabt.tools.JsonSerializer;

class FinancingServiceImplTest {

  private static final ObjectMapper objectMapper = new ObjectMapper();
  private static final JsonSerializer jsonSerializer = new JsonSerializer(objectMapper);

  @Test
  void givenOperationCode_WhenGetPaymentInstallments_ThenReturnPaymentInstallments() {
    // Arrange
    FinancingServiceImpl service = Mock.financingService();
    String operationCode = "00001";

    // Act
    List<PaymentInstallmentResponseDto> actual = service.getPaymentInstallments(operationCode);

    // Assert
    List<PaymentInstallmentResponseDto> expected =
            jsonSerializer.readListFromFile("fixtures/financings/PaymentInstallmentResponseDto.json", PaymentInstallmentResponseDto.class);
    expected.setOperationCode(operationCode);

    assertThat(actual)
            .usingRecursiveComparison()
            .ignoringFields("id", "createdAt", "updatedAt")
            .ignoringCollectionOrder()
            .isEqualTo(expected);
  }

  @NoArgsConstructor(access = AccessLevel.PRIVATE)
  static final class Mock {

    static FinancingServiceImpl financingService() {
      FinancingMapper financingMapper = Mappers.getMapper(FinancingMapper.class);
      PaymentInstallmentMapper paymentInstallmentMapper = Mappers.getMapper(PaymentInstallmentMapper.class);
      FinancingServiceImpl service = new FinancingServiceImpl(financingMapper, paymentInstallmentMapper);
      service.financingApi = financingApi();
      return service;
    }

    static FinancingApi financingApi() {
      FinancingApi api = Mockito.mock(FinancingApi.class);

      FinancingScheduleResponseWrapper response =
              jsonSerializer.readElementFromFile("fixtures/financings/FinancingScheduleResponseWrapper.json", FinancingScheduleResponseWrapper.class);

      Mockito.when(api.getFinancingSchedule(Mockito.anyString()))
              .thenReturn(response);

      return api;
    }
  }
}
```

```java
package io.github.miguelarmasabt.tools;

import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.type.CollectionType;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

public class JsonSerializer {

  private final ObjectMapper mapper;

  public JsonSerializer(ObjectMapper objectMapper) {
    this.mapper = objectMapper;
    this.mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, true);
  }

  protected static InputStream getResourcesAsStream(String filePath) {
    String normalizedPath = filePath.startsWith("/") ? filePath.substring(1) : filePath;
    return Thread.currentThread().getContextClassLoader().getResourceAsStream(normalizedPath);
  }

  protected <T> T readValue(InputStream inputStream, Class<T> objectClass) {
    try (InputStream is = inputStream) {
      return mapper.readValue(is, objectClass);
    } catch (IOException ex) {
      throw new IllegalArgumentException(ex);
    }
  }

  protected <T> List<T> readList(InputStream inputStream, Class<T> objectClass) {
    try (InputStream is = inputStream) {
      CollectionType collectionType = mapper.getTypeFactory().constructCollectionType(List.class, objectClass);
      return mapper.readValue(is, collectionType);
    } catch (IOException ex) {
      throw new IllegalArgumentException(ex);
    }
  }

  public <T> T readElementFromFile(String filePath, Class<T> objectClass) {
    return readValue(getResourcesAsStream(filePath), objectClass);
  }

  public <T> List<T> readListFromFile(String filePath, Class<T> objectClass) {
    return readList(getResourcesAsStream(filePath), objectClass);
  }
}
```