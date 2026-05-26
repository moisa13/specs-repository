# Exemplo — Log de exceção 4xx

## Contexto

Requisição `POST /users` com body inválido (campo obrigatório ausente). A exceção tem status 400.

---

## Entrada gerada

```json
{
  "level": "warn",
  "message": "Validation failed",
  "correlationId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "timestamp": "2026-05-25T14:33:05.210Z",
  "context": "HttpExceptionFilter",
  "statusCode": 400,
  "path": "/users",
  "method": "POST"
}
```

> Nível `warn`. Sem campo `stack`. O body da requisição, se logado, passa pela sanitização antes.

---

## Arquivos onde a entrada aparece

| Arquivo | Presente |
|---------|----------|
| `2026-05-25.log` | ✅ |
| `2026-05-25-error.log` | ❌ |
