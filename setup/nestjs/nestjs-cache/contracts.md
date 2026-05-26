# Contracts — Setup NestJS — Cache

> Contratos específicos para a implementação NestJS. Para os contratos de comportamento (remember, deletePattern, evento de invalidação), ver `caching/cache/contracts.md`.

---

## Variáveis de ambiente

| Variável | Tipo | Obrigatório | Padrão | Descrição |
|----------|------|-------------|--------|-----------|
| `REDIS_HOST` | string | ✅ | — | Host do Redis |
| `REDIS_PORT` | number | ❌ | `6379` | Porta do Redis |
| `REDIS_PASSWORD` | string | ❌ | — | Senha do Redis |
| `REDIS_CACHE_DB` | number | ✅ | — | Banco Redis dedicado para cache; deve ser diferente do banco de filas |
| `CACHE_ENABLED` | boolean | ❌ | `true` | Desabilita o cache globalmente quando `false` |
| `CACHE_KEY_PREFIX` | string | ✅ | — | Prefixo aplicado a todas as chaves |
| `CACHE_TTL_MULTIPLIER` | number | ❌ | `1` | Multiplicador global de TTL |
| `CACHE_LOCK_TTL` | number | ❌ | `5000` | TTL do lock distribuído em milissegundos |

Todas as variáveis devem ser adicionadas ao schema de validação de ambiente do projeto (conforme `setup/nestjs/nestjs-init`).

---

## Schema de validação (Joi)

```typescript
REDIS_HOST: Joi.string().required(),
REDIS_PORT: Joi.number().default(6379),
REDIS_PASSWORD: Joi.string().optional(),
REDIS_CACHE_DB: Joi.number().required(),
CACHE_ENABLED: Joi.boolean().default(true),
CACHE_KEY_PREFIX: Joi.string().required(),
CACHE_TTL_MULTIPLIER: Joi.number().positive().default(1),
CACHE_LOCK_TTL: Joi.number().positive().default(5000),
```

---

## Convenções de nomenclatura

| Artefato | Nome da classe | Localização |
|----------|---------------|-------------|
| Módulo | `CacheModule` | `src/cache/cache.module.ts` |
| Serviço base | `CacheService` | `src/cache/cache.service.ts` |
| Serviço abstrato de domínio | `GenericCacheService` | `src/cache/generic-cache.service.ts` |
| Serviço de aquecimento | `CacheWarmingService` | `src/cache/cache-warming.service.ts` |
| Serviço de eventos | `CacheEventsService` | `src/cache/cache-events.service.ts` |
| Interface de warmer | `CacheWarmer` | `src/cache/interfaces/cache-warmer.interface.ts` |
| Token do cliente Redis | `CACHE_REDIS_CLIENT` | constante em `src/cache/cache.constants.ts` |
| Token de registro de warmers | `CACHE_WARMERS` | constante em `src/cache/cache.constants.ts` |

---

## Adições ao agents.md

Ao aplicar esta spec, acrescentar a seção abaixo ao `agents.md` do projeto:

```markdown
## Cache (setup/nestjs/nestjs-cache)

- O serviço central é `CacheService` em `src/cache/cache.service.ts`; nunca usar ioredis diretamente fora dele
- Para cachear resultados com consistência entre instâncias, usar `remember()` — nunca `get` + `set` manual
- Módulos de domínio devem estender `GenericCacheService` para padronizar operações de cache de seus recursos
- Para registrar recursos a pré-aquecer no boot, implementar a interface `CacheWarmer` e registrar via `CACHE_WARMERS`
- O prefixo de chave é aplicado automaticamente — nunca incluí-lo ao construir a chave no caller
- `REDIS_CACHE_DB` deve ser diferente do banco de filas (`REDIS_DB` em `setup/nestjs/nestjs-queue`) e de sessão
```
