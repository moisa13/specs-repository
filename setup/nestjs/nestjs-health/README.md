# NestJS — Health Check (Liveness + Readiness)

> Expõe `/health/live` e `/health/ready` via `@nestjs/terminus` para monitoramento de liveness e readiness da aplicação.

---

**Spec version:** 0.3.1
**Status:** ✅ pronto
**Última revisão:** 2026-05
**Revisado por:** Moiséis Almeida

---

## O que é

Descreve como configurar endpoints de health check em um projeto NestJS seguindo o padrão liveness/readiness. O endpoint `/health/live` confirma que o processo está no ar sem verificar dependências externas. O endpoint `/health/ready` verifica se a aplicação está apta a receber tráfego, checando banco de dados e uso de memória.

## Quando usar

- Ao adicionar observabilidade a um projeto NestJS já inicializado com `setup/nestjs/nestjs-init`
- Ao configurar probes de liveness e readiness em Kubernetes
- Ao expor endpoints de health para load balancers e ferramentas de monitoramento

## Quando NÃO usar

- Para métricas de desempenho (latência, throughput) — ver `observability/nestjs-metrics` (a criar)
- Para logging estruturado — ver `observability/nestjs-logging` (a criar)
- Antes de ter `setup/nestjs/nestjs-init` aplicado

## Módulos relacionados

- `setup/nestjs/nestjs-init` — pré-requisito; disponibiliza `ConfigService` e a estrutura global
- `setup/nestjs/nestjs-database` — fornece a conexão TypeORM usada pelo `TypeOrmHealthIndicator`
- `observability/nestjs-metrics` — métricas de desempenho (a criar)

## Arquivos desta spec

| Arquivo | Conteúdo |
|---------|----------|
| [`context.md`](./context.md) | Plataformas, pré-requisitos, escopo |
| [`behavior.md`](./behavior.md) | Regras, fluxos, edge cases |
| [`contracts.md`](./contracts.md) | Endpoints, pacotes, estrutura de diretórios, formato de resposta |
| [`integration.md`](./integration.md) | Dependências e integrações |
| [`acceptance.md`](./acceptance.md) | Critérios de aceitação |
| [`decisions.md`](./decisions.md) | Decisões arquiteturais (ADRs) |
| [`CHANGELOG.md`](./CHANGELOG.md) | Histórico de versões |
| [`examples/`](./examples/) | Exemplos concretos |
