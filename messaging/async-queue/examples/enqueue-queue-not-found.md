# Exemplo — Fila não registrada

## Contexto

Produtor tenta enfileirar um job em uma fila que não existe no sistema.

---

## Entrada

```json
{
  "queueName": "fila-inexistente",
  "jobType": "process-order",
  "payload": {
    "orderId": "ord_abc123"
  }
}
```

## Saída — Falha

```json
{
  "error": {
    "code": "QUEUE_NOT_FOUND",
    "message": "A fila 'fila-inexistente' não está registrada"
  }
}
```

## Notas

- Nenhum job é criado
- O chamador deve tratar o erro e decidir se deve registrar a fila ou revisar o nome utilizado
