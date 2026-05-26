# Exemplo — Configurar WsJwtGuard

## Arquivo: `src/infrastructure/realtime/guards/ws-jwt.guard.ts`

```typescript
import { JwtService } from '@nestjs/jwt';
import { Socket } from 'socket.io';

export class WsJwtGuard {
  static async authenticate(socket: Socket): Promise<string | null> {
    const token = socket.handshake.query['token'];
    if (!token || typeof token !== 'string') return null;

    try {
      const jwtService = socket.data._jwtService as JwtService;
      const payload = await jwtService.verifyAsync<{ sub: string }>(token);
      return payload.sub;
    } catch {
      return null;
    }
  }
}
```

> O `JwtService` é injetado no socket via `socket.data._jwtService` pelo `RealtimeGateway` durante o `handleConnection`, antes de chamar `WsJwtGuard.authenticate`. O gateway recebe o `JwtService` por injeção de dependência do NestJS via `RealtimeModule`.

## Integração no gateway (trecho relevante)

```typescript
@WebSocketGateway({ cors: { origin: '*' } })
export class RealtimeGateway implements OnGatewayConnection {
  constructor(
    private readonly realtimeService: RealtimeService,
    private readonly jwtService: JwtService,
  ) {}

  async handleConnection(socket: Socket): Promise<void> {
    socket.data._jwtService = this.jwtService;
    const userId = await WsJwtGuard.authenticate(socket);
    if (!userId) {
      socket.emit('error', { code: 'UNAUTHORIZED' });
      socket.disconnect();
      return;
    }
    socket.data.userId = userId;
    this.realtimeService.addConnection(userId, socket);
  }
}
```
