# Changelog — Setup NestJS — Cache

> Versões seguem Semantic Versioning: MAJOR.MINOR.PATCH
>
> - **MAJOR** — mudança que quebra implementações existentes
> - **MINOR** — nova regra ou edge case (implementações existentes precisam de revisão)
> - **PATCH** — correção de texto, sem mudança de comportamento

---

## [Não lançado]

### Adicionado
-

### Alterado
-

### Removido
-

---

## [0.2.0] — 2026-05

### Adicionado
- AC-08 (`invalidateById` + evento) e renumeração de AC-09 a AC-14
- Clarificação em `behavior.md`: `CacheEventsService` é responsável por adicionar o campo `timestamp` ao payload
- AC-04 atualizado para incluir `deletePattern` no cenário `CACHE_ENABLED=false`

### Alterado
- `behavior.md`: `GenericCacheService` não emite eventos — responsabilidade transferida para `CacheService`
- `behavior.md`: `CacheModule` exporta apenas `CacheService`; `CacheEventsService` e `CacheWarmingService` são internos
- `examples/configurar-generic-cache-service.md`: `CacheEventsService` removido do construtor de `GenericCacheService` e do exemplo de uso em domínio
- `examples/configurar-cache-module.md`: `CacheEventsService` removido dos exports
- `examples/configurar-cache-service.md`: derivação do recurso corrigida para `pattern.split(':')[0]` (era `pattern.replace(':*', '')`)
- `integration.md` e `contracts.md`: referências a `REDIS_QUEUE_DB` corrigidas para `REDIS_DB` (nome real em `setup/nestjs/nestjs-queue`)

---

## [0.1.0] — 2026-05

### Adicionado
- Versão inicial da spec
- Descrição dos componentes: `CacheService`, `GenericCacheService`, `CacheWarmingService`, `CacheEventsService`
- Fluxo de inicialização e fluxo de operação remember
- Contratos de variáveis de ambiente e schema Joi
- Convenções de nomenclatura e localização de arquivos
- Adições ao `agents.md`
- ADR-001 a ADR-004
