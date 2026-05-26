# Acceptance Criteria — NestJS Canal de Comunicação em Tempo Real

> Formato: **Dado** [contexto] **Quando** [ação] **Então** [resultado esperado]

---

## Fluxo principal

**AC-01 — Gateway sobe junto com a aplicação**
- **Dado** que `RealtimeModule` está importado no `AppModule` e `JWT_SECRET` está configurado
- **Quando** a aplicação inicia
- **Então** o gateway WebSocket está ativo na mesma porta da API HTTP e pronto para aceitar conexões

**AC-02 — Conexão aceita com JWT válido**
- **Dado** um cliente enviando uma conexão WebSocket com `?token=<jwt-válido>` no query param
- **Quando** o `WsJwtGuard` valida o token
- **Então** a conexão é aceita, o `userId` é extraído do `sub` do payload JWT, armazenado em `socket.data.userId` e o socket é adicionado ao mapa do `RealtimeService`

**AC-03 — Emissão via RealtimeService entrega a todos os sockets do cliente**
- **Dado** um cliente com dois sockets ativos registrados no `RealtimeService`
- **Quando** um service de feature chama `RealtimeService.sendToUser(userId, 'evento', payload)`
- **Então** o evento é emitido para os dois sockets via `socket.emit`

---

## Centralização

**AC-04 — RealtimeService como único ponto de emissão**
- **Dado** que qualquer módulo de feature precisa emitir um evento em tempo real
- **Quando** auditado
- **Então** nenhum acesso direto ao `RealtimeGateway` ou ao `socket` existe fora de `src/infrastructure/realtime/`

**AC-05 — Handlers de eventos cliente→servidor exclusivos no gateway**
- **Dado** que um evento WebSocket enviado pelo cliente precisa de tratamento
- **Quando** o handler é criado
- **Então** ele está decorado com `@SubscribeMessage` dentro de `realtime.gateway.ts` — não em outro arquivo

---

## Cenários de falha

**AC-06 — Conexão rejeitada com token ausente**
- **Dado** um cliente conectando sem o query param `token`
- **Quando** o `WsJwtGuard` é acionado no `handleConnection`
- **Então** o guard emite o erro `UNAUTHORIZED` via `socket.emit('error', { code: 'UNAUTHORIZED' })`, desconecta o socket e nenhuma entrada é criada no mapa

**AC-07 — Conexão rejeitada com token inválido ou expirado**
- **Dado** um cliente conectando com `?token=<jwt-inválido-ou-expirado>`
- **Quando** o `WsJwtGuard` tenta validar o token
- **Então** o guard emite o erro `UNAUTHORIZED`, desconecta o socket e nenhuma entrada é criada no mapa

---

## Edge cases

**AC-08 — Silent drop para cliente sem conexão ativa**
- **Dado** que `RealtimeService.sendToUser` é chamado com um `userId` sem sockets no mapa
- **Quando** o método é executado
- **Então** retorna sem erro e sem log de aviso — nenhum evento é emitido

**AC-09 — Desconexão remove o socket do mapa**
- **Dado** um cliente com um socket ativo registrado no mapa
- **Quando** o socket é encerrado (voluntariamente ou por perda de rede)
- **Então** o `handleDisconnect` remove o socket do conjunto do cliente; se o conjunto ficar vazio, a entrada do `userId` é removida do mapa

**AC-10 — Reinício da aplicação limpa todas as conexões**
- **Dado** que clientes estão conectados ao gateway
- **Quando** a aplicação reinicia
- **Então** o mapa é recriado vazio; os clientes precisam reconectar e reautenticar

---

## O que caracteriza uma implementação incorreta

- `RealtimeGateway` injetado em módulo de feature
- `@SubscribeMessage` definido fora de `realtime.gateway.ts`
- Socket armazenado fora do `RealtimeService`
- `RealtimeModule` importado novamente no `AppModule` ao adicionar uma feature (já está lá)
- Gateway rodando em porta separada da API HTTP

---

## Cobertura mínima de testes

- [ ] Gateway aceita conexão com JWT válido e adiciona socket ao mapa (AC-02)
- [ ] Gateway rejeita conexão com token ausente e não adiciona ao mapa (AC-06)
- [ ] Gateway rejeita conexão com token inválido e não adiciona ao mapa (AC-07)
- [ ] `sendToUser` emite evento para todos os sockets do cliente (AC-03)
- [ ] `sendToUser` retorna sem erro quando cliente não tem sockets ativos (AC-08)
- [ ] `handleDisconnect` remove socket do mapa e limpa entrada vazia (AC-09)
