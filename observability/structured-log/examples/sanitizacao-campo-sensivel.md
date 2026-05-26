# Exemplo — Sanitização de campo sensível

## Contexto

O handler de exceções captura uma exceção e inclui o contexto da requisição no log. O body da requisição contém um campo `password` e um objeto aninhado com `authorizationHeader`.

---

## Entrada antes da sanitização

```json
{
  "body": {
    "email": "user@example.com",
    "password": "minha-senha-secreta",
    "metadata": {
      "authorizationHeader": "Bearer eyJhbGciOiJIUzI1NiJ9..."
    }
  }
}
```

## Entrada após a sanitização

```json
{
  "body": {
    "email": "user@example.com",
    "password": "[REDACTED]",
    "metadata": {
      "authorizationHeader": "[REDACTED]"
    }
  }
}
```

---

## Regras aplicadas

| Campo | Regra disparada |
|-------|----------------|
| `password` | Nome exato na lista de campos protegidos |
| `authorizationHeader` | Contém o padrão `auth` (case-insensitive) |

---

## Entrada final gravada no arquivo

```json
{
  "level": "warn",
  "message": "Unauthorized",
  "correlationId": "c0ffee00-dead-beef-cafe-123456789abc",
  "timestamp": "2026-05-25T14:40:01.555Z",
  "context": "HttpExceptionFilter",
  "statusCode": 401,
  "body": {
    "email": "user@example.com",
    "password": "[REDACTED]",
    "metadata": {
      "authorizationHeader": "[REDACTED]"
    }
  }
}
```
