# Decisions — Processamento Assíncrono com Fila

> Decisões que podem parecer questionáveis sem contexto — evita que implementadores "melhorem" comportamentos deliberadamente escolhidos.

---

## ADR-001 — Produção fire-and-forget

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
O produtor poderia aguardar a conclusão do job (await) ou retornar imediatamente após o enfileiramento. A escolha afeta o modelo de uso e o acoplamento entre produtor e consumidor.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Aguardar conclusão (blocking) | Simplicidade de uso; erro propagado diretamente | Acopla produtor ao tempo de processamento; bloqueia o ciclo de request |
| Fire-and-forget (escolhida) | Desacoplamento total; produtor não é afetado por lentidão do consumidor | Requer rastreamento separado para saber o resultado do job |

### Decisão
O produtor retorna imediatamente após o enfileiramento, sem aguardar o processamento. O resultado do job é rastreado via status no broker ou via callback/evento, conforme necessidade do domínio.

### Consequências
- O chamador não pode assumir que o job foi processado após receber o `jobId`
- Specs de domínio que precisam do resultado do processamento devem definir um mecanismo de notificação separado (ex: webhook, evento, polling de status)

---

## ADR-002 — Backoff exponencial como padrão de retry

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Falhas transitórias (rede, banco sobrecarregado) tendem a se resolver com o tempo. Retransmitir imediatamente agrava o problema; esperar um tempo fixo é subótimo para falhas rápidas.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Retry imediato | Recuperação rápida em falhas muito curtas | Amplifica carga em momentos de instabilidade |
| Delay fixo | Simples de configurar | Não adapta o intervalo ao número de tentativas |
| Backoff exponencial (escolhida) | Reduz carga progressivamente; padrão amplamente adotado | Aumenta o tempo total de recovery em falhas longas |

### Decisão
Backoff exponencial com delay base de 1000ms: `1s → 2s`. Máximo de 3 tentativas por padrão (2 retries).

### Consequências
- O tempo máximo antes de ir para DLQ é de ~3 segundos com os padrões (1s + 2s): 3 tentativas geram 2 intervalos de retry
- Specs de domínio podem sobrescrever `attempts` e `backoff.delay` conforme a criticidade do job

---

## ADR-003 — Dead letter queue obrigatória

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Jobs que falham definitivamente poderiam ser simplesmente descartados ou mantidos na fila original com status `failed`.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Descartar job silenciosamente | Simples; sem acúmulo | Perda de dados; impossível auditar ou reprocessar |
| Manter na fila original | Visibilidade sem infraestrutura extra | Polui a fila principal; afeta métricas |
| Dead letter queue separada (escolhida) | Isolamento; permite auditoria e reprocessamento manual | Requer infraestrutura e monitoramento adicionais |

### Decisão
Jobs que esgotam as tentativas são movidos para uma DLQ dedicada. A DLQ não remove jobs automaticamente — toda remoção é manual ou via política explícita configurada.

### Consequências
- A DLQ deve ser monitorada para detectar falhas recorrentes
- O reprocessamento manual reinicia a contagem de tentativas
- Specs de implementação devem garantir que a DLQ seja configurada junto com a fila principal

---

## ADR-004 — Spec agnóstica de tecnologia

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
O repositório tem specs de implementação específicas por stack. Esta spec poderia já ser escrita para BullMQ/Redis, mas isso limitaria o reuso para outros contextos.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Spec acoplada a BullMQ | Mais direta; menos abstração | Não reutilizável em outros contextos |
| Spec agnóstica (escolhida) | Reutilizável; separa comportamento de implementação | Requer uma segunda spec para a implementação concreta |

### Decisão
Esta spec descreve apenas comportamento. A implementação em NestJS com BullMQ é coberta por `setup/nestjs/nestjs-queue`, que referencia esta spec.

### Consequências
- Toda spec de domínio que precisa de fila referencia esta spec para o comportamento e a spec de implementação para o "como"
- Mudanças de tecnologia de fila afetam apenas a spec de implementação, não esta
