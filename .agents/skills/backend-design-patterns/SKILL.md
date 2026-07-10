---
name: backend-design-patterns
description: "Backend design patterns"
---

[← Back](../SKILL.md) <br>

---

# Annexes

_(without annexes)_

---

# Table of content

- [1. Strategy pattern](#1-strategy-pattern)

---

# 1. Strategy pattern

## 1.1. Java

The following examples use Spring Boot, but the same approach can be applied to other Java frameworks that support dependency injection.

### 1.1.1. Using a list

> Use a list when the number of strategy implementations is relatively small and strategy selection is based on a condition.

1. Create an interface for the strategy and its implementations.

```java
public interface EncryptionStrategy {
    String encrypt(String value, CryptographyProperties properties);
    String decrypt(String cipherMessage, CryptographyProperties properties);

    boolean supports(EncryptionMethod encryptionMethod); // condition
}
```

2. Create a selector to resolve the appropriate strategy.

```java
@Component
@RequiredArgsConstructor
public class EncryptionSelector {
  private final List<EncryptionStrategy> strategies;
  
  public EncryptionStrategy selectStrategy(EncryptionMethod encryptionMethod) {
    return strategies
        .stream()
        .filter(strategy -> strategy.supports(encryptionMethod))
        .findFirst()
        .orElseThrow(NoSuchEncryptionStrategyException::new);
  }
}
```

3. Use the selector within your business logic.

```java
@Component
@RequiredArgsConstructor
public class EncryptionServiceImpl implements EncryptionService {

    private final EncryptionSelector encryptionSelector;
    private final ApplicationProperties properties;

    @Override
    public EncryptionResponseDto encrypt(EncryptionMethod encryptionMethod, String feature, String valueToEncrypt) {
        String cipherMessage = encryptionSelector
            .selectStrategy(encryptionMethod)
            .encrypt(valueToEncrypt, searchFeature(feature));
        
        return EncryptionResponseDto.builder()
            .cipherMessage(cipherMessage)
            .build();
    }

    @Override
    public DecryptionResponseDto decrypt(EncryptionMethod encryptionMethod, String feature, String cipherMessage) {
        String value = encryptionSelector
            .selectStrategy(encryptionMethod)
            .decrypt(cipherMessage, searchFeature(feature));

        return DecryptionResponseDto
            .builder()
            .value(value)
            .build();
    }

    private CryptographyProperties searchFeature(String feature) {
        return Optional.ofNullable(properties.getCryptography().get(feature.toLowerCase(Locale.ROOT)))
            .orElseThrow(NoSuchCryptographyPropertiesException::new);
    }
}
```

### 1.1.2. Using a map

> Use a map when you have multiple strategy implementations and need to resolve them directly using a key, avoiding a sequential search through the collection.

1. Create an interface for the strategy and its implementations.

```java
public interface EncryptionStrategy {
    String encrypt(String value, CryptographyProperties properties);
    String decrypt(String cipherMessage, CryptographyProperties properties);

  EncryptionMethod getEncryptionMethod(); // key
}
```

2. Create a selector that builds a map of strategies.

```java
@Component
public class EncryptionSelector {

    private final Map<EncryptionMethod, EncryptionStrategy> strategies;

    public EncryptionSelector(List<EncryptionStrategy> strategies) {
        this.strategies = strategies.stream()
            .collect(Collectors.toUnmodifiableMap(
                EncryptionStrategy::getEncryptionMethod,
                Function.identity()
            ));
    }

    public EncryptionStrategy selectStrategy(EncryptionMethod encryptionMethod) {
        return Optional.ofNullable(strategies.get(encryptionMethod))
            .orElseThrow(NoSuchEncryptionStrategyException::new);
    }
}
```


