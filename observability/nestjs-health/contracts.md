# Contracts — NestJS — Health Check (Liveness + Readiness)

> Contratos de estrutura, endpoints, variáveis de ambiente e formato de resposta. O agente deve seguir estes contratos ao configurar os endpoints de health.

---

## Estrutura de diretórios

```
src/
└── health/
    ├── health.controller.ts
    ├── health.exception-filter.ts
    └── health.module.ts
```

---

## Pacotes a instalar

| Pacote | Versão mínima | Finalidade |
|--------|---------------|------------|
| `@nestjs/terminus` | ^11.0.0 | Indicadores de health e integração com NestJS |

---

## Variáveis de ambiente

As variáveis abaixo devem ser adicionadas ao schema Joi em `src/config/env.validation.ts` e ao `.env.example`.

| Variável | Tipo | Obrigatória | Validação Joi | Descrição |
|----------|------|-------------|---------------|-----------|
| `HEALTH_MEMORY_HEAP_THRESHOLD_MB` | number | ❌ | `Joi.number().integer().min(1).default(150)` | Limite de heap em MB acima do qual `/health/ready` retorna 503 |
| `HEALTH_MEMORY_RSS_THRESHOLD_MB` | number | ❌ | `Joi.number().integer().min(1).default(300)` | Limite de RSS em MB acima do qual `/health/ready` retorna 503 |

Adições ao `.env.example`:

```dotenv
# Limite de memória heap para o health check (em MB)
HEALTH_MEMORY_HEAP_THRESHOLD_MB=150

# Limite de memória RSS para o health check (em MB)
HEALTH_MEMORY_RSS_THRESHOLD_MB=300
```

---

## Endpoints

| Método | Path | Finalidade | HTTP success | HTTP failure | Tag Swagger |
|--------|------|------------|--------------|--------------|-------------|
| `GET` | `/health/live` | Liveness — confirma que o processo está no ar | 200 | — | `health` |
| `GET` | `/health/ready` | Readiness — verifica banco e memória | 200 | 503 | `health` |

### DTOs de resposta

| Classe | Finalidade |
|--------|------------|
| `HealthIndicatorResultDto` | Representa o resultado de um indicador individual (`{ status: string }`) |
| `HealthCheckResponseDto` | Envelope completo do Terminus (`status`, `info`, `error`, `details`) |

Ambas as classes devem ser registradas explicitamente no Swagger via `@ApiExtraModels(HealthIndicatorResultDto, HealthCheckResponseDto)` no controller. Os campos do tipo `Record<string, HealthIndicatorResultDto>` devem usar `getSchemaPath(HealthIndicatorResultDto)` no `additionalProperties` para que o schema seja resolvido corretamente.

### Decorators Swagger por endpoint

| Endpoint | Decorators |
|----------|------------|
| Controller | `@ApiTags('health')`, `@ApiExtraModels(HealthIndicatorResultDto, HealthCheckResponseDto)` |
| `GET /health/live` | `@ApiOperation({ summary: 'Liveness probe' })`, `@ApiOkResponse({ type: HealthCheckResponseDto, description: 'Processo ativo' })` |
| `GET /health/ready` | `@ApiOperation({ summary: 'Readiness probe' })`, `@ApiOkResponse({ type: HealthCheckResponseDto, description: 'Todos os indicadores saudáveis' })`, `@ApiServiceUnavailableResponse({ description: 'Um ou mais indicadores com falha' })` |

---

## Formato de resposta

O Terminus retorna um envelope padrão em ambos os endpoints. Os campos `info` e `error` contêm apenas os indicadores com status `up` e `down`, respectivamente. O campo `details` sempre contém todos os indicadores.

### Sucesso (HTTP 200)

```json
{
  "status": "ok",
  "info": {
    "database": { "status": "up" },
    "memory_heap": { "status": "up" },
    "memory_rss": { "status": "up" }
  },
  "error": {},
  "details": {
    "database": { "status": "up" },
    "memory_heap": { "status": "up" },
    "memory_rss": { "status": "up" }
  }
}
```

### Falha parcial (HTTP 503)

```json
{
  "status": "error",
  "info": {
    "memory_heap": { "status": "up" },
    "memory_rss": { "status": "up" }
  },
  "error": {
    "database": {
      "status": "down",
      "message": "connect ECONNREFUSED 127.0.0.1:5432"
    }
  },
  "details": {
    "database": {
      "status": "down",
      "message": "connect ECONNREFUSED 127.0.0.1:5432"
    },
    "memory_heap": { "status": "up" },
    "memory_rss": { "status": "up" }
  }
}
```

### Resposta de `/health/live` (HTTP 200)

```json
{
  "status": "ok",
  "info": {
    "live": { "status": "up" }
  },
  "error": {},
  "details": {
    "live": { "status": "up" }
  }
}
```

---

## Nomes dos indicadores

| Indicador | Nome no JSON | Endpoint |
|-----------|-------------|----------|
| Processo ativo | `live` | `/health/live` |
| Banco de dados | `database` | `/health/ready` |
| Memória heap | `memory_heap` | `/health/ready` |
| Memória RSS | `memory_rss` | `/health/ready` |
