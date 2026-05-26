# Behavior — NestJS Filas Assíncronas (BullMQ + Redis)

---

## Objetivo

Configurar o BullMQ em um projeto NestJS de forma que todas as filas sejam registradas e gerenciadas em um único `QueueModule` centralizado, com um `QueueService` como único ponto de produção de jobs.

---

## Estado inicial (bootstrap)

Ao aplicar esta spec pela primeira vez, `QUEUE_NAMES` é um objeto vazio e o `QueueService` não tem nenhum `@InjectQueue`. O projeto compila e sobe normalmente nesse estado — não há filas registradas, não há workers ativos, e `QueueService.add` nunca é chamado porque nenhuma feature usa filas ainda. A primeira fila é adicionada junto com o primeiro módulo de feature que a requer: `QUEUE_NAMES` recebe a entrada, `QueueModule` inclui o `BullModule.registerQueue` correspondente, e o `QueueService` ganha o `@InjectQueue` e a entrada no `Map`.

---

## Fluxo de configuração (setup)

1. As variáveis de ambiente Redis e opções de job são carregadas via `ConfigService`
2. O `QueueModule` registra a conexão Redis uma única vez via `BullModule.forRootAsync`
3. O `QueueModule` registra todas as filas da aplicação via `BullModule.registerQueue`
4. O `QueueModule` exporta o `QueueService` para que módulos que o importam possam injetá-lo
5. Cada módulo de feature importa o `QueueModule` e registra seus Processors

## Fluxo de produção de job

1. Um service de domínio chama `QueueService.add(queueName, jobType, payload, options?)`
2. O `QueueService` localiza a fila pelo nome e enfileira o job via BullMQ
3. Retorna o `jobId` como confirmação; não aguarda o processamento

## Fluxo de consumo de job

1. O BullMQ entrega o job ao Processor registrado para aquela fila
2. O Processor recebe o job no método `process(job)` herdado de `WorkerHost` e despacha pelo `job.name`
3. Se o método concluir sem erro, o job é marcado como `completed`
4. Se o método lançar um erro, o BullMQ aplica retry conforme as opções do job
5. Após esgotar as tentativas, o job é movido para a DLQ da fila

---

## Regras de negócio

- O `QueueModule` faz o registro canônico de todas as filas via `BullModule.registerQueue` — é o único ponto que disponibiliza as filas ao `QueueService`
- Módulos de feature que declaram um Processor devem também chamar `BullModule.registerQueue` para a sua fila, como requisito do `@nestjs/bullmq` para que o worker conecte — isso não substitui o registro canônico no `QueueModule`
- O `QueueService` é o único provider que acessa `InjectQueue` — services de domínio nunca injetam `Queue` diretamente
- Toda fila nova deve ser registrada no `QueueModule` e ter seu nome adicionado ao arquivo `queue.constants.ts`
- Todo Processor deve ser declarado no módulo de feature correspondente e importar `QueueModule`
- Um Processor deve estender `WorkerHost` e implementar o método `process(job: Job)`, despachando pelo `job.name` para o handler correspondente ao `jobType`
- Filas sem Processor registrado não lançam erro na inicialização — jobs ficam em `waiting` até um Processor ser registrado
- A conexão Redis é compartilhada por todas as filas; não se cria uma conexão por fila
- A concorrência de workers por fila é configurada no segundo argumento do decorator `@Processor`: `@Processor(QUEUE_NAMES.FILA, { concurrency: N })`; o padrão do BullMQ é `1` worker simultâneo por fila — alterar apenas quando houver necessidade explícita de paralelismo

---

## Edge cases

| Situação | Comportamento esperado |
|----------|----------------------|
| `QueueService.add` chamado com nome de fila não registrada | Lança `QueueNotFoundException` com o nome da fila |
| Redis indisponível na inicialização | A aplicação falha ao iniciar com erro explícito de conexão |
| Redis cai após a aplicação estar rodando | `QueueService.add` lança `QueueUnavailableException`; workers pausam e reconectam automaticamente |
| Processor registrado para jobType inexistente na fila | Fica inativo; nenhum erro; processará quando um job daquele tipo chegar |
| `QueueService.add` chamado com `options.jobId` já existente na fila | BullMQ ignora o enfileiramento silenciosamente; o job existente não é alterado — este é o mecanismo nativo de deduplicação; usar `options.jobId` quando idempotência de produção for necessária |
| Dois Processors registrados para a mesma fila em módulos diferentes | Comportamento indefinido — é proibido; cada fila deve ter exatamente um Processor |
| Job enfileirado antes de o Processor ser carregado (restart da app) | O job permanece em `waiting` e é processado após o Processor ser registrado |
| Payload não serializável em JSON passado ao `QueueService.add` | Lança `InvalidPayloadException` antes de tentar enfileirar |

---

## Restrições

- Nunca chamar `BullModule.registerQueue` em módulos que não sejam o `QueueModule` ou módulos de feature com Processor
- Nunca injetar `Queue` (via `@InjectQueue`) fora do `QueueService`
- Nunca criar nomes de fila como strings literais espalhadas no código — usar apenas as constantes de `queue.constants.ts`
- Nunca configurar `removeOnComplete: false` como padrão — jobs completados devem ser removidos para não acumular no Redis
- Nunca usar `synchronize: true` ou qualquer opção que altere automaticamente o schema do Redis em produção

---

## Painel de inspeção de filas

O `@bull-board/nestjs` expõe uma rota HTTP para visualizar e gerenciar jobs em todas as filas registradas. A instalação é obrigatória quando esta spec é aplicada.

- O painel expõe uma rota HTTP (padrão `/queues`) que lista filas, jobs e seus estados
- O `QueueModule` deve ser importado no `AppModule` para que a rota `/queues` esteja disponível globalmente — ver ADR-006 em `decisions.md`
- Todas as filas registradas em `BullModule.registerQueue` devem ser registradas também em `BullBoardModule.forFeature`; filas ausentes não aparecem no painel
- A rota `/queues` deve ser protegida por Basic Auth via `express-basic-auth` no `main.ts`, aplicado antes do `app.listen` — obrigatório em todos os ambientes; as credenciais vêm de `BULL_BOARD_USER` e `BULL_BOARD_PASSWORD` via `ConfigService`; o middleware é aplicado condicionalmente quando `REDIS_HOST` está presente (`configService.get('REDIS_HOST')`), de forma que projetos sem Redis não exijam as credenciais do painel

---

## O que esta spec NÃO cobre

- O comportamento de retry e DLQ → definido em `messaging/async-queue`
- Métricas de filas para coleta externa (Prometheus, OpenTelemetry)
- Lógica de negócio dentro dos Processors → responsabilidade da spec de cada feature
