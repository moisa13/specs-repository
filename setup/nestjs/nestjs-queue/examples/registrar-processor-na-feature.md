# Exemplo — Registrar Processor em um módulo de feature

## Contexto

O módulo `notifications` precisa processar jobs do tipo `send-email` da fila `notifications`. O Processor fica dentro do módulo de feature, que importa `QueueModule`.

---

## `src/notifications/processors/send-email.processor.ts`

```typescript
import { Processor, WorkerHost } from '@nestjs/bullmq';
import { Job } from 'bullmq';
import { QUEUE_NAMES } from '../../infrastructure/queue/queue.constants';
import { EmailService } from '../email.service';

@Processor(QUEUE_NAMES.NOTIFICATIONS, { concurrency: 1 })
export class SendEmailProcessor extends WorkerHost {
  constructor(private readonly emailService: EmailService) {
    super();
  }

  async process(job: Job): Promise<void> {
    switch (job.name) {
      case 'send-email':
        await this.emailService.send(job.data);
        break;
      default:
        throw new Error(`Tipo de job desconhecido: ${job.name}`);
    }
  }
}
```

---

## `src/notifications/notifications.module.ts`

```typescript
import { BullModule } from '@nestjs/bullmq';
import { Module } from '@nestjs/common';
import { QUEUE_NAMES } from '../infrastructure/queue/queue.constants';
import { QueueModule } from '../infrastructure/queue/queue.module';
import { SendEmailProcessor } from './processors/send-email.processor';
import { EmailService } from './email.service';

@Module({
  imports: [
    QueueModule,
    BullModule.registerQueue({ name: QUEUE_NAMES.NOTIFICATIONS }),
  ],
  providers: [EmailService, SendEmailProcessor],
})
export class NotificationsModule {}
```

---

## Produção do job a partir de outro service

```typescript
import { Injectable } from '@nestjs/common';
import { QueueService } from '../infrastructure/queue/queue.service';
import { QUEUE_NAMES } from '../infrastructure/queue/queue.constants';

@Injectable()
export class UserService {
  constructor(private readonly queueService: QueueService) {}

  async registerUser(data: CreateUserDto): Promise<void> {
    const user = await this.userRepository.create(data);

    await this.queueService.add(
      QUEUE_NAMES.NOTIFICATIONS,
      'send-email',
      { to: user.email, subject: 'Bem-vindo', templateId: 'welcome' },
    );
  }
}
```

## Notas

- O `NotificationsModule` importa `QueueModule` para ter acesso ao `QueueService` via injeção de dependência
- O `BullModule.registerQueue` no módulo de feature é necessário para que o `@Processor` consiga conectar ao worker da fila correta — não substitui o registro centralizado no `QueueModule`
- Lançar erro no `process()` para jobType desconhecido garante que jobs mal roteados não sejam descartados silenciosamente
- `concurrency` controla quantos jobs dessa fila o worker processa simultaneamente; o padrão é `1` — aumentar apenas quando o processamento for I/O-bound e o paralelismo for desejado explicitamente
