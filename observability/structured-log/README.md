# Registro Estruturado de Logs

> Comportamento de produção, propagação e sanitização de logs estruturados em aplicações de servidor.

---

**Spec version:** 0.1.0
**Status:** ✅ pronto
**Última revisão:** 2026-05
**Revisado por:** Moiséis Almeida

---

## O que é

Este módulo define como uma aplicação de servidor deve registrar sua atividade interna de forma estruturada, rastreável e segura. Cobre desde a geração do identificador de correlação por requisição até as regras de sanitização de dados sensíveis e a classificação de exceções por severidade.

## Quando usar

- Ao implementar logging estruturado em qualquer serviço de backend
- Quando rastreabilidade de requisições por `correlationId` for requisito
- Quando o serviço precisa garantir que dados sensíveis nunca sejam escritos em logs

## Quando NÃO usar

- Para monitoramento de erros com captura em serviço externo (ex: Sentry) → ver `observability/error-tracking` (a criar)
- Para tracing distribuído entre serviços → ver `observability/tracing` (a criar)
- Para logs de auditoria de ações de usuário com persistência em banco → escopo distinto

## Módulos relacionados

- `setup/nestjs/nestjs-logging` — implementação desta spec para NestJS com Winston e nest-cls

## Arquivos desta spec

| Arquivo | Conteúdo |
|---------|----------|
| [`context.md`](./context.md) | Plataformas, pré-requisitos, escopo |
| [`behavior.md`](./behavior.md) | Regras, fluxos, edge cases |
| [`contracts.md`](./contracts.md) | Entidades, campos, validações |
| [`integration.md`](./integration.md) | Dependências e integrações |
| [`acceptance.md`](./acceptance.md) | Critérios de aceitação |
| [`decisions.md`](./decisions.md) | Decisões arquiteturais (ADRs) |
| [`CHANGELOG.md`](./CHANGELOG.md) | Histórico de versões |
| [`examples/`](./examples/) | Exemplos concretos de entradas de log |
