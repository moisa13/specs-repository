# Acceptance Criteria — Caching — Cache de Dados

> Formato: **Dado** [contexto] **Quando** [ação] **Então** [resultado esperado]

---

## Padrão remember

**AC-01 — Hit retorna valor do cache sem executar fn()**
- **Dado** que a chave `users:42` existe no cache com valor `{ id: 42, name: "João" }`
- **Quando** `remember('users:42', 300, fn)` é chamado
- **Então** retorna `{ id: 42, name: "João" }` sem executar `fn()`

**AC-02 — Miss executa fn() e grava no cache**
- **Dado** que a chave `users:42` não existe no cache
- **Quando** `remember('users:42', 300, fn)` é chamado
- **Então** executa `fn()`, grava o resultado com TTL 300 × multiplicador e retorna o valor

**AC-03 — Double-checked locking previne thundering herd**
- **Dado** que 10 instâncias chamam `remember('users:42', 300, fn)` simultaneamente com miss
- **Quando** todas tentam adquirir o lock distribuído
- **Então** apenas uma executa `fn()`; as demais aguardam e leem o valor gravado

**AC-04 — Lock expira sem gravação: instâncias aguardando executam fn() diretamente**
- **Dado** que uma instância adquiriu o lock e caiu antes de gravar
- **Quando** as instâncias que aguardavam releem o cache após a espera
- **Então** encontram `null`; executam `fn()` diretamente como fallback e retornam o valor

**AC-05 — Exceção em fn() libera o lock e propaga o erro**
- **Dado** que `fn()` lança uma exceção
- **Quando** `remember` executa `fn()`
- **Então** o lock distribuído é liberado e a exceção é propagada ao caller sem gravar nada no cache

---

## Flag de desabilitação

**AC-06 — CACHE_ENABLED=false executa fn() diretamente**
- **Dado** que `CACHE_ENABLED=false`
- **Quando** `remember('users:42', 300, fn)` é chamado
- **Então** executa `fn()` e retorna o resultado sem ler ou gravar no Redis

**AC-07 — CACHE_ENABLED=false torna get, set e delete no-op**
- **Dado** que `CACHE_ENABLED=false`
- **Quando** `get`, `set` ou `delete` são chamados
- **Então** `get` retorna `null`; `set` e `delete` retornam sem interagir com o Redis

**AC-08 — CACHE_ENABLED=false torna deletePattern no-op**
- **Dado** que `CACHE_ENABLED=false`
- **Quando** `deletePattern('users:*')` é chamado
- **Então** nenhuma chave é varrida ou removida; nenhum evento é publicado no canal Pub/Sub

---

## Namespacing e prefixo

**AC-09 — Prefixo aplicado automaticamente**
- **Dado** que `CACHE_KEY_PREFIX=myapp` e o caller passa a chave `users:42`
- **Quando** qualquer operação de cache é executada
- **Então** a chave gravada no Redis é `myapp:users:42`; o caller nunca inclui o prefixo

---

## TTL e multiplicador

**AC-10 — TTL efetivo aplica o multiplicador global e é arredondado**
- **Dado** que `CACHE_TTL_MULTIPLIER=2` e o caller passa `ttl=300`
- **Quando** `remember` ou `set` grava no Redis
- **Então** o TTL gravado é 600 segundos (arredondado para inteiro)

---

## Invalidação

**AC-11 — delete remove chave exata e publica evento**
- **Dado** que a chave `users:42` existe no Redis
- **Quando** `delete('users:42')` é chamado
- **Então** a chave é removida e um evento é publicado no canal `cache-events:users` (primeiro segmento de `users:42`)

**AC-12 — deletePattern remove todas as chaves correspondentes sem bloquear**
- **Dado** que existem 10.000 chaves com padrão `users:*` no Redis
- **Quando** `deletePattern('users:*')` é chamado
- **Então** todas as 10.000 chaves são removidas via varredura incremental; o Redis permanece responsivo

**AC-13 — Evento de invalidação publicado após remoção com recurso derivado do padrão**
- **Dado** que `deletePattern('users:42:*')` remove chaves com sucesso
- **Quando** a remoção é concluída
- **Então** um evento é publicado no canal `cache-events:users` (primeiro segmento de `users:42:*`) com as chaves removidas e timestamp

**AC-14 — Falha na publicação do evento não desfaz a invalidação**
- **Dado** que as chaves foram removidas e a publicação do evento falha
- **Quando** o erro de Pub/Sub ocorre
- **Então** as chaves permanecem removidas; o erro é logado; nenhuma exceção é propagada ao caller

---

## Redis indisponível

**AC-15 — Redis indisponível não propaga erro ao caller via remember**
- **Dado** que o Redis está indisponível
- **Quando** `remember('users:42', 300, fn)` é chamado
- **Então** executa `fn()` e retorna o resultado como fallback; nenhuma exceção é propagada ao caller

---

## Aquecimento de cache

**AC-16 — Falha no warming de um recurso não impede a inicialização**
- **Dado** que o aquecimento de um recurso falha com erro
- **Quando** a aplicação inicia
- **Então** o erro é logado; os demais recursos continuam sendo aquecidos; a aplicação sobe normalmente

---

## O que caracteriza uma implementação incorreta

- `fn()` executada mais de uma vez por miss concorrente (thundering herd)
- Uso de `KEYS` no Redis para `deletePattern` — bloqueia o servidor
- Prefixo de chave incluído pelo caller em vez de aplicado pela camada de cache
- Erros de cache propagando para o fluxo principal quando `fn()` poderia ser o fallback
- `CACHE_ENABLED=false` ainda interagindo com o Redis
