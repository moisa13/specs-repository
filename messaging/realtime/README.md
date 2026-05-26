# Canal de Comunicação em Tempo Real

> Define o comportamento de um canal de comunicação persistente entre servidor e clientes, com autenticação no handshake e entrega bidirecional de eventos.

---

**Spec version:** 0.1.0
**Status:** 📝 rascunho
**Última revisão:** 2026-05
**Revisado por:** —

---

## O que é

Descreve o modelo genérico de comunicação em tempo real baseado em conexão persistente: como uma conexão é estabelecida e autenticada, como eventos trafegam em ambas as direções e como o ciclo de vida da conexão é gerenciado. A spec é agnóstica de linguagem, framework e protocolo de transporte — serve como referência de comportamento para specs de implementação específicas.

## Quando usar

- Ao modelar qualquer funcionalidade que precise entregar dados a clientes sem polling
- Como referência de comportamento para specs de implementação (ex: `setup/nestjs/nestjs-realtime`)

## Quando NÃO usar

- Para implementar o canal em uma stack específica — ver a spec de implementação correspondente (ex: `setup/nestjs/nestjs-realtime`)
- Para comunicação assíncrona sem necessidade de conexão persistente — ver `messaging/async-queue`
- Para definir o que transita pelo canal (quais eventos, quando, para quem) — responsabilidade da spec de feature correspondente

## Módulos relacionados

- `setup/nestjs/nestjs-realtime` — implementação desta spec em NestJS com Socket.IO

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
