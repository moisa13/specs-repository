# Changelog — NestJS Filas Assíncronas (BullMQ + Redis)

> Versões seguem Semantic Versioning: MAJOR.MINOR.PATCH
>
> - **MAJOR** — mudança que quebra implementações existentes
> - **MINOR** — nova regra ou edge case (implementações existentes precisam de revisão)
> - **PATCH** — correção de texto, sem mudança de comportamento

---

## [0.1.1] — 2026-05

### Corrigido
- `contracts.md`, `examples/`: caminhos `src/queue/` atualizados para `src/infrastructure/queue/` — reflete a reestruturação do diretório de infraestrutura no projeto

---

## [0.1.0] — 2026-05

### Corrigido
- `examples/configurar-queue-service.md`: `Map<QueueName, Queue>` → `Map<string, Queue>` no campo privado — `QueueName` resolve para `never` quando `QUEUE_NAMES` está vazio
- `examples/configurar-bull-board.md`: import de `express-basic-auth` corrigido para default import
- `contracts.md`: `REDIS_HOST`, `BULL_BOARD_USER` e `BULL_BOARD_PASSWORD` marcados como `optional()` no schema Joi global para compatibilidade com projetos sem Redis; obrigatoriedade em runtime documentada via `getOrThrow`
- `behavior.md`: middleware Basic Auth do painel documentado como condicional à presença de `REDIS_HOST`

### Adicionado
- Versão inicial da spec
- Comportamento do `QueueModule` centralizado com `BullModule.forRootAsync` e `registerQueue`
- Comportamento do `QueueService` como único ponto de produção de jobs
- Convenção de Processors nos módulos de feature
- Variáveis de ambiente: `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD`, `REDIS_DB`, `QUEUE_DEFAULT_ATTEMPTS`, `QUEUE_DEFAULT_BACKOFF_DELAY`, `BULL_BOARD_USER`, `BULL_BOARD_PASSWORD`
- Instruções explícitas para adição ao schema Joi (`env.validation.ts`) e ao `.env.example`
- Tipo `QueueName` na assinatura pública do `QueueService.add()` com nota sobre origem
- Painel de inspeção de filas via `@bull-board/nestjs` com `BullBoardModule.forRoot` e `BullBoardModule.forFeature`
- Configuração de concorrência de workers via segundo argumento do `@Processor`: `{ concurrency: N }`
- Mecanismo de deduplicação via `options.jobId` documentado nos edge cases
- Documentação do estado inicial sem filas: `QUEUE_NAMES` começa vazio; primeira fila entra com a primeira feature
- Estrutura de diretórios: `src/queue/` e `src/[feature]/processors/`
- Convenção de `QUEUE_NAMES` em `queue.constants.ts`
- Edge cases: fila não registrada, Redis indisponível, dois Processors para a mesma fila, jobId duplicado
- 13 critérios de aceitação cobrindo fluxo feliz, centralização, falhas e edge cases
- 6 ADRs: QueueModule centralizado, QueueService facade, BullMQ vs Bull, Processors na feature, removeOnComplete, QueueModule no AppModule para Bull Board
- 5 exemplos de código: `configurar-queue-module`, `configurar-queue-service`, `registrar-processor-na-feature`, `testar-processor`, `configurar-bull-board`
- Adições ao `agents.md` do projeto
