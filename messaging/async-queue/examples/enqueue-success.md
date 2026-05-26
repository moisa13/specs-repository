# Exemplo — Enfileiramento bem-sucedido

## Contexto

Produtor enfileira um job de envio de e-mail com delay de 5 segundos e 3 tentativas máximas.

---

## Entrada

```json
{
  "queueName": "notifications",
  "jobType": "send-email",
  "payload": {
    "to": "user@example.com",
    "subject": "Confirmação de cadastro",
    "templateId": "welcome"
  },
  "options": {
    "delay": 5000,
    "attempts": 3,
    "backoff": {
      "type": "exponential",
      "delay": 1000
    }
  }
}
```

## Saída — Sucesso

```json
{
  "jobId": "notifications:1748200000000:0",
  "queueName": "notifications",
  "status": "delayed"
}
```

## Notas

- Status é `delayed` porque `options.delay > 0`; após 5 segundos o job entra em `waiting`
- Em caso de falha, o sistema aguarda 1s antes da 2ª tentativa, 2s antes da 3ª
- Após a 3ª falha, o job é movido para a DLQ da fila `notifications`
