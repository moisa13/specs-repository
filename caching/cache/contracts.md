# Contracts — Caching — Cache de Dados

---

## Configuração

| Variável | Tipo | Obrigatório | Padrão | Descrição |
|----------|------|-------------|--------|-----------|
| `CACHE_ENABLED` | boolean | ❌ | `true` | Habilita ou desabilita o cache globalmente |
| `CACHE_KEY_PREFIX` | string | ✅ | — | Prefixo aplicado a todas as chaves; deve ser único por aplicação |
| `CACHE_TTL_MULTIPLIER` | number | ❌ | `1` | Multiplicador aplicado a todos os TTLs; `0.5` reduz à metade; `2` dobra |

TTLs por recurso são definidos pela implementação e configurados via variáveis próprias de cada módulo de domínio.

---

## Contratos das operações base

| Operação | Entrada | Saída | Garantias |
|----------|---------|-------|-----------|
| `get(key)` | `key: string` — chave sem prefixo | `Promise<T \| null>` — valor desserializado ou `null` | Retorna `null` em miss; nunca lança exceção ao caller; retorna `null` se Redis indisponível |
| `set(key, value, ttl)` | `key: string`; `value: unknown`; `ttl: number` (segundos, antes do multiplicador) | `Promise<void>` | TTL efetivo = arredondar(ttl × multiplicador); no-op quando `CACHE_ENABLED=false` |
| `delete(key)` | `key: string` — chave sem prefixo | `Promise<void>` | Remove chave exata; no-op quando `CACHE_ENABLED=false` ou chave inexistente |

---

## Contrato da operação `remember`

**Entrada:**

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `key` | string | Chave completa sem o prefixo — o prefixo é aplicado pela camada de cache |
| `ttl` | number | TTL em segundos antes do multiplicador global |
| `fn` | `() => Promise<T>` | Função que produz o valor caso não haja cache |

**Saída:**

| Campo | Tipo | Descrição |
|-------|------|-----------|
| Retorno | `Promise<T>` | Valor do cache (hit) ou resultado de `fn()` (miss) |

**Garantias:**
- `fn()` é executada no máximo uma vez por miss concorrente, graças ao double-checked locking
- Se `CACHE_ENABLED=false`, `fn()` é executada sempre; cache não é lido nem gravado
- Se Redis estiver indisponível, `fn()` é executada como fallback; nenhuma exceção é propagada ao caller

---

## Contrato da operação `deletePattern`

**Entrada:**

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `pattern` | string | Padrão glob para correspondência de chaves (ex: `users:*`) — prefixo é aplicado automaticamente |

**Garantias:**
- Varredura incremental — não bloqueia o Redis independentemente do volume de chaves
- Todas as chaves correspondentes são removidas antes que a operação retorne
- O recurso para o evento de invalidação é derivado como o **primeiro segmento do padrão antes do primeiro `:`** — ex: `users:42:*` → `users`; `products:list:*` → `products`
- No-op quando `CACHE_ENABLED=false` — nenhuma chave é varrida ou removida; nenhum evento é publicado

---

## Evento de invalidação

Publicado no canal `cache-events:{recurso}` após remoção de chaves.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `resource` | string | Nome do recurso invalidado |
| `keys` | string[] | Lista das chaves removidas (sem prefixo) |
| `timestamp` | string | ISO 8601 |

> A publicação é melhor-esforço. Consumidores do evento não devem assumir entrega garantida.

---

## Padrão de namespacing de chaves

```
{recurso}:{identificador}
{recurso}:{identificador}:{sub-recurso}:{identificador}
{recurso}:list:{hashDosFiltros}
```

O prefixo configurado em `CACHE_KEY_PREFIX` é anteposto automaticamente pela camada de cache. O caller opera sempre sem o prefixo.

**Chaves de lock** seguem o padrão `{prefixo}:lock:{chave}` — o segmento `lock` é reservado e garante que nunca colidam com chaves de dados. O caller nunca opera diretamente com chaves de lock.

**TTL efetivo** é sempre um inteiro: `arredondar(ttl × CACHE_TTL_MULTIPLIER)`. A camada de cache aplica o arredondamento antes de gravar — TTLs fracionários não são válidos no Redis.

---

## Adições ao agents.md

Ao aplicar esta spec, acrescentar a seção abaixo ao `agents.md` do projeto:

```markdown
## Cache (caching/cache)

- Nunca construir chaves de cache com concatenação livre — usar o padrão hierárquico `{recurso}:{id}`
- Nunca chamar Redis diretamente fora do `CacheService` — toda operação passa por ele
- Para cachear resultados com consistência entre instâncias, usar `remember()` — nunca `get` + `set` manual
- `deletePattern` usa varredura incremental; nunca usar comandos bloqueantes (KEYS) para remoção em massa
- O prefixo de chave é aplicado automaticamente — nunca incluí-lo na construção da chave no caller
- Chaves de lock usam o segmento reservado `lock` — padrão `{prefixo}:lock:{chave}`; nunca colidem com chaves de dados
- TTL efetivo é sempre inteiro: `arredondar(ttl × CACHE_TTL_MULTIPLIER)`; TTLs fracionários não são válidos
```
