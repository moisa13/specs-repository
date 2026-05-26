# Exemplo — Log de requisição concluída com sucesso

## Contexto

Requisição `GET /users/42` processada com sucesso. O `correlationId` foi gerado pelo middleware no início do ciclo e propagado automaticamente para todas as entradas de log da requisição.

---

## Entradas geradas

### Entrada do interceptor ao concluir a requisição

```json
{
  "level": "info",
  "message": "GET /users/42 200 45ms",
  "correlationId": "b3d1c2e4-8f7a-4b0e-9c3d-2a1f5e6b7c8d",
  "timestamp": "2026-05-25T14:32:10.123Z",
  "context": "LoggingInterceptor"
}
```

### Entrada gerada pelo service durante o processamento

```json
{
  "level": "debug",
  "message": "Buscando usuário no banco de dados",
  "correlationId": "b3d1c2e4-8f7a-4b0e-9c3d-2a1f5e6b7c8d",
  "timestamp": "2026-05-25T14:32:10.090Z",
  "context": "UsersService"
}
```

> O mesmo `correlationId` aparece em ambas as entradas sem ser passado explicitamente pelo código de negócio.

---

## Arquivos onde as entradas aparecem

| Arquivo | Presente |
|---------|----------|
| `2026-05-25.log` | ✅ |
| `2026-05-25-error.log` | ❌ |
