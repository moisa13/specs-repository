# Behavior — Processamento Assíncrono com Fila

---

## Objetivo

Garantir que jobs sejam produzidos, enfileirados e consumidos de forma confiável, com retry automático em caso de falha e encaminhamento para dead letter queue após esgotar as tentativas.

---

## Fluxo principal

1. O produtor cria um job com tipo, payload e opções de enfileiramento
2. O job é persistido na fila com status `waiting`
3. Um worker disponível assume o job; o status muda para `active`
4. O worker executa o processamento
5. Se bem-sucedido, o status muda para `completed`
6. Se falhar, o sistema aplica o comportamento de retry (ver seção abaixo)

---

## Fluxo de retry

1. O job falha durante o processamento
2. O sistema verifica se o número de tentativas realizadas é menor que o máximo configurado
3. Se sim, o job transita para o estado `delayed` pelo período de backoff configurado (exponencial ou fixo); ao expirar, retorna para `waiting` e aguarda um worker disponível
4. Se o número de tentativas for igual ao máximo, o job é movido para a dead letter queue com status `failed` e a razão da falha registrada

---

## Regras de negócio

- Todo job deve ter um tipo (`jobType`) que identifica qual processamento será executado
- O payload do job deve ser serializável (JSON)
- O número máximo de tentativas padrão é 3
- O delay entre tentativas depende do tipo de backoff configurado:
  - `exponential` (padrão): delay cresce a cada retry — `baseDelay * 2^(retry - 1)`; delay base padrão é 1000ms
  - `fixed`: delay constante igual a `backoff.delay` em todas as tentativas, independente do número de retries
- Cada fila tem um limite de workers simultâneos (concorrência) configurável; o padrão é 1
- Jobs com prioridade mais alta são processados antes de jobs com prioridade mais baixa na mesma fila
- Jobs agendados com delay só entram no estado `waiting` após o delay expirar
- O produtor não aguarda a conclusão do job — a produção é fire-and-forget

---

## Edge cases

| Situação | Comportamento esperado |
|----------|----------------------|
| Worker falha no meio do processamento (crash) | O job volta para `waiting` e conta como tentativa falha |
| Job produzido com payload inválido (não serializável) | Rejeitado imediatamente pelo produtor; não entra na fila |
| Fila indisponível no momento da produção | O produtor lança erro; o chamador é responsável por tratar ou retentar |
| Job duplicado com o mesmo ID | Ignorado se já existe na fila; não substitui o job existente |
| Todos os workers ocupados | O job permanece em `waiting` até um worker ficar disponível |
| Job na DLQ reprocessado manualmente | Entra como novo job; contagem de tentativas reinicia do zero |
| Timeout durante o processamento | O job é considerado falho; o comportamento de retry se aplica |
| Job sem consumidor registrado para o tipo | Permanece em `waiting`; não é descartado automaticamente |

---

## Restrições

- Nunca processar o mesmo job mais de uma vez simultaneamente (sem lock distribuído)
- Nunca descartar um job silenciosamente — falhas devem ser registradas e o job movido para DLQ
- Nunca alterar o payload do job após o enfileiramento
- Nunca remover um job da DLQ automaticamente — a remoção é sempre manual ou via política explícita
- Nunca bloquear o produtor aguardando a conclusão do job

---

## O que esta spec NÃO cobre

- Configuração de conexão com o broker → ver spec de implementação (ex: `setup/nestjs/nestjs-queue`)
- Estrutura de módulos e código da implementação → ver spec de implementação
- Monitoramento, métricas e alertas de filas → ver `observability/nestjs-metrics` (a criar)
- Comunicação pub/sub entre serviços → escopo distinto
