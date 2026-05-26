# Exemplo — Configurar QueueService

## Contexto

Implementação do `QueueService`, único provider que acessa `@InjectQueue`. Expõe o método `add()` para todos os modules de feature.

---

## `src/queue/queue.service.ts`

```typescript
import { InjectQueue } from '@nestjs/bullmq';
import { Injectable } from '@nestjs/common';
import { Queue } from 'bullmq';
import { QUEUE_NAMES, QueueName } from './queue.constants';

export interface JobOptions {
  jobId?: string;
  delay?: number;
  priority?: number;
  attempts?: number;
  backoff?: {
    type?: 'exponential' | 'fixed';
    delay: number;
  };
}

export interface EnqueueResult {
  jobId: string;
  queueName: string;
  status: 'waiting' | 'delayed';
}

@Injectable()
export class QueueService {
  private readonly queues: Map<string, Queue>;

  constructor(
    @InjectQueue(QUEUE_NAMES.NOTIFICATIONS) private notificationsQueue: Queue,
    @InjectQueue(QUEUE_NAMES.REPORTS) private reportsQueue: Queue,
  ) {
    this.queues = new Map([
      [QUEUE_NAMES.NOTIFICATIONS, notificationsQueue],
      [QUEUE_NAMES.REPORTS, reportsQueue],
    ]);
  }

  async add(
    queueName: QueueName,
    jobType: string,
    payload: Record<string, unknown>,
    options: JobOptions = {},
  ): Promise<EnqueueResult> {
    const queue = this.queues.get(queueName);

    if (!queue) {
      throw new QueueNotFoundException(queueName);
    }

    if (!isJsonSerializable(payload)) {
      throw new InvalidPayloadException();
    }

    try {
      const job = await queue.add(jobType, payload, {
        jobId: options.jobId,
        delay: options.delay,
        priority: options.priority,
        attempts: options.attempts,
        backoff: options.backoff
          ? { type: options.backoff.type ?? 'exponential', delay: options.backoff.delay }
          : undefined,
      });

      return {
        jobId: job.id!,
        queueName,
        status: options.delay && options.delay > 0 ? 'delayed' : 'waiting',
      };
    } catch (error) {
      if (isRedisConnectionError(error)) {
        throw new QueueUnavailableException();
      }
      throw error;
    }
  }
}

function isRedisConnectionError(error: unknown): boolean {
  if (!(error instanceof Error)) return false;
  return (
    error.message.includes('ECONNREFUSED') ||
    error.message.includes('ENOTFOUND') ||
    error.message.includes('Connection is closed')
  );
}

function isJsonSerializable(value: unknown): boolean {
  try {
    JSON.stringify(value);
    return true;
  } catch {
    return false;
  }
}

export class QueueNotFoundException extends Error {
  constructor(queueName: string) {
    super(`A fila '${queueName}' não está registrada no QueueModule`);
    this.name = 'QueueNotFoundException';
  }
}

export class InvalidPayloadException extends Error {
  constructor() {
    super('O payload fornecido não é serializável em JSON');
    this.name = 'InvalidPayloadException';
  }
}

export class QueueUnavailableException extends Error {
  constructor() {
    super('O broker de filas está inacessível');
    this.name = 'QueueUnavailableException';
  }
}
```

## Notas

- O construtor recebe uma injeção por fila registrada; ao adicionar uma nova fila, o construtor e o `Map` devem ser atualizados
- O `Map` interno usa `string` como chave — não `QueueName` — porque quando `QUEUE_NAMES` está vazio, `QueueName` resolve para `never`, tornando `Map<QueueName, Queue>` inválido em TypeScript; `QueueName` é usado apenas na assinatura pública do `add()`, onde provê a verificação em tempo de compilação
- `QueueNotFoundException`, `InvalidPayloadException` e `QueueUnavailableException` devem ser exportadas para que os callers possam fazer `instanceof` no catch
