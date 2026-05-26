# Changelog — Caching — Cache de Dados

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
- Comportamento de publicação de evento para `delete` (além de `deletePattern`) em `behavior.md` e `contracts.md`
- Regra: nenhum evento publicado quando `deletePattern` não corresponde a nenhuma chave
- Guidance sobre TTL do lock distribuído em `behavior.md`
- AC-11 (`delete` + evento) e renumeração dos ACs subsequentes (AC-12 a AC-16)
- Contratos formais de entrada/saída para `get`, `set` e `delete` em `contracts.md`
- Regra explícita de derivação do `resource` a partir do padrão em `deletePattern` (primeiro segmento antes do primeiro `:`)
- Formato de chave de lock como contrato: `{prefixo}:lock:{chave}` — segmento `lock` reservado
- Regra de arredondamento de TTL: `arredondar(ttl × CACHE_TTL_MULTIPLIER)` — sempre inteiro
- `deletePattern` adicionado à lista de no-ops de `CACHE_ENABLED=false` em `behavior.md` e `contracts.md`
- AC para `deletePattern` + `CACHE_ENABLED=false` (AC-08)
- Edge case: lock expira sem gravação — instâncias aguardando executam `fn()` diretamente (AC-04)
- Passos 5a e 5b no fluxo de `remember` em `behavior.md` e no exemplo `remember-cache-aside.md`

### Alterado
- `integration.md`: referência a `GenericCacheService` removida da spec de negócio — detalhe de implementação NestJS
- `contracts.md` (seção agents.md): adicionadas regras de chave de lock e arredondamento de TTL
- AC-10 a AC-15 renumerados por inserção de novos critérios

---

## [0.1.0] — 2026-05

### Adicionado
- Versão inicial da spec
- Padrão cache-aside com double-checked locking distribuído (`remember`)
- Operações base: get, set, delete, deletePattern
- Namespacing hierárquico de chaves com prefixo global
- TTLs configuráveis com multiplicador global
- Invalidação por chave exata, por padrão e publicação de evento em Pub/Sub
- Aquecimento de cache no boot (cache warming)
- Flag de desabilitação global `CACHE_ENABLED`
- ADR-001 a ADR-004
