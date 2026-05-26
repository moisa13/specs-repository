# Contracts — NestJS Canal de Comunicação em Tempo Real

---

## Variáveis de ambiente

Nenhuma variável nova é introduzida por esta spec. O `WsJwtGuard` reutiliza `JWT_SECRET` já definido por `setup/nestjs/nestjs-init`.

| Variável | Definida em | Uso nesta spec |
|----------|-------------|----------------|
| `JWT_SECRET` | `setup/nestjs/nestjs-init` | Validação do token JWT no handshake via `JwtService` |

---

## Estrutura de diretórios

```
src/
└── infrastructure/
    └── realtime/
        ├── realtime.module.ts      ← declara e exporta RealtimeGateway e RealtimeService
        ├── realtime.gateway.ts     ← WebSocketGateway; handleConnection, handleDisconnect, @SubscribeMessage
        ├── realtime.service.ts     ← Map de conexões; único ponto de emissão via sendToUser
        └── guards/
            └── ws-jwt.guard.ts     ← valida JWT do query param; desconecta socket em falha
```

---

## API do RealtimeService

### `RealtimeService.sendToUser`

Emite um evento para todas as conexões ativas de um cliente identificado pelo `userId`.

**Assinatura:**
```typescript
sendToUser(userId: string, event: string, payload: unknown): void
```

**Parâmetros:**

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| `userId` | string | ✅ | Identidade do cliente; deve corresponder ao `sub` do JWT |
| `event` | string | ✅ | Nome do evento; determina o handler no cliente |
| `payload` | unknown | ✅ | Dados do evento; deve ser serializável em JSON |

**Comportamento:**
- Se o `userId` tiver conexões ativas, o evento é emitido via `socket.emit(event, payload)` para cada socket do conjunto
- Se o `userId` não tiver conexões ativas, o método retorna sem erro — sem exceção, sem log de aviso

### `RealtimeService.addConnection`

Registra um socket no mapa de conexões. Chamado pelo gateway no `handleConnection`.

**Assinatura:**
```typescript
addConnection(userId: string, socket: Socket): void
```

### `RealtimeService.removeConnection`

Remove um socket do mapa de conexões. Chamado pelo gateway no `handleDisconnect`.

**Assinatura:**
```typescript
removeConnection(userId: string, socket: Socket): void
```

---

## Convenção de identificação de cliente

O `userId` é extraído do campo `sub` do payload JWT e armazenado em `socket.data.userId` no momento da conexão. O `handleDisconnect` usa `socket.data.userId` para localizar e remover o socket do mapa sem precisar revalidar o token.

---

## Adições ao agents.md

Ao aplicar esta spec, acrescentar a seção abaixo ao `agents.md` do projeto:

```markdown
## Canal em tempo real (setup/nestjs/nestjs-realtime)

- `RealtimeService` é o único provider que emite eventos para clientes — nunca injetar `RealtimeGateway` diretamente em módulos de feature
- Handlers de eventos cliente→servidor (`@SubscribeMessage`) são definidos exclusivamente em `src/infrastructure/realtime/realtime.gateway.ts`
- Para emitir um evento a partir de uma feature, importar `RealtimeModule` no módulo da feature e injetar `RealtimeService`
- `RealtimeModule` está importado no `AppModule` — não importar novamente no `AppModule` ao adicionar features
```
