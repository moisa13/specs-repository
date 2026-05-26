# Exemplo — Configurar RealtimeService

## Arquivo: `src/infrastructure/realtime/realtime.service.ts`

```typescript
import { Injectable } from '@nestjs/common';
import { Socket } from 'socket.io';

@Injectable()
export class RealtimeService {
  private readonly connections = new Map<string, Set<Socket>>();

  addConnection(userId: string, socket: Socket): void {
    if (!this.connections.has(userId)) {
      this.connections.set(userId, new Set());
    }
    this.connections.get(userId)!.add(socket);
  }

  removeConnection(userId: string, socket: Socket): void {
    const sockets = this.connections.get(userId);
    if (!sockets) return;
    sockets.delete(socket);
    if (sockets.size === 0) {
      this.connections.delete(userId);
    }
  }

  sendToUser(userId: string, event: string, payload: unknown): void {
    const sockets = this.connections.get(userId);
    if (!sockets) return;
    for (const socket of sockets) {
      socket.emit(event, payload);
    }
  }
}
```

## Uso em um service de feature

```typescript
import { Injectable } from '@nestjs/common';
import { RealtimeService } from '../infrastructure/realtime/realtime.service';

@Injectable()
export class PedidoService {
  constructor(private readonly realtimeService: RealtimeService) {}

  async atualizarStatus(pedidoId: string, userId: string, novoStatus: string): Promise<void> {
    // ... lógica de domínio

    this.realtimeService.sendToUser(userId, 'pedido.atualizado', {
      pedidoId,
      status: novoStatus,
    });
  }
}
```
