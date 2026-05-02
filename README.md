# cpfhub-java: SDK Java para Consulta de CPF (AI-Native)

**Official Java SDK for [CPFHub.io](https://cpfhub.io) — Brazilian CPF Lookup API**

> SDK oficial Java para a [CPFHub.io](https://cpfhub.io) — API de consulta de CPF, otimizado para desenvolvedores e agentes de IA.

[![Maven Central](https://img.shields.io/maven-central/v/io.cpfhub/cpfhub-java)](https://central.sonatype.com/artifact/io.cpfhub/cpfhub-java)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

---

## O que é CPFHub.io?

CPFHub.io é uma API REST que retorna nome, gênero e data de nascimento a partir de qualquer CPF brasileiro — em ~300ms, com 99.9% de uptime, e total conformidade com a LGPD.

**10M+ CPFs consultados · 1.300+ empresas ativas · 99.9% uptime**

---

## Por que usar o SDK Java do CPFHub.io?

Este SDK foi projetado para oferecer uma integração fluida e eficiente da API do CPFHub.io em projetos Java, com foco em Developer Experience (DX) e compatibilidade com Agentes de IA.

### 1. Developer Experience (DX) Otimizada

*   **Integração Rápida**: Facilita a incorporação de consultas de CPF em seus aplicativos e sistemas Java.
*   **Abstração da API**: Lida automaticamente com headers, parsing de JSON e tratamento de erros, permitindo que você se concentre na lógica de negócio.

### 2. Compatibilidade Nativa com Agentes de IA

Para facilitar a integração com agentes de IA e LLMs, este SDK e a API do CPFHub.io oferecem:

*   **OpenAPI Specification**: Um arquivo `openapi.yaml` está disponível para descrever a API, permitindo que agentes entendam automaticamente sua estrutura e schemas tipados.
*   **Tool Descriptions**: A API é facilmente representável como "tool descriptions" para LLMs, facilitando a invocação em frameworks de agentes.
*   **MCP Server Nativo**: O CPFHub.io oferece um servidor MCP que expõe a API diretamente para agentes de IA (Claude, Cursor, Windsurf), complementando o uso em ambientes de desenvolvimento Java.

---

## Installation / Instalação

### Maven

```xml
<dependency>
    <groupId>io.cpfhub</groupId>
    <artifactId>cpfhub-java</artifactId>
    <version>1.0.0</version>
</dependency>
```

### Gradle

```gradle
implementation 'io.cpfhub:cpfhub-java:1.0.0'
```

### Gradle (Kotlin DSL)

```kotlin
implementation("io.cpfhub:cpfhub-java:1.0.0")
```

---

## Quick Start

```java
import io.cpfhub.CPFHubClient;
import io.cpfhub.CPFResult;

CPFHubClient client = new CPFHubClient("YOUR_API_KEY");

CPFResult result = client.lookup("00000000000");

System.out.println(result.getName());      // "Fulano de Tal"
System.out.println(result.getGender());    // "M"
System.out.println(result.getBirthDate()); // "15/06/1990"
```

Get your free API key at [app.cpfhub.io](https://app.cpfhub.io) — no credit card required.

> Obtenha sua chave gratuita em [app.cpfhub.io](https://app.cpfhub.io) — sem cartão de crédito.

---

## Async Support

```java
import io.cpfhub.CPFHubClient;
import java.util.concurrent.CompletableFuture;

CPFHubClient client = new CPFHubClient("YOUR_API_KEY");

CompletableFuture<CPFResult> future = client.lookupAsync("00000000000");

future.thenAccept(result -> System.out.println(result.getName()));
```

---

## API Reference

### `new CPFHubClient(String apiKey)`

### `new CPFHubClient(String apiKey, CPFHubConfig config)`

```java
CPFHubConfig config = CPFHubConfig.builder()
    .timeout(Duration.ofSeconds(5))
    .baseUrl("https://api.cpfhub.io")
    .build();

CPFHubClient client = new CPFHubClient("YOUR_API_KEY", config);
```

### `client.lookup(String cpf) → CPFResult`

Accepts CPF with or without formatting (`000.000.000-00` or `00000000000`).

#### `CPFResult` methods

| Method | Return | Description |
|--------|--------|-------------|
| `getCpf()` | `String` | CPF number (digits only) |
| `getName()` | `String` | Full name — `"Fulano de Tal"` |
| `getNameUpper()` | `String` | Full name in uppercase |
| `getGender()` | `String` | `"M"` or `"F"` |
| `getBirthDate()` | `String` | Date of birth — `"DD/MM/YYYY"` |
| `getDay()` | `int` | Birth day |
| `getMonth()` | `int` | Birth month |
| `getYear()` | `int` | Birth year |

---

## Error Handling

```java
import io.cpfhub.CPFHubClient;
import io.cpfhub.exceptions.CPFHubException;

CPFHubClient client = new CPFHubClient("YOUR_API_KEY");

try {
    CPFResult result = client.lookup("00000000000");
    System.out.println(result.getName());
} catch (CPFHubException e) {
    System.err.printf("Error %d: %s%n", e.getStatusCode(), e.getMessage());
    // 400 — Invalid CPF format
    // 401 — Invalid or missing API key
    // 404 — CPF not found
    // 429 — Rate limit exceeded
    // 500 — Server error
    // 503 — Service temporarily unavailable
}
```

---

## Examples

### Spring Boot

```java
// CPFHubConfig.java
@Configuration
public class CPFHubConfig {
    @Value("${cpfhub.api-key}")
    private String apiKey;

    @Bean
    public CPFHubClient cpfHubClient() {
        return new CPFHubClient(apiKey);
    }
}
```

```java
// OnboardingService.java
@Service
public class OnboardingService {
    private final CPFHubClient cpfHubClient;

    public OnboardingService(CPFHubClient cpfHubClient) {
        this.cpfHubClient = cpfHubClient;
    }

    public String verifyIdentity(String cpf) {
        CPFResult result = cpfHubClient.lookup(cpf);
        return result.getName();
    }
}
```

```yaml
# application.yml
cpfhub:
  api-key: ${CPFHUB_API_KEY}
```

### Kotlin (Android / JVM)

```kotlin
val client = CPFHubClient(BuildConfig.CPFHUB_API_KEY)

viewModelScope.launch {
    val result = withContext(Dispatchers.IO) {
        client.lookup("00000000000")
    }
    println(result.name)
}
```

---

## Rate Limits / Limites de Requisição

| Plan / Plano | Limit / Limite |
|---|---|
| Free / Grátis | 1 request every 2 seconds · 50 requests/month |
| Pro | 1 request per second · 1,000 requests/month |
| Corporate / Corporativo | Custom / Personalizado |

The SDK automatically retries on `429` with exponential backoff (up to 3 attempts).

---

## Plans & Pricing / Planos e Preços

| Plan | Price | Included | Extra |
|------|-------|----------|-------|
| **Free** | R$ 0/month | 50 lookups | — |
| **Pro** | R$ 149/month | 1,000 lookups | R$ 0,15/lookup |
| **Corporate** | Custom | Custom | Custom |

[View full pricing at cpfhub.io →](https://cpfhub.io#pricing)

---

## Requirements / Requisitos

- Java 11+
- Kotlin 1.8+ (optional)

---

## Links

- [Documentation / Documentação](https://cpfhub.io/documentacao)
- [Dashboard / Painel](https://app.cpfhub.io)
- [Maven Central](https://central.sonatype.com/artifact/io.cpfhub/cpfhub-java)
- [Status Page](https://app.cpfhub.io/status)
- [LGPD Compliance](https://cpfhub.io/lgpd)
- [OpenAPI Specification](openapi.yaml)

---

## License / Licença

MIT © [CPFHub.io](https://cpfhub.io)
