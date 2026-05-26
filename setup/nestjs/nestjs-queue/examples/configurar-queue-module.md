# Exemplo — Configurar QueueModule

## Contexto

Configuração inicial do `QueueModule` com duas filas registradas: `notifications` e `reports`.

---

## `src/queue/queue.constants.ts`

```typescript
export const QUEUE_NAMES = {
  NOTIFICATIONS: 'notifications',
  REPORTS: 'reports',
} as const;

export type QueueName = (typeof QUEUE_NAMES)[keyof typeof QUEUE_NAMES];
```

---

## `src/queue/queue.module.ts`

```typescript
import { BullModule } from '@nestjs/bullmq';
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
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
          db: config.get<number>('REDIS_DB', 0),
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
  ],
  providers: [QueueService],
  exports: [QueueService],
})
export class QueueModule {}
```

## Notas

- `removeOnFail: false` garante que jobs falhos permaneçam na DLQ para auditoria
- `REDIS_PASSWORD` é opcional — em ambientes sem autenticação, `ConfigService.get` retorna `undefined` e o BullMQ omite a senha
- Para adicionar uma nova fila: adicionar em `QUEUE_NAMES` e incluir em `BullModule.registerQueue`
