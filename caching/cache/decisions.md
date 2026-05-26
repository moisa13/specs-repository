# Decisions — Caching — Cache de Dados

> Decisões que podem parecer questionáveis sem contexto — evita que implementadores "melhorem" comportamentos deliberadamente escolhidos.

---

## ADR-001 — Cache-aside como padrão único de leitura

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Existem múltiplos padrões de integração com cache: cache-aside (lazy loading), read-through, write-through e write-behind. É necessário escolher um padrão principal para garantir consistência de implementação.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Read-through | Cache transparente para o caller | Requer middleware ou proxy de cache; menos controle |
| Write-through | Dados sempre atualizados no cache | Latência na escrita; complexidade de sincronização |
| Cache-aside — **escolhida** | Controle total no caller; falha do cache não afeta a operação principal | Miss na primeira requisição |

### Decisão
Cache-aside implementado via `remember()`. O caller define a chave, o TTL e a função produtora. A camada de cache decide se lê do Redis ou executa a função.

### Consequências
- Cold start após deploy é esperado — mitigado pelo `CacheWarmingService`
- A lógica de cache fica no código de aplicação, não em middleware externo

---

## ADR-002 — Double-checked locking distribuído via SET NX PX

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Em ambientes com múltiplas instâncias, um miss concorrente na mesma chave causaria thundering herd: todas as instâncias executariam `fn()` simultaneamente, anulando o benefício do cache e sobrecarregando o banco.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Sem controle de concorrência | Simples | Thundering herd em misses simultâneos |
| Lock via SET NX PX — **escolhida** | Atômico no Redis; sem dependência adicional | Lock deve ter TTL para evitar deadlock se a instância cair |

### Decisão
`remember()` usa SET NX PX para adquirir um lock distribuído antes de executar `fn()`. Instâncias que não adquirem o lock aguardam e releem o cache após a gravação pelo produtor.

### Consequências
- Apenas uma instância executa `fn()` por miss, independentemente do número de instâncias
- O lock tem TTL curto — se a instância produtora cair, o lock expira e outra instância pode assumir

---

## ADR-003 — deletePattern usa varredura incremental (SCAN), nunca KEYS

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Remoção de múltiplas chaves por padrão é necessária para invalidar recursos inteiros. O comando `KEYS` do Redis retorna todas as chaves de uma vez e bloqueia o servidor durante a execução — inaceitável em produção.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| KEYS + DEL | Simples de implementar | Bloqueia o Redis; inaceitável em produção com volume de chaves |
| SCAN + DEL em lotes — **escolhida** | Não bloqueante; seguro para qualquer volume | Mais lento que KEYS; não atômico |

### Decisão
`deletePattern` usa SCAN iterativo com cursor para encontrar as chaves em lotes e DEL para removê-las. O uso de KEYS é proibido neste contexto.

### Consequências
- A remoção pode levar mais tempo em padrões que correspondem a muitas chaves
- O Redis permanece responsivo durante a operação

---

## ADR-004 — Multiplicador global de TTL

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Em produção, pode ser necessário ajustar o comportamento do cache sem modificar cada definição de TTL individualmente — por exemplo, reduzir TTLs durante um incidente ou aumentá-los em ambiente de carga.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Alterar cada TTL individualmente | Controle granular | Operacionalmente custoso; exige redeploy |
| Multiplicador global via variável de ambiente — **escolhida** | Ajuste operacional sem redeploy | TTLs resultantes podem ser fracionários — devem ser arredondados |

### Decisão
`CACHE_TTL_MULTIPLIER` é aplicado a todos os TTLs pela camada de cache antes de gravar. TTL efetivo = TTL do recurso × multiplicador, arredondado para o inteiro mais próximo.

### Consequências
- Operadores podem ajustar o comportamento do cache sem redeploy
- Multiplicador `0` resulta em TTL `0` — entradas expiram imediatamente; equivale a desabilitar o cache com persistência zero
