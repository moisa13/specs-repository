# Behavior — Caching — Cache de Dados

---

## Objetivo

Definir como dados são armazenados, recuperados, invalidados e pré-aquecidos em cache, garantindo consistência entre instâncias concorrentes e controle granular sobre o ciclo de vida das entradas.

---

## Padrão principal: cache-aside com double-checked locking distribuído

O padrão `remember` é a forma canônica de consumir cache nesta spec. O fluxo é:

```
remember(chave, ttl, fn)
  1. Lê do cache
  2. Se hit → retorna valor desserializado
  3. Se miss → tenta adquirir lock distribuído (SET NX PX)
  4. Se lock adquirido → executa fn(), grava resultado, libera lock
  5. Se lock não adquirido → aguarda e relê do cache
     5a. Se hit após espera → retorna valor (produtor gravou com sucesso)
     5b. Se ainda null → executa fn() diretamente (produtor caiu antes de gravar)
  6. Retorna valor
```

O double-checked locking evita que múltiplas instâncias executem `fn()` simultaneamente para a mesma chave após um miss concorrente — apenas uma instância grava; as demais aguardam e leem o resultado. Se o produtor cair antes de gravar (lock expirou sem gravação), as instâncias que aguardavam executam `fn()` diretamente como fallback.

> Se `CACHE_ENABLED=false`, o `remember` executa `fn()` diretamente e retorna sem gravar ou ler do cache.

---

## Operações base

| Operação | Comportamento |
|----------|--------------|
| `get(chave)` | Retorna o valor desserializado ou `null` se ausente |
| `set(chave, valor, ttl)` | Grava valor serializado com TTL em segundos |
| `delete(chave)` | Remove uma chave exata |
| `deletePattern(padrão)` | Remove todas as chaves que correspondem ao padrão — usa varredura incremental (nunca bloqueante) |
| `remember(chave, ttl, fn)` | Cache-aside com double-checked locking — ver padrão principal acima |

---

## Namespacing de chaves

As chaves seguem namespacing hierárquico separado por `:`:

```
{prefixo}:{recurso}:{identificador}
{prefixo}:{recurso}:{identificador}:{sub-recurso}:{identificador}
```

- O prefixo é configurado globalmente e aplicado a todas as chaves automaticamente pela camada de cache
- Identificadores compostos usam `:` como separador
- Parâmetros variáveis de listagem (filtros, paginação) são representados por um hash determinístico dos parâmetros como sufixo da chave — garante que listas com parâmetros distintos tenham chaves distintas sem tornar a chave legível ilegível

**Exemplos de padrão:**

```
{prefixo}:users:{userId}
{prefixo}:users:{userId}:orders:{orderId}
{prefixo}:products:list:{hashDosFiltros}
```

---

## TTLs

- Cada entrada tem um TTL em segundos configurado no momento da gravação
- TTLs são definidos por recurso e configuráveis via variável de ambiente
- Um multiplicador global (`CACHE_TTL_MULTIPLIER`) é aplicado a todos os TTLs — permite aumentar ou reduzir o tempo de cache de toda a aplicação sem alterar cada definição individualmente
- TTL efetivo = arredondar(TTL do recurso × multiplicador global) — sempre um inteiro; TTLs fracionários causam comportamento indefinido no backend de cache

---

## Invalidação

A invalidação ocorre em dois níveis:

**Por chave exata** — remove uma entrada específica.

**Por padrão** — remove todas as chaves que correspondem a um padrão (ex: todas as entradas de um recurso). A varredura é feita de forma incremental para não bloquear o Redis.

Após a remoção das chaves, o sistema publica um evento de invalidação no canal de Pub/Sub do recurso. O **recurso** é derivado do padrão como o primeiro segmento antes do primeiro `:` — por exemplo, `users:42:*` → recurso `users`; `products:list:*` → recurso `products`. Isso garante que toda invalidação relativa a um mesmo tipo de dado chegue ao mesmo canal, independentemente da granularidade do padrão usado.

> A publicação do evento é melhor-esforço: a invalidação das chaves é a operação principal. Falha na publicação não deve impedir nem desfazer a remoção das chaves.

---

## Aquecimento de cache (Cache Warming)

O aquecimento pré-carrega no Redis, durante o boot da aplicação, recursos que seriam custosos de computar na primeira requisição após um deploy.

Comportamento:
- Executa após a aplicação estar completamente inicializada
- Cada recurso registrado para aquecimento fornece sua própria função de carga
- Erros de aquecimento são logados mas não impedem a inicialização — a aplicação sobe mesmo que o warming falhe
- O aquecimento usa o mesmo mecanismo de `set` do cache regular, com o TTL configurado para cada recurso

---

## Flag de desabilitação

Quando `CACHE_ENABLED=false`:
- `get` retorna sempre `null`
- `set`, `delete` e `deletePattern` são no-op
- `remember` executa `fn()` diretamente e retorna o resultado sem interagir com o Redis
- O aquecimento é ignorado
- Útil para testes e ambientes de desenvolvimento onde cache pode obscurecer bugs

---

## Edge cases

| Situação | Comportamento esperado |
|----------|----------------------|
| Miss concorrente na mesma chave em múltiplas instâncias | Apenas a instância que adquiriu o lock executa `fn()`; as demais aguardam e leem o valor gravado |
| Lock expira antes do produtor gravar (produtor caiu) | Instâncias que aguardavam releem e encontram `null`; executam `fn()` diretamente como fallback |
| `fn()` lança exceção durante `remember` | Lock é liberado; exceção é propagada para o caller; nada é gravado no cache |
| Redis indisponível com `CACHE_ENABLED=true` | Operações de cache falham silenciosamente; `remember` executa `fn()` e retorna o resultado sem cache |
| `deletePattern` em padrão que corresponde a milhares de chaves | Remoção ocorre de forma incremental sem bloquear o Redis; pode levar mais tempo que um delete simples |
| Aquecimento falha para um dos recursos | Log do erro; demais recursos continuam sendo aquecidos; aplicação sobe normalmente |
| `CACHE_ENABLED=false` | Todas as operações de cache são no-op; `remember` executa `fn()` diretamente |

---

## Regras de implementação

- O prefixo de chaves é sempre aplicado pela camada de cache — o caller nunca o inclui manualmente
- Chaves nunca são construídas com concatenação livre — o padrão de namespacing hierárquico é obrigatório
- Chaves de lock seguem o padrão `{prefixo}:lock:{chave}` — o segmento `lock` garante que nunca colidam com chaves de dados
- `deletePattern` nunca usa comandos bloqueantes no Redis (ex: `KEYS`) — sempre varredura incremental
- TTL efetivo é sempre um inteiro — o resultado de TTL × multiplicador é arredondado antes de ser gravado
- Erros de cache nunca propagam para o fluxo principal da aplicação quando há fallback disponível (ex: `remember` com `fn()`)
- O multiplicador global de TTL é aplicado pela camada de cache, não pelo caller

---

## O que esta spec NÃO cobre

- Notificação em tempo real para clientes via WebSocket → escopo de real-time/WebSocket
- Cache de sessão → escopo de autenticação
- Cache de respostas HTTP em nível de proxy → escopo de infraestrutura
- Implementação dos comandos Redis específicos → ver `setup/nestjs/nestjs-cache`
