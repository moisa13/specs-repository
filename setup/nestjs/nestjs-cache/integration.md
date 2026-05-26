# Integration — Setup NestJS — Cache

---

## Dependências

| Módulo | Obrigatório | O que usa |
|--------|-------------|-----------|
| `setup/nestjs/nestjs-init` | ✅ | Estrutura base do projeto, schema de validação de env |
| `caching/cache` | ✅ | Comportamento que esta spec implementa |
| Redis | ✅ | Banco dedicado via `REDIS_CACHE_DB` |

---

## Dependentes

| Módulo | Como usa este módulo |
|--------|----------------------|
| Qualquer módulo de domínio | Estende `GenericCacheService` ou injeta `CacheService` diretamente |
| `setup/nestjs/nestjs-queue` | Usa Redis separado (`REDIS_DB`); não compartilha banco com este módulo |

---

## Pontos de integração no projeto

| Ponto | Integração |
|-------|-----------|
| `AppModule` | Importa `CacheModule`; registra warmers de domínio via token `CACHE_WARMERS` |
| Schema de env | Adiciona `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD`, `REDIS_CACHE_DB`, `CACHE_ENABLED`, `CACHE_KEY_PREFIX`, `CACHE_TTL_MULTIPLIER`, `CACHE_LOCK_TTL` |
| Módulos de domínio | Estendem `GenericCacheService`; implementam `CacheWarmer` se precisarem de warming |
| `onApplicationBootstrap` | `CacheWarmingService` executa todos os warmers registrados |

---

## Separação de bancos Redis

Esta spec usa um banco Redis dedicado (`REDIS_CACHE_DB`) para isolar o cache de filas e sessões:

| Uso | Variável | Banco sugerido |
|-----|----------|---------------|
| Cache de dados | `REDIS_CACHE_DB` | ex: `0` |
| Filas BullMQ | `REDIS_DB` | ex: `1` |

A separação evita que comandos de varredura (`SCAN`, `deletePattern`) interfiram em chaves de outros bancos.

---

## Eventos emitidos

| Canal | Quando | Consumidor esperado |
|-------|--------|---------------------|
| `cache-events:{resource}` | Após invalidação de chaves | Gateway WebSocket (fora do escopo desta spec) |

---

## Eventos consumidos

Esta spec não consome eventos de outros módulos.
