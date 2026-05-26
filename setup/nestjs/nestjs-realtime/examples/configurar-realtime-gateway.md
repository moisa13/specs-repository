# Exemplo — Configurar RealtimeGateway

## Arquivo: `src/infrastructure/realtime/realtime.gateway.ts`

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  OnGatewayConnection,
  OnGatewayDisconnect,
  SubscribeMessage,
  MessageBody,
  ConnectedSocket,
} from '@nestjs/websockets';
import { UseGuards } from '@nestjs/common';
import { Server, Socket } from 'socket.io';
import { RealtimeService } from './realtime.service';
import { WsJwtGuard } from './guards/ws-jwt.guard';

@WebSocketGateway({ cors: { origin: '*' } })
export class RealtimeGateway implements OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer()
  server: Server;

  constructor(private readonly realtimeService: RealtimeService) {}

  async handleConnection(socket: Socket): Promise<void> {
    const userId = await WsJwtGuard.authenticate(socket);
    if (!userId) {
      socket.emit('error', { code: 'UNAUTHORIZED' });
      socket.disconnect();
      return;
    }
    socket.data.userId = userId;
    this.realtimeService.addConnection(userId, socket);
  }

  handleDisconnect(socket: Socket): void {
    const userId = socket.data.userId as string | undefined;
    if (userId) {
      this.realtimeService.removeConnection(userId, socket);
    }
  }

  @SubscribeMessage('ping')
  handlePing(@ConnectedSocket() socket: Socket, @MessageBody() payload: unknown): void {
    socket.emit('pong', payload);
  }
}
```

> O método `@SubscribeMessage('ping')` é apenas ilustrativo. Remova-o ou substitua pelo handler adequado ao projeto. Todos os handlers de eventos cliente→servidor devem ser definidos neste arquivo.
