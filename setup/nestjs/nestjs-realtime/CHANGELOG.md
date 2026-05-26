# Changelog — NestJS Canal de Comunicação em Tempo Real

> Versões seguem Semantic Versioning: MAJOR.MINOR.PATCH
>
> - **MAJOR** — mudança que quebra implementações existentes
> - **MINOR** — nova regra ou edge case (implementações existentes precisam de revisão)
> - **PATCH** — correção de texto, sem mudança de comportamento

---

## [0.1.0] — 2026-05

### Adicionado
- Versão inicial da spec
- Fluxo de configuração do `RealtimeModule` com gateway, service e guard
- Fluxo de conexão com autenticação JWT no handshake via `WsJwtGuard`
- Fluxo de desconexão com limpeza do mapa de conexões
- Fluxo de emissão via `RealtimeService.sendToUser`
- Fluxo de recebimento de eventos cliente→servidor via `@SubscribeMessage`
- Contratos: estrutura de diretórios, API do `RealtimeService`, adições ao `agents.md`
- ADR-001: Socket.IO em vez de WebSocket nativo
- ADR-002: `RealtimeService` como único ponto de emissão
- ADR-003: token JWT no query param do handshake
- ADR-004: `Map<userId, Set<Socket>>` para múltiplas conexões
- ADR-005: `RealtimeModule` importado no `AppModule`
