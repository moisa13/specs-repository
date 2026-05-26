# Acceptance Criteria — NestJS Filas Assíncronas (BullMQ + Redis)

> Formato: **Dado** [contexto] **Quando** [ação] **Então** [resultado esperado]

---

## Fluxo principal

**AC-01 — Inicialização com Redis disponível**
- **Dado** que as variáveis `REDIS_HOST` e `REDIS_PORT` estão configuradas e o Redis está acessível
- **Quando** a aplicação inicia
- **Então** o `QueueModule` conecta ao Redis, registra todas as filas e o `QueueService` está disponível para injeção

**AC-02 — Produção de job bem-sucedida**
- **Dado** que o `QueueService` está disponível e a fila `notifications` está registrada no `QueueModule`
- **Quando** um service chama `QueueService.add('notifications', 'send-email', payload)`
- **Então** o job é enfileirado e o método retorna `{ jobId, queueName: 'notifications', status: 'waiting' }`

**AC-03 — Consumo de job pelo Processor**
- **Dado** que existe um job `send-email` na fila `notifications` e o `SendEmailProcessor` está registrado
- **Quando** o BullMQ entrega o job ao Processor
- **Então** o método `process(job)` do Processor é chamado, o job é despachado pelo `job.name` e marcado como `completed` ao término sem erro

**AC-04 — Job com delay**
- **Dado** que um job é enfileirado com `options.delay = 5000`
- **Quando** o `QueueService.add` retorna
- **Então** o status retornado é `delayed` e o job só entra em `waiting` após 5 segundos

---

## Centralização

**AC-05 — Registro de fila exclusivamente no QueueModule**
- **Dado** que um módulo de feature precisa de uma nova fila
- **Quando** a fila é adicionada
- **Então** `BullModule.registerQueue` é chamado em `queue.module.ts` (registro canônico) e no módulo de feature que declara o Processor; o nome da fila é adicionado a `queue.constants.ts`

**AC-06 — Produção exclusivamente via QueueService**
- **Dado** que qualquer service de domínio precisa enfileirar um job
- **Quando** o service enfileira
- **Então** ele utiliza `QueueService.add()` — não há `@InjectQueue` fora de `queue.service.ts`

**AC-07 — Processor declarado no módulo de feature**
- **Dado** que uma feature precisa processar jobs de uma fila
- **Quando** o Processor é criado
- **Então** ele está em `src/[feature]/processors/` e declarado no módulo da feature, não no `QueueModule`

---

## Cenários de falha

**AC-08 — Fila não registrada**
- **Dado** que `QueueService.add` é chamado com `queueName: 'fila-inexistente'`
- **Quando** a chamada é feita
- **Então** `QueueNotFoundException` é lançada e nenhum job é criado

**AC-09 — Payload inválido**
- **Dado** que `QueueService.add` é chamado com um payload não serializável em JSON
- **Quando** a chamada é feita
- **Então** `InvalidPayloadException` é lançada antes de qualquer interação com o Redis

**AC-10 — Redis indisponível na inicialização**
- **Dado** que o Redis está inacessível quando a aplicação inicia
- **Quando** o NestJS tenta iniciar
- **Então** a aplicação falha com erro explícito de conexão Redis — não sobe silenciosamente

**AC-11 — Redis cai em runtime**
- **Dado** que a aplicação está rodando e o Redis fica indisponível
- **Quando** `QueueService.add` é chamado
- **Então** `QueueUnavailableException` é lançada; os workers pausam e tentam reconectar automaticamente

---

## Edge cases

**AC-12 — Dois Processors para a mesma fila**
- **Dado** que dois módulos de feature registram um Processor para a mesma fila
- **Quando** a aplicação inicia
- **Então** a situação viola a restrição da spec e resulta em comportamento indefinido — não há detecção automática; a prevenção é responsabilidade do desenvolvedor

**AC-13 — Referência a QUEUE_NAMES**
- **Dado** que o código referencia um nome de fila
- **Quando** auditado
- **Então** nenhuma string literal de nome de fila existe fora de `queue.constants.ts`

**AC-14 — Rota /queues protegida por Basic Auth**
- **Dado** que a aplicação está rodando
- **Quando** uma requisição é feita a `/queues` sem credenciais ou com credenciais inválidas
- **Então** a resposta é `401 Unauthorized` e o acesso ao painel é negado

**AC-15 — Acesso autenticado ao painel**
- **Dado** que a aplicação está rodando e `BULL_BOARD_USER` / `BULL_BOARD_PASSWORD` estão configurados
- **Quando** uma requisição é feita a `/queues` com as credenciais corretas via Basic Auth
- **Então** o painel é acessível e exibe as filas registradas

---

## O que caracteriza uma implementação incorreta

- `BullModule.registerQueue` chamado em módulos que não sejam `queue.module.ts` ou módulos de feature com Processor
- `@InjectQueue` usado fora de `queue.service.ts`
- Nomes de fila definidos como strings literais no código
- Processor declarado diretamente no `QueueModule`
- Jobs completados acumulando no Redis por falta de `removeOnComplete`
- Rota `/queues` acessível sem autenticação

---

## Cobertura mínima de testes

- [ ] `QueueService.add` enfileira o job e retorna `jobId`
- [ ] `QueueService.add` com fila não registrada lança `QueueNotFoundException`
- [ ] `QueueService.add` com payload inválido lança `InvalidPayloadException`
- [ ] Processor executa o método correto para o `jobType` recebido
- [ ] Retry é aplicado quando o Processor lança um erro
