# Context — Setup NestJS — Cache

---

## Plataformas suportadas

| Plataforma | Suportada | Observações |
|------------|-----------|-------------|
| API REST | ✅ | Comportamento principal |
| Web SPA | ❌ | |
| Mobile (iOS/Android) | ❌ | |
| Backend (serviço interno) | ✅ | Inclui workers BullMQ quando integrados com `setup/nestjs/nestjs-queue` |

## Pré-requisitos

- [ ] Projeto NestJS inicializado conforme `setup/nestjs/nestjs-init`
- [ ] `caching/cache` lida e compreendida antes desta spec
- [ ] Redis disponível — banco dedicado para cache (`REDIS_CACHE_DB`), diferente do banco de filas

## Escopo

**Dentro do escopo:**
- `CacheService`: operações base sobre ioredis (get, set, delete, deletePattern, remember)
- `GenericCacheService`: classe abstrata para módulos de domínio
- `CacheWarmingService`: mecanismo de aquecimento no boot via `OnApplicationBootstrap`
- `CacheEventsService`: publicação de eventos de invalidação via Redis Pub/Sub com debounce
- `CacheModule`: módulo NestJS que registra e exporta os serviços
- Schema de variáveis de ambiente e validação via Joi
- Adições ao `agents.md` do projeto

**Fora do escopo:**
- `CacheEventsGateway` (WebSocket para notificação de clientes) → escopo de real-time/WebSocket
- Cache de sessão de usuário → escopo de autenticação
- Decorators `@CacheKey` / `@CacheTTL` do `@nestjs/cache-manager` → esta spec usa ioredis diretamente

## Bibliotecas externas

| Pacote | Versão mínima | Uso |
|--------|--------------|-----|
| `ioredis` | `^5.0.0` | Cliente Redis com suporte a Pub/Sub, SCAN e SET NX PX |
| `@types/ioredis` | bundled no pacote | Tipos incluídos no próprio `ioredis` — nenhum `@types/` adicional necessário |
