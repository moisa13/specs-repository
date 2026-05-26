# Exemplo — Configurar Bull Board (painel de inspeção de filas)

## Contexto

O `@bull-board/nestjs` expõe uma rota HTTP para visualizar e gerenciar jobs em todas as filas registradas. O painel é obrigatório quando esta spec é aplicada. Este exemplo mostra como configurar o painel no `QueueModule` e importar o `QueueModule` no `AppModule`.

---

## Pacotes a instalar

```bash
pnpm add @bull-board/api @bull-board/nestjs @bull-board/express
```

---

## `src/infrastructure/queue/queue.module.ts` (com Bull Board)

```typescript
import { BullModule } from '@nestjs/bullmq';
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { BullBoardModule } from '@bull-board/nestjs';
import { BullMQAdapter } from '@bull-board/api/bullMQAdapter';
import { ExpressAdapter } from '@bull-board/express';
import { QUEUE_NAMES } from './queue.constants';
import { QueueService } from './queue.service';

@Module({
  imports: [
    BullModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        connection: {
          host: config.getOrThrow<string>('REDIS_HOST'),
          port: config.get<number>('REDIS_PORT', 6379),
          password: config.get<string>('REDIS_PASSWORD'),
        },
        defaultJobOptions: {
          attempts: config.get<number>('QUEUE_DEFAULT_ATTEMPTS', 3),
          backoff: {
            type: 'exponential',
            delay: config.get<number>('QUEUE_DEFAULT_BACKOFF_DELAY', 1000),
          },
          removeOnComplete: { count: 100 },
          removeOnFail: false,
        },
      }),
    }),
    BullModule.registerQueue(
      { name: QUEUE_NAMES.NOTIFICATIONS },
      { name: QUEUE_NAMES.REPORTS },
    ),
    BullBoardModule.forRoot({
      route: '/queues',
      adapter: ExpressAdapter,
    }),
    BullBoardModule.forFeature(
      { name: QUEUE_NAMES.NOTIFICATIONS, adapter: BullMQAdapter },
      { name: QUEUE_NAMES.REPORTS, adapter: BullMQAdapter },
    ),
  ],
  providers: [QueueService],
  exports: [QueueService],
})
export class QueueModule {}
```

---

## Protegendo a rota em produção

Sem proteção, qualquer requisição pode acessar o painel e reprocessar jobs. Aplicar um guard ou middleware na rota `/queues` antes de subir em produção:

```typescript
// src/main.ts — exemplo com BasicAuth via middleware
import basicAuth from 'express-basic-auth';

app.use(
  '/queues',
  basicAuth({
    users: { [process.env.BULL_BOARD_USER]: process.env.BULL_BOARD_PASSWORD },
    challenge: true,
  }),
);
```

> `express-basic-auth` é apenas um exemplo — instalar com `pnpm add express-basic-auth` se optar por este mecanismo. Qualquer outro mecanismo de autenticação disponível no projeto pode ser usado — guards NestJS, JWT, session, etc.

---

## Notas

- Ao adicionar uma nova fila em `BullModule.registerQueue`, adicionar a entrada correspondente em `BullBoardModule.forFeature`; filas não registradas não aparecem no painel
- O painel é acessível em `http://localhost:<PORT>/queues` durante o desenvolvimento
- Em ambiente de desenvolvimento, a proteção da rota é opcional
