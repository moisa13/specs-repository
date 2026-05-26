# Exemplo — Retry esgotado; job movido para DLQ

## Contexto

Um job de processamento de relatório falha nas 3 tentativas configuradas. O sistema registra a falha e move o job para a dead letter queue.

---

## Estado do job após cada tentativa

**Tentativa 1** — falha; aguarda 1s em `delayed` (backoff exponencial, delay base 1000ms)

```json
{
  "jobId": "reports:1748200000001:0",
  "queueName": "reports",
  "jobType": "generate-report",
  "status": "delayed",
  "attempts": 1,
  "createdAt": "2026-05-25T10:00:00Z",
  "failReason": "Connection timeout to storage service"
}
```

**Tentativa 2** — falha; aguarda 2s em `delayed`

```json
{
  "jobId": "reports:1748200000001:0",
  "queueName": "reports",
  "jobType": "generate-report",
  "status": "delayed",
  "attempts": 2,
  "createdAt": "2026-05-25T10:00:00Z",
  "failReason": "Connection timeout to storage service"
}
```

**Tentativa 3** — falha; tentativas esgotadas

```json
{
  "jobId": "reports:1748200000001:0",
  "queueName": "reports",
  "jobType": "generate-report",
  "status": "failed",
  "attempts": 3,
  "createdAt": "2026-05-25T10:00:00Z",
  "processedAt": "2026-05-25T10:00:06Z",
  "failReason": "Connection timeout to storage service"
}
```

## Notas

- Todos os timestamps (`createdAt`, `processedAt`) estão em UTC, indicado pelo sufixo `Z` (ISO 8601)
- `processedAt` (T+6s) = ~1s de execução por tentativa × 3 tentativas + 1s de delay após tentativa 1 + 2s de delay após tentativa 2; consistente com o ADR-002 (delays totais: 3s)
- Durante os períodos de backoff (tentativas 1 e 2), o status real do job é `delayed`, não `waiting`; o status retorna para `waiting` apenas quando o delay expira e o job aguarda um worker
- O job está agora na DLQ da fila `reports`
- Não será removido automaticamente
- Para reprocessar, o operador deve mover o job de volta para `waiting`; a contagem de tentativas reinicia do zero
