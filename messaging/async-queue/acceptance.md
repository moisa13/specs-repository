# Acceptance Criteria — Processamento Assíncrono com Fila

> Formato: **Dado** [contexto] **Quando** [ação] **Então** [resultado esperado]

---

## Fluxo principal

**AC-01 — Enfileiramento bem-sucedido**
- **Dado** que a fila está registrada e o broker está disponível
- **Quando** o produtor enfileira um job com tipo e payload válidos
- **Então** o job é persistido com status `waiting` e o produtor recebe o `jobId` como confirmação

**AC-02 — Processamento bem-sucedido**
- **Dado** que há um job com status `waiting` na fila e um worker disponível
- **Quando** o worker assume o job
- **Então** o status muda para `active` durante o processamento e para `completed` ao terminar

**AC-03 — Job com delay**
- **Dado** que o produtor enfileira um job com `options.delay > 0`
- **Quando** o job é criado
- **Então** o status inicial é `delayed` e o job só entra em `waiting` após o delay expirar

---

## Cenários de falha

**AC-04 — Retry automático após falha**
- **Dado** que um job falhou durante o processamento e o número de tentativas realizadas é menor que o máximo configurado
- **Quando** o worker lança um erro
- **Então** o job transita para `delayed` pelo período de backoff configurado (exponencial ou fixo, conforme `options.backoff.type`) e retorna para `waiting` ao expirar

**AC-05 — Encaminhamento para DLQ após esgotar tentativas**
- **Dado** que um job falhou e o número de tentativas realizadas atingiu o máximo configurado
- **Quando** o worker lança um erro na última tentativa
- **Então** o job é movido para a dead letter queue com status `failed` e a razão da falha registrada em `failReason`

**AC-06 — Fila não registrada**
- **Dado** que o produtor tenta enfileirar um job em uma fila inexistente
- **Quando** a chamada de produção é feita
- **Então** o erro `QUEUE_NOT_FOUND` é retornado e nenhum job é criado

**AC-07 — Payload inválido**
- **Dado** que o produtor fornece um payload não serializável em JSON
- **Quando** a chamada de produção é feita
- **Então** o erro `INVALID_PAYLOAD` é retornado e nenhum job é criado

**AC-08 — Broker indisponível**
- **Dado** que o broker de filas está inacessível
- **Quando** o produtor tenta enfileirar um job
- **Então** o erro `QUEUE_UNAVAILABLE` é retornado; nenhum job é enfileirado silenciosamente

---

## Edge cases

**AC-09 — Prioridade respeitada**
- **Dado** que há múltiplos jobs em `waiting` com prioridades diferentes na mesma fila
- **Quando** um worker fica disponível
- **Então** o job com maior prioridade é assumido primeiro

**AC-10 — Job duplicado ignorado**
- **Dado** que já existe um job com determinado `jobId` na fila
- **Quando** o produtor tenta enfileirar outro job com o mesmo `jobId`
- **Então** o segundo job é ignorado e o primeiro permanece inalterado

**AC-11 — Reprocessamento de job na DLQ**
- **Dado** que existe um job com status `failed` na dead letter queue
- **Quando** o job é reprocessado manualmente
- **Então** ele entra na fila como novo job com contagem de tentativas zerada

**AC-12 — Sem consumidor registrado**
- **Dado** que um job do tipo `X` é enfileirado e nenhum worker está registrado para o tipo `X`
- **Quando** o job fica em `waiting`
- **Então** o job permanece em `waiting` indefinidamente e não é descartado

---

## O que caracteriza uma implementação incorreta

- O produtor bloqueia aguardando a conclusão do job
- Um job falho é descartado sem ser movido para a DLQ
- O mesmo job é processado por dois workers simultaneamente
- O payload do job é alterado após o enfileiramento
- Jobs na DLQ são removidos automaticamente sem política explícita configurada

---

## Cobertura mínima de testes

- [ ] Fluxo principal completo (produção → consumo → conclusão)
- [ ] Retry com backoff configurável (exponencial e fixo)
- [ ] Movimentação para DLQ após esgotar tentativas
- [ ] Rejeição de payload inválido
- [ ] Comportamento com broker indisponível
