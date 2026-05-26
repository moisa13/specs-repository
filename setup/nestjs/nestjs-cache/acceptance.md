# Acceptance Criteria — Setup NestJS — Cache

> Formato: **Dado** [contexto] **Quando** [ação] **Então** [resultado esperado]

---

## CacheService

**AC-01 — remember retorna valor do cache sem executar fn()**
- **Dado** que a chave `users:42` existe no Redis com prefixo aplicado
- **Quando** `cacheService.remember('users:42', 300, fn)` é chamado
- **Então** retorna o valor desserializado sem executar `fn()`

**AC-02 — remember executa fn() em miss e grava com TTL efetivo**
- **Dado** que a chave `users:42` não existe no Redis
- **E** `CACHE_TTL_MULTIPLIER=2`
- **Quando** `cacheService.remember('users:42', 300, fn)` é chamado
- **Então** executa `fn()`, grava com TTL de 600 segundos e retorna o valor

**AC-03 — Prefixo aplicado automaticamente**
- **Dado** que `CACHE_KEY_PREFIX=myapp`
- **Quando** qualquer operação de `CacheService` é executada com a chave `users:42`
- **Então** a chave no Redis é `myapp:users:42`; o caller nunca inclui o prefixo

**AC-04 — CACHE_ENABLED=false torna todas as operações no-op ou passthrough**
- **Dado** que `CACHE_ENABLED=false`
- **Quando** `remember`, `get`, `set`, `delete` ou `deletePattern` são chamados
- **Então** `remember` executa `fn()` diretamente; `get` retorna `null`; `set`, `delete` e `deletePattern` retornam sem interagir com o Redis; nenhum evento é publicado

**AC-05 — deletePattern usa SCAN, não KEYS**
- **Dado** que existem chaves com padrão `users:*` no Redis
- **Quando** `cacheService.deletePattern('users:*')` é chamado
- **Então** a varredura usa SCAN iterativo com cursor; o comando `KEYS` não é usado

---

## GenericCacheService

**AC-06 — rememberById delega para CacheService com chave correta**
- **Dado** que um serviço de domínio estende `GenericCacheService`
- **Quando** `rememberById('users', 42, 300, fn)` é chamado
- **Então** `CacheService.remember` é chamado com a chave `users:42`

**AC-07 — rememberList usa hash dos parâmetros como sufixo**
- **Dado** que `rememberList('products', { status: 'active', page: 2 }, 300, fn)` é chamado
- **Quando** a chave é construída
- **Então** a chave contém `products:list:{hash}` onde `{hash}` é determinístico para os mesmos parâmetros

**AC-08 — invalidateAll remove todas as chaves do recurso e emite evento**
- **Dado** que existem múltiplas chaves `users:*` no Redis
- **Quando** `invalidateAll('users')` é chamado
- **Então** todas as chaves `users:*` são removidas e um evento é publicado em `cache-events:users`

---

## CacheWarmingService

**AC-09 — Warmers executam no onApplicationBootstrap**
- **Dado** que dois warmers estão registrados via `CACHE_WARMERS`
- **Quando** a aplicação inicializa
- **Então** ambos os warmers são executados em sequência no `onApplicationBootstrap`

**AC-10 — Falha em um warmer não impede os demais**
- **Dado** que o primeiro warmer lança exceção
- **Quando** o `CacheWarmingService` itera os warmers
- **Então** o erro é logado; o segundo warmer é executado normalmente; a aplicação sobe

---

## CacheEventsService

**AC-11 — Eventos do mesmo recurso na janela de debounce são mergeados**
- **Dado** que três eventos de invalidação do recurso `users` chegam em 50ms
- **Quando** o debounce de 100ms expira
- **Então** um único payload é publicado em `cache-events:users` com as chaves das três invalidações

**AC-12 — Falha na publicação do evento não propaga exceção**
- **Dado** que o Redis Pub/Sub está indisponível
- **Quando** `CacheEventsService` tenta publicar
- **Então** o erro é logado e nenhuma exceção é propagada ao caller

---

## CacheModule

**AC-13 — CacheService injetável em módulos de domínio sem importar CacheModule**
- **Dado** que `CacheModule` está importado no `AppModule` e decorado com `@Global()`
- **Quando** um módulo de domínio injeta `CacheService` sem declarar `CacheModule` em seus imports
- **Então** a injeção é resolvida corretamente pelo container NestJS

---

## O que caracteriza uma implementação incorreta

- `KEYS` usado para varredura de chaves em vez de SCAN
- Prefixo incluído pelo caller ao construir a chave
- `fn()` executada mais de uma vez por miss concorrente (falta double-checked locking)
- `CACHE_ENABLED=false` ainda interagindo com o Redis
- `CacheModule` sem `@Global()` — módulos de domínio falham na resolução de `CacheService`
- Banco Redis de cache compartilhado com filas (`REDIS_QUEUE_DB`)

---

## Cobertura mínima de testes

- [ ] `CacheService.remember`: hit retorna cache; miss executa fn() e grava; concorrência com locking
- [ ] `CacheService.remember`: exceção em fn() libera lock e propaga erro
- [ ] `CacheService.deletePattern`: usa SCAN; remove todas as chaves correspondentes
- [ ] `CacheService`: `CACHE_ENABLED=false` — todas as operações são no-op ou passthrough
- [ ] `GenericCacheService.rememberList`: mesmos parâmetros geram mesma chave (hash determinístico)
- [ ] `CacheWarmingService`: falha em um warmer não impede os demais
