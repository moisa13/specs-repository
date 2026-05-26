# Changelog — Canal de Comunicação em Tempo Real

> Versões seguem Semantic Versioning: MAJOR.MINOR.PATCH
>
> - **MAJOR** — mudança que quebra implementações existentes
> - **MINOR** — nova regra ou edge case (implementações existentes precisam de revisão)
> - **PATCH** — correção de texto, sem mudança de comportamento

---

## [0.1.0] — 2026-05

### Adicionado
- Versão inicial da spec
- Fluxo de conexão com autenticação no handshake
- Fluxo de entrega de eventos servidor → cliente com agrupamento por identidade do cliente
- Fluxo de entrega de eventos cliente → servidor com despacho por nome de evento
- Fluxo de desconexão com limpeza de estado
- Contratos de Event e Connection
- Edge cases: token expirado pós-conexão, múltiplas conexões, cliente offline, payload inválido
- ADR-001: autenticação no handshake
- ADR-002: endereçamento por identidade do cliente
- ADR-003: entrega fire-and-forget sem confirmação de recebimento
- ADR-004: descarte silencioso para clientes offline

### Corrigido
- Terminologia para payload inválido (cliente→servidor) unificada para "ignorado silenciosamente" em behavior.md (edge case) e acceptance.md (AC-09)
- Nível de detalhe do edge case "sem handler registrado" equiparado ao de "payload inválido" em behavior.md
- Adicionado example `envio-evento-cliente.md` cobrindo a direção cliente→servidor (fluxo principal, sem handler, payload inválido)
