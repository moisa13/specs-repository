# NestJS — Filas Assíncronas (BullMQ + Redis)

> Configura o processamento assíncrono com BullMQ em projetos NestJS, centralizando todas as filas em um único módulo gerenciado.

---

**Spec version:** 0.1.1
**Status:** ✅ pronto
**Última revisão:** 2026-05
**Revisado por:** Moiséis Almeida

---

## O que é

Descreve como implementar processamento assíncrono com BullMQ e Redis em um projeto NestJS já inicializado. Define um `QueueModule` centralizado que registra todas as filas da aplicação e um `QueueService` que serve como único ponto de produção de jobs. Módulos de feature registram apenas seus Processors — nunca filas diretamente.

## Quando usar

- Ao adicionar processamento assíncrono a um projeto NestJS já inicializado com `setup/nestjs/nestjs-init`
- Ao implementar qualquer spec de domínio que referencia `messaging/async-queue`

## Quando NÃO usar

- Antes de ter `setup/nestjs/nestjs-init` aplicado — o `ConfigService` deve estar disponível
- Para comunicação pub/sub entre serviços — esta spec cobre apenas filas de trabalho (job queues)
- Para agendamento de tarefas sem fila persistente — use `@nestjs/schedule`

## Módulos relacionados

- `setup/nestjs/nestjs-init` — pré-requisito; disponibiliza `ConfigService` e a estrutura global
- `messaging/async-queue` — spec de comportamento que esta implementação satisfaz

## Arquivos desta spec

| Arquivo | Conteúdo |
|---------|----------|
| [`context.md`](./context.md) | Plataformas, pré-requisitos, escopo |
| [`behavior.md`](./behavior.md) | Regras, fluxos, edge cases |
| [`contracts.md`](./contracts.md) | Estrutura de diretórios, variáveis de ambiente, API do QueueService |
| [`integration.md`](./integration.md) | Dependências e integrações |
| [`acceptance.md`](./acceptance.md) | Critérios de aceitação |
| [`decisions.md`](./decisions.md) | Decisões arquiteturais (ADRs) |
| [`CHANGELOG.md`](./CHANGELOG.md) | Histórico de versões |
| [`examples/configurar-queue-module.md`](./examples/configurar-queue-module.md) | `queue.constants.ts` e `queue.module.ts` com duas filas registradas |
| [`examples/configurar-queue-service.md`](./examples/configurar-queue-service.md) | `queue.service.ts` com `add()`, erros e mapa de filas |
| [`examples/registrar-processor-na-feature.md`](./examples/registrar-processor-na-feature.md) | Processor em módulo de feature e produção de job a partir de um service |
| [`examples/testar-processor.md`](./examples/testar-processor.md) | Testes unitários de Processor e QueueService com `@nestjs/testing` |
| [`examples/configurar-bull-board.md`](./examples/configurar-bull-board.md) | Painel de inspeção de filas com `@bull-board/nestjs` |
