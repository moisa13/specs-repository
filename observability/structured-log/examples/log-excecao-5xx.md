# Exemplo — Log de exceção 5xx

## Contexto

Requisição `GET /orders/99` falhou com erro interno (banco de dados inacessível). A exceção tem status 500.

---

## Entrada gerada

```json
{
  "level": "error",
  "message": "Internal server error",
  "correlationId": "f9e8d7c6-b5a4-3210-fedc-ba9876543210",
  "timestamp": "2026-05-25T14:35:42.887Z",
  "context": "HttpExceptionFilter",
  "statusCode": 500,
  "path": "/orders/99",
  "method": "GET",
  "stack": "Error: connect ECONNREFUSED 127.0.0.1:5432\n    at TCPConnectWrap.afterConnect [as oncomplete] (node:net:1494:16)"
}
```

> Nível `error`. Campo `stack` presente com o stack trace completo. A entrada é gravada nos dois arquivos de log.

---

## Arquivos onde a entrada aparece

| Arquivo | Presente |
|---------|----------|
| `2026-05-25.log` | ✅ |
| `2026-05-25-error.log` | ✅ |
