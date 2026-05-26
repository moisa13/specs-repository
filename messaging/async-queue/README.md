# Processamento Assíncrono com Fila

> Define o comportamento de produção e consumo de jobs em filas assíncronas, com retry automático e dead letter queue.

---

**Spec version:** 0.1.0
**Status:** 📝 rascunho
**Última revisão:** 2026-05
**Revisado por:** Moiséis Almeida

---

## O que é

Descreve o modelo genérico de processamento assíncrono baseado em filas: como jobs são produzidos, enfileirados, consumidos e tratados em caso de falha. A spec é agnóstica de linguagem, framework e tecnologia de fila — serve como referência de comportamento para specs de implementação específicas.

## Quando usar

- Ao modelar qualquer fluxo que precisa ser executado fora do ciclo de request/response
- Ao projetar integração entre módulos via processamento assíncrono
- Como referência para specs de implementação que dependem de filas (ex: `setup/nestjs/nestjs-queue`)

## Quando NÃO usar

- Para implementar filas em uma stack específica — ver a spec de implementação correspondente (ex: `setup/nestjs/nestjs-queue`)
- Para comunicação síncrona entre serviços — use chamadas diretas ou RPC
- Para eventos de domínio internos sem necessidade de persistência ou retry — use event emitters

## Módulos relacionados

- `setup/nestjs/nestjs-queue` — implementação desta spec em NestJS com BullMQ

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
| [`examples/`](./examples/) | Exemplos concretos de entrada e saída |
