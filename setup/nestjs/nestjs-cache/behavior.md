# Behavior — Setup NestJS — Cache

---

## Objetivo

Implementar o comportamento de `caching/cache` em um projeto NestJS, fornecendo os componentes necessários para cache de dados com consistência distribuída, invalidação controlada e aquecimento no boot.

---

## Componentes

### CacheModule

Módulo NestJS decorado com `@Global()`. Deve:
- Criar e registrar a conexão ioredis como provider (`CACHE_REDIS_CLIENT`)
- Declarar `CacheService`, `CacheWarmingService` e `CacheEventsService` em `providers`
- Exportar apenas `CacheService` — `CacheEventsService` e `CacheWarmingService` são internos ao módulo
- Ser importado uma única vez no `AppModule`

A anotação `@Global()` garante que `CacheService` esteja disponível para injeção em qualquer parte da aplicação sem que cada módulo de feature precise importar `CacheModule`.

### CacheService

Serviço `@Injectable()` que opera diretamente sobre o cliente ioredis. Implementa:

- `get<T>(key: string): Promise<T | null>` — lê e desserializa; retorna `null` em miss ou quando `CACHE_ENABLED=false`
- `set(key, value, ttl): Promise<void>` — serializa e grava com TTL efetivo (ttl × multiplicador); no-op quando `CACHE_ENABLED=false`
- `delete(key): Promise<void>` — remove chave exata; no-op quando `CACHE_ENABLED=false`
- `deletePattern(pattern): Promise<void>` — varredura incremental via SCAN + DEL em lotes; no-op quando `CACHE_ENABLED=false`
- `remember<T>(key, ttl, fn): Promise<T>` — cache-aside com double-checked locking via SET NX PX; executa `fn()` diretamente quando `CACHE_ENABLED=false` ou Redis indisponível

O prefixo (`CACHE_KEY_PREFIX`) é aplicado internamente em todas as operações. O caller nunca inclui o prefixo.

### GenericCacheService

Classe abstrata que módulos de domínio estendem para padronizar operações de cache em seus recursos. Injetada com `CacheService`. Fornece:

- `rememberById<T>(resource, id, ttl, fn)` — delega para `CacheService.remember` com chave `{resource}:{id}`
- `rememberList<T>(resource, params, ttl, fn)` — chave `{resource}:list:{hash(params)}`; o hash é gerado a partir da serialização estável dos parâmetros
- `invalidateById(resource, id)` — delega para `CacheService.delete`; o evento de invalidação é publicado pelo próprio `CacheService`
- `invalidateAll(resource)` — delega para `CacheService.deletePattern('{resource}:*')`; o evento é publicado pelo próprio `CacheService`

Módulos de domínio estendem `GenericCacheService` e definem seus próprios métodos com os recursos e TTLs específicos.

### CacheWarmingService

Serviço `@Injectable()` que implementa `OnApplicationBootstrap`. Deve:
- Receber uma lista de warmers registrados via injeção (array de `CacheWarmer`)
- No `onApplicationBootstrap()`: executar cada warmer em sequência
- Erros em um warmer individual são capturados, logados e não interrompem os demais
- Após todos os warmers, logar o resultado agregado (sucesso / falha por warmer)

**CacheWarmer** é uma interface que os módulos de domínio implementam para registrar recursos a aquecer:

```
interface CacheWarmer {
  warm(): Promise<void>;
}
```

Cada warmer é responsável por chamar `CacheService.set` ou `GenericCacheService.rememberById` para seus recursos.

### CacheEventsService

Serviço `@Injectable()` responsável por publicar eventos de invalidação no Redis Pub/Sub. Deve:
- Receber eventos de invalidação de `CacheService`
- Aplicar debounce de 100ms: eventos do mesmo recurso na mesma janela são mergeados em um único payload
- Publicar no canal `cache-events:{resource}` com o payload definido em `caching/cache/contracts.md`
- Falha na publicação é logada e não propaga exceção

---

## Fluxo de inicialização

1. `CacheModule` importado no `AppModule` — cria conexão ioredis e registra todos os providers
2. No `onApplicationBootstrap()` do `CacheWarmingService`: executa os warmers registrados

---

## Fluxo de uma operação remember

```
caller.remember('users:42', 300, fn)
  → CacheService aplica prefixo → 'myapp:users:42'
  → GET 'myapp:users:42'
  → Hit: desserializa e retorna
  → Miss: SET NX PX 'myapp:lock:users:42' TTL_LOCK
    → Adquiriu: executa fn(), SET 'myapp:users:42', DEL lock
    → Não adquiriu: aguarda, GET 'myapp:users:42'
      → Hit: retorna valor
      → Null: executa fn() diretamente (produtor caiu antes de gravar)
```

---

## Edge cases

| Situação | Comportamento esperado |
|----------|----------------------|
| `CACHE_ENABLED=false` | Todas as operações são no-op ou passthrough; `remember` executa `fn()` diretamente |
| Redis indisponível durante `remember` | `fn()` é executada como fallback; erro de conexão é logado; nenhuma exceção ao caller |
| Lock expira sem gravação (produtor caiu) | Instâncias que aguardavam releem e encontram `null`; executam `fn()` diretamente como fallback |
| Warmer falha no boot | Erro logado; demais warmers executam normalmente; aplicação sobe |
| `deletePattern` em padrão sem correspondências | Operação concluída sem erro; nenhuma chave removida; evento não publicado |
| Módulo de domínio não registra warmer | `CacheWarmingService` executa com lista vazia; sem erro |

---

## Regras de implementação

- O prefixo de chave é sempre aplicado por `CacheService` — nunca pelo caller
- Chaves de lock seguem o padrão `{prefixo}:lock:{chave}` — segmento `lock` reservado para evitar colisão com chaves de dados
- TTL efetivo é sempre um inteiro: `arredondar(ttl × CACHE_TTL_MULTIPLIER)`; aplicado por `CacheService` antes de gravar
- O recurso para o evento de invalidação é o primeiro segmento do padrão antes do primeiro `:` — ex: `users:42:*` → `users`
- `deletePattern` nunca usa `KEYS`; sempre SCAN iterativo com cursor
- Erros de cache não propagam para o fluxo principal quando há fallback via `fn()`
- `CacheModule` é `@Global()` — sem essa anotação, módulos de domínio não conseguem injetar `CacheService` sem importar `CacheModule` explicitamente
- A conexão ioredis do cache usa `REDIS_CACHE_DB` — nunca o banco de filas ou sessão

---

## O que esta spec NÃO cobre

- Notificação em tempo real para clientes via WebSocket → escopo de real-time/WebSocket
- Cache de sessão → escopo de autenticação
- Decorators `@CacheKey` / `@CacheTTL` do `@nestjs/cache-manager` → não utilizados nesta spec
