# Decisions — Registro Estruturado de Logs

> Decisões que podem parecer questionáveis sem contexto — evita que implementadores "melhorem" comportamentos deliberadamente escolhidos.

---

## ADR-001 — Arquivo separado para logs de erro

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Em operações de on-call, engenheiros precisam localizar rapidamente erros críticos sem percorrer o arquivo de log completo, que mistura todos os níveis.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Arquivo único com todos os níveis | Menos arquivos para gerenciar | Dificulta triagem rápida de erros |
| Arquivo separado para erros — **escolhida** | Triagem imediata sem filtro manual | Uso de disco maior para erros (gravados em dois arquivos) |

### Decisão
Manter dois transportes de arquivo ativos: um para todos os níveis e um exclusivo para `error`.

### Consequências
- Erros aparecem em dois arquivos (uso de disco dobrado para entradas `error`)
- On-call pode abrir `YYYY-MM-DD-error.log` diretamente sem ferramentas de filtro

---

## ADR-002 — Profundidade de sanitização limitada a 5 níveis

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
A sanitização precisa ser recursiva para cobrir objetos aninhados, mas sem limite de profundidade pode causar problemas de performance ou loops em estruturas circulares.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Profundidade ilimitada | Cobertura total | Risco de loop em estruturas circulares; custo imprevisível |
| Profundidade fixa de 5 níveis — **escolhida** | Previsível; suficiente para payloads bem estruturados | Campos sensíveis além do nível 5 não são redijidos |

### Decisão
Limitar a sanitização recursiva a 5 níveis de profundidade.

### Consequências
- Campos sensíveis aninhados além do 5º nível não são redijidos (risco residual aceito)
- Nenhum risco de loop infinito ou degradação de performance por sanitização

---

## ADR-003 — Exceções 4xx registradas como warn, não error

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Erros 4xx representam comportamento incorreto do cliente — entrada inválida, recurso não encontrado, sem permissão. Não indicam falha no servidor. Tratá-los como `error` geraria ruído em alertas e dashboards de on-call.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Tudo como `error` | Simples | Polui alertas com erros esperados do cliente |
| 4xx como `warn`, 5xx como `error` — **escolhida** | Alertas refletem falhas reais do servidor | Requer classificação por faixa de status |

### Decisão
4xx → `warn` sem stack trace. 5xx → `error` com stack trace completo.

### Consequências
- Alertas baseados em `error` só disparam para falhas reais do servidor
- Logs de 4xx ainda são visíveis em `warn` para diagnóstico de abuso ou bugs de cliente

---

## ADR-004 — correlationId ausente omitido do JSON, não serializado como null

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Quando não há contexto de correlação ativo (job assíncrono, chamada direta), o `correlationId` não está disponível. Duas opções: omitir o campo do objeto JSON ou incluí-lo com valor `null`.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Serializar como `null` | Campo sempre presente; schema previsível | Agregadores precisam tratar null explicitamente; ruído em queries |
| Omitir o campo — **escolhida** | JSON mais limpo; ausência é semanticamente distinta de null | Consumidores precisam verificar existência do campo, não apenas o valor |

### Decisão
Quando o `correlationId` não estiver disponível, o campo é omitido da entrada de log. Nunca serializado como `null`.

### Consequências
- Entradas de log sem contexto de correlação têm JSON menor
- Consumidores devem verificar a presença do campo, não apenas seu valor
- Filtros como `correlationId IS NOT NULL` em agregadores funcionam corretamente
