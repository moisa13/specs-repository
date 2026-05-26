# NestJS — Canal de Comunicação em Tempo Real (Socket.IO)

> Configura um canal de comunicação persistente em projetos NestJS usando Socket.IO, com autenticação por JWT no handshake e um serviço centralizado de emissão de eventos.

---

**Spec version:** 0.1.0
**Status:** 📝 rascunho
**Última revisão:** 2026-05
**Revisado por:** —

---

## O que é

Descreve como implementar o comportamento definido em `messaging/realtime` em um projeto NestJS. Define os componentes necessários (`RealtimeModule`, `RealtimeGateway`, `RealtimeService`, `WsJwtGuard`), sua configuração com Socket.IO e a integração com o ciclo de vida do framework.

## Quando usar

- Ao adicionar um canal de comunicação em tempo real a um projeto NestJS já inicializado com `setup/nestjs/nestjs-init`
- Ao implementar qualquer spec de feature que referencie `messaging/realtime`

## Quando NÃO usar

- Antes de ter `setup/nestjs/nestjs-init` aplicado — o `ConfigService` e a variável `JWT_SECRET` devem estar disponíveis
- Para comunicação assíncrona sem necessidade de conexão persistente — ver `setup/nestjs/nestjs-queue`
- Em projetos com framework diferente de NestJS — usar `messaging/realtime` como guia e adaptar à stack

## Módulos relacionados

- `messaging/realtime` — spec de comportamento que esta implementação satisfaz
- `setup/nestjs/nestjs-init` — pré-requisito; disponibiliza `ConfigService` e `JWT_SECRET`
- `setup/nestjs/nestjs-queue` — usa Redis separado para filas; independente desta spec

## Arquivos desta spec

| Arquivo | Conteúdo |
|---------|----------|
| [`context.md`](./context.md) | Plataformas, pré-requisitos, escopo |
| [`behavior.md`](./behavior.md) | Componentes e fluxo de implementação |
| [`contracts.md`](./contracts.md) | Estrutura de diretórios, API do RealtimeService, agents.md |
| [`integration.md`](./integration.md) | Dependências e pontos de integração |
| [`acceptance.md`](./acceptance.md) | Critérios de aceitação |
| [`decisions.md`](./decisions.md) | Decisões arquiteturais (ADRs) |
| [`CHANGELOG.md`](./CHANGELOG.md) | Histórico de versões |
| [`examples/configurar-realtime-module.md`](./examples/configurar-realtime-module.md) | `realtime.module.ts` com gateway e service |
| [`examples/configurar-realtime-gateway.md`](./examples/configurar-realtime-gateway.md) | `realtime.gateway.ts` com lifecycle hooks e WsJwtGuard |
| [`examples/configurar-realtime-service.md`](./examples/configurar-realtime-service.md) | `realtime.service.ts` com `sendToUser` e mapa de conexões |
| [`examples/configurar-ws-jwt-guard.md`](./examples/configurar-ws-jwt-guard.md) | `ws-jwt.guard.ts` validando JWT do query param |
