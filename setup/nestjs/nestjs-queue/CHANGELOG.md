# Changelog — NestJS Filas Assíncronas (BullMQ + Redis)

> Versões seguem Semantic Versioning: MAJOR.MINOR.PATCH
>
> - **MAJOR** — mudança que quebra implementações existentes
> - **MINOR** — nova regra ou edge case (implementações existentes precisam de revisão)
> - **PATCH** — correção de texto, sem mudança de comportamento

---

## [0.1.0] — 2026-05

### Adicionado
- Versão inicial da spec
- Comportamento do `QueueModule` centralizado com `BullModule.forRootAsync` e `registerQueue`
- Comportamento do `QueueService` como único ponto de produção de jobs
- Convenção de Processors nos módulos de feature
- Variáveis de ambiente: `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD`, `QUEUE_DEFAULT_ATTEMPTS`, `QUEUE_DEFAULT_BACKOFF_DELAY`
- Estrutura de diretórios: `src/queue/` e `src/[feature]/processors/`
- Convenção de `QUEUE_NAMES` em `queue.constants.ts`
- Edge cases: fila não registrada, Redis indisponível, dois Processors para a mesma fila
- 13 critérios de aceitação cobrindo fluxo feliz, centralização, falhas e edge cases
- 5 ADRs: QueueModule centralizado, QueueService facade, BullMQ vs Bull, Processors na feature, removeOnComplete
- Adições ao `agents.md` do projeto
