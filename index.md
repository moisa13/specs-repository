# Catálogo de Specs

> Especificações de comportamento reutilizáveis — agnósticas de stack (quando possível), prontas para uso com Code Agents.

---

## Specs de comportamento

> Agnósticas de stack — descrevem o *o quê*. Podem ser usadas com qualquer linguagem ou framework.

| Módulo | Descrição | Versão | Status |
|--------|-----------|--------|--------|
| [`messaging/async-queue`](./messaging/async-queue/README.md) | Produção e consumo de jobs em filas assíncronas, com retry e dead letter queue | 0.1.0 | pronto |
| [`messaging/realtime`](./messaging/realtime/README.md) | Canal de comunicação persistente entre servidor e clientes, com autenticação no handshake e entrega bidirecional de eventos | 0.1.0 | rascunho |
| [`observability/structured-log`](./observability/structured-log/README.md) | Produção, propagação e sanitização de logs estruturados em aplicações de servidor | 0.1.0 | pronto |
| [`caching/cache`](./caching/cache/README.md) | Cache de dados com cache-aside, double-checked locking, invalidação e aquecimento no boot | 0.2.0 | pronto |

---

## Specs de implementação — NestJS

> Orientadas a stack — descrevem o *como* para projetos NestJS.

| Módulo | Descrição | Versão | Status |
|--------|-----------|--------|--------|
| [`setup/nestjs/nestjs-init`](./setup/nestjs/nestjs-init/README.md) | Estrutura base e configuração global de um projeto NestJS para APIs | 0.3.1 | pronto |
| [`setup/nestjs/nestjs-database`](./setup/nestjs/nestjs-database/README.md) | Conexão com PostgreSQL via TypeORM e configuração de migrations | 0.2.1 | pronto |
| [`setup/nestjs/nestjs-health`](./setup/nestjs/nestjs-health/README.md) | Endpoints de liveness e readiness via `@nestjs/terminus` com verificação de banco e memória | 0.3.1 | pronto |
| [`setup/nestjs/nestjs-logging`](./setup/nestjs/nestjs-logging/README.md) | Logging estruturado com Winston e rotação diária de arquivos | 0.1.1 | pronto |
| [`setup/nestjs/nestjs-cache`](./setup/nestjs/nestjs-cache/README.md) | Cache de dados com ioredis, CacheService, GenericCacheService e CacheWarmingService | 0.2.0 | pronto |
| [`setup/nestjs/nestjs-queue`](./setup/nestjs/nestjs-queue/README.md) | Filas assíncronas com BullMQ e Redis, com módulo centralizado | 0.1.1 | pronto |
| [`setup/nestjs/nestjs-realtime`](./setup/nestjs/nestjs-realtime/README.md) | Canal em tempo real com WebSocket e autenticação no handshake | — | rascunho |

---

## Legenda de status

| Status | Significado |
|--------|-------------|
| template | Estrutura criada, conteúdo a preencher |
| rascunho | Em elaboração, não usar em produção |
| pronto | Revisado e aprovado para uso |
| desatualizado | Implementações existentes podem estar desalinhadas |
| depreciado | Substituído por outro módulo, não usar em novos projetos |
