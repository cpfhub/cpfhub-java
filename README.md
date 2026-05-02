# cpfhub-java: Java SDK for CPFHub.io

🇺🇸 **English** | [🇧🇷 Português](#português)

**Official Java SDK for [CPFHub.io](https://cpfhub.io) — Brazilian CPF Lookup API**

[![Maven Central](https://img.shields.io/maven-central/v/io.cpfhub/cpfhub-java)](https://central.sonatype.com/artifact/io.cpfhub/cpfhub-java)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

---

## What is CPFHub.io?

CPFHub.io is a REST API that returns name, gender, and date of birth from any Brazilian CPF number — in ~300ms, with 99.9% uptime and full LGPD compliance.

**10M+ CPFs queried · 1,300+ active companies · 99.9% uptime**

---

## Installation

**Maven:**

```xml
<dependency>
  <groupId>io.cpfhub</groupId>
  <artifactId>cpfhub-java</artifactId>
  <version>1.0.0</version>
</dependency>
```

**Gradle:**

```groovy
implementation 'io.cpfhub:cpfhub-java:1.0.0'
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

---

## curl Example

```bash
curl -X GET "https://api.cpfhub.io/cpf/12345678909" \
  -H "x-api-key: YOUR_API_KEY"
```

**Response:**

```json
{
  "success": true,
  "data": {
    "cpf": "12345678909",
    "name": "Fulano de Tal",
    "nameUpper": "FULANO DE TAL",
    "gender": "M",
    "birthDate": "15/06/1990",
    "day": 15,
    "month": 6,
    "year": 1990
  }
}
```

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
import java.time.Duration;
import io.cpfhub.CPFHubConfig;

CPFHubConfig config = CPFHubConfig.builder()
    .timeout(Duration.ofSeconds(5))
    .baseUrl("https://api.cpfhub.io")
    .build();

CPFHubClient client = new CPFHubClient("YOUR_API_KEY", config);
```

### `client.lookup(String cpf) → CPFResult`

Looks up a CPF and returns the associated identity data.

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
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import io.cpfhub.CPFHubClient;

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
import org.springframework.stereotype.Service;
import io.cpfhub.CPFHubClient;
import io.cpfhub.CPFResult;

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
import io.cpfhub.CPFHubClient
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch
import kotlinx.coroutines.withContext

val client = CPFHubClient(BuildConfig.CPFHUB_API_KEY)

viewModelScope.launch {
    val result = withContext(Dispatchers.IO) {
        client.lookup("00000000000")
    }
    println(result.name)
}
```

---

## Rate Limits

| Plan | Limit |
|---|---|
| Free | 1 request every 2 seconds · 50 requests/month |
| Pro | 1 request per second · 1,000 requests/month |
| Corporate | Custom |

The SDK automatically retries on `429` with exponential backoff (up to 3 attempts).

---

## Plans & Pricing

| Plan | Price | Included | Extra |
|------|-------|----------|-------|
| **Free** | R$ 0/month | 50 lookups | — |
| **Pro** | R$ 149/month | 1,000 lookups | R$ 0,15/lookup |
| **Corporate** | Custom | Custom | Custom |

[View full pricing at cpfhub.io →](https://cpfhub.io#pricing)

---

## Requirements

- Java 11+
- Kotlin 1.8+ (optional)

---

## Links

- [Documentation](https://cpfhub.io/documentacao)
- [Dashboard](https://app.cpfhub.io)
- [Maven Central](https://central.sonatype.com/artifact/io.cpfhub/cpfhub-java)
- [Status Page](https://app.cpfhub.io/status)
- [Pricing](https://cpfhub.io#pricing)
- [LGPD Compliance](https://cpfhub.io/lgpd)
- [OpenAPI Specification](https://github.com/cpfhub/cpfhub-openapi/blob/main/openapi.yaml)
- [MCP Server (AI Agents)](https://github.com/cpfhub/cpfhub-mcp)

---

## License

MIT © [CPFHub.io](https://cpfhub.io)

---

# Português

[🇺🇸 English](#cpfhub-java-java-sdk-for-cpfhubio) | 🇧🇷 **Português**

**SDK Java oficial para [CPFHub.io](https://cpfhub.io) — API de Consulta de CPF Brasileiro**

---

## O que é o CPFHub.io?

O CPFHub.io é uma API REST que retorna nome, gênero e data de nascimento de qualquer CPF brasileiro — em ~300ms, com 99,9% de uptime e total conformidade com a LGPD.

**10M+ CPFs consultados · 1.300+ empresas ativas · 99,9% uptime**

---

## Instalação

**Maven:**

```xml
<dependency>
  <groupId>io.cpfhub</groupId>
  <artifactId>cpfhub-java</artifactId>
  <version>1.0.0</version>
</dependency>
```

**Gradle:**

```groovy
implementation 'io.cpfhub:cpfhub-java:1.0.0'
```

---

## Início Rápido

```java
import io.cpfhub.CPFHubClient;
import io.cpfhub.CPFResult;

CPFHubClient client = new CPFHubClient("SUA_CHAVE_DE_API");

CPFResult result = client.lookup("00000000000");

System.out.println(result.getName());      // "Fulano de Tal"
System.out.println(result.getGender());    // "M"
System.out.println(result.getBirthDate()); // "15/06/1990"
```

Obtenha sua chave de API gratuita em [app.cpfhub.io](https://app.cpfhub.io) — sem cartão de crédito.

---

## Exemplo curl

```bash
curl -X GET "https://api.cpfhub.io/cpf/12345678909" \
  -H "x-api-key: SUA_CHAVE_DE_API"
```

**Resposta:**

```json
{
  "success": true,
  "data": {
    "cpf": "12345678909",
    "name": "Fulano de Tal",
    "nameUpper": "FULANO DE TAL",
    "gender": "M",
    "birthDate": "15/06/1990",
    "day": 15,
    "month": 6,
    "year": 1990
  }
}
```

---

## Suporte a Async

```java
import io.cpfhub.CPFHubClient;
import java.util.concurrent.CompletableFuture;

CPFHubClient client = new CPFHubClient("SUA_CHAVE_DE_API");

CompletableFuture<CPFResult> future = client.lookupAsync("00000000000");
future.thenAccept(result -> System.out.println(result.getName()));
```

---

## Referência da API

### `new CPFHubClient(String apiKey)`

### `new CPFHubClient(String apiKey, CPFHubConfig config)`

```java
import java.time.Duration;
import io.cpfhub.CPFHubConfig;

CPFHubConfig config = CPFHubConfig.builder()
    .timeout(Duration.ofSeconds(5))
    .baseUrl("https://api.cpfhub.io")
    .build();

CPFHubClient client = new CPFHubClient("SUA_CHAVE_DE_API", config);
```

### `client.lookup(String cpf) → CPFResult`

Consulta um CPF e retorna os dados de identidade associados.

Aceita CPF com ou sem formatação (`000.000.000-00` ou `00000000000`).

#### Métodos de `CPFResult`

| Método | Retorno | Descrição |
|--------|---------|-----------|
| `getCpf()` | `String` | CPF (apenas dígitos) |
| `getName()` | `String` | Nome completo — `"Fulano de Tal"` |
| `getNameUpper()` | `String` | Nome completo em maiúsculas |
| `getGender()` | `String` | `"M"` ou `"F"` |
| `getBirthDate()` | `String` | Data de nascimento — `"DD/MM/YYYY"` |
| `getDay()` | `int` | Dia de nascimento |
| `getMonth()` | `int` | Mês de nascimento |
| `getYear()` | `int` | Ano de nascimento |

---

## Tratamento de Erros

```java
import io.cpfhub.CPFHubClient;
import io.cpfhub.exceptions.CPFHubException;

CPFHubClient client = new CPFHubClient("SUA_CHAVE_DE_API");

try {
    CPFResult result = client.lookup("00000000000");
    System.out.println(result.getName());
} catch (CPFHubException e) {
    System.err.printf("Erro %d: %s%n", e.getStatusCode(), e.getMessage());
    // 400 — Formato de CPF inválido
    // 401 — Chave de API inválida ou ausente
    // 404 — CPF não encontrado
    // 429 — Limite de requisições excedido
    // 500 — Erro no servidor
    // 503 — Serviço temporariamente indisponível
}
```

---

## Exemplos

### Spring Boot

```java
// CPFHubConfig.java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import io.cpfhub.CPFHubClient;

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
import org.springframework.stereotype.Service;
import io.cpfhub.CPFHubClient;
import io.cpfhub.CPFResult;

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
import io.cpfhub.CPFHubClient
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch
import kotlinx.coroutines.withContext

val client = CPFHubClient(BuildConfig.CPFHUB_API_KEY)

viewModelScope.launch {
    val result = withContext(Dispatchers.IO) {
        client.lookup("00000000000")
    }
    println(result.name)
}
```

---

## Limites de Requisição

| Plano | Limite |
|---|---|
| Gratuito | 1 requisição a cada 2 segundos · 50 requisições/mês |
| Pro | 1 requisição por segundo · 1.000 requisições/mês |
| Corporativo | Personalizado |

O SDK faz retry automático no erro `429` com backoff exponencial (até 3 tentativas).

---

## Planos e Preços

| Plano | Preço | Incluído | Extra |
|-------|-------|----------|-------|
| **Gratuito** | R$ 0/mês | 50 consultas | — |
| **Pro** | R$ 149/mês | 1.000 consultas | R$ 0,15/consulta |
| **Corporativo** | Personalizado | Personalizado | Personalizado |

[Ver preços completos em cpfhub.io →](https://cpfhub.io#pricing)

---

## Requisitos

- Java 11+
- Kotlin 1.8+ (opcional)

---

## Links

- [Documentação](https://cpfhub.io/documentacao)
- [Dashboard](https://app.cpfhub.io)
- [Maven Central](https://central.sonatype.com/artifact/io.cpfhub/cpfhub-java)
- [Página de Status](https://app.cpfhub.io/status)
- [Preços](https://cpfhub.io#pricing)
- [Conformidade LGPD](https://cpfhub.io/lgpd)
- [Especificação OpenAPI](https://github.com/cpfhub/cpfhub-openapi/blob/main/openapi.yaml)
- [Servidor MCP (Agentes de IA)](https://github.com/cpfhub/cpfhub-mcp)

---

## Licença

MIT © [CPFHub.io](https://cpfhub.io)
