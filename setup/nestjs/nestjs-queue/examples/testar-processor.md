# Exemplo — Testar Processor e QueueService

## Contexto

Processors dependem de `@nestjs/bullmq` e recebem objetos `Job` criados pelo BullMQ em runtime. Testar unitariamente exige criar um `Job` mínimo e chamar `process()` diretamente, sem subir o broker. Testar o `QueueService` exige mockar as filas injetadas via `@InjectQueue`.

---

## Testando um Processor (unitário)

```typescript
import { Test } from '@nestjs/testing';
import { Job } from 'bullmq';
import { SendEmailProcessor } from './send-email.processor';
import { EmailService } from '../email.service';

describe('SendEmailProcessor', () => {
  let processor: SendEmailProcessor;
  let emailService: jest.Mocked<EmailService>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        SendEmailProcessor,
        { provide: EmailService, useValue: { send: jest.fn() } },
      ],
    }).compile();

    processor = module.get(SendEmailProcessor);
    emailService = module.get(EmailService);
  });

  it('executa send-email', async () => {
    const job = { name: 'send-email', data: { to: 'user@example.com', subject: 'Bem-vindo', templateId: 'welcome' } } as Job;

    await processor.process(job);

    expect(emailService.send).toHaveBeenCalledWith(job.data);
  });

  it('lança erro para jobType desconhecido', async () => {
    const job = { name: 'tipo-inexistente', data: {} } as Job;

    await expect(processor.process(job)).rejects.toThrow('Tipo de job desconhecido');
  });
});
```

---

## Testando o QueueService (unitário)

O `QueueService` recebe as filas via `@InjectQueue`. Para mockar, usar o token gerado por `getQueueToken` do `@nestjs/bullmq`.

```typescript
import { Test } from '@nestjs/testing';
import { getQueueToken } from '@nestjs/bullmq';
import { QueueService, QueueNotFoundException, InvalidPayloadException } from './queue.service';
import { QUEUE_NAMES } from './queue.constants';

const mockQueue = { add: jest.fn() };

describe('QueueService', () => {
  let service: QueueService;

  beforeEach(async () => {
    mockQueue.add.mockReset();

    const module = await Test.createTestingModule({
      providers: [
        QueueService,
        { provide: getQueueToken(QUEUE_NAMES.NOTIFICATIONS), useValue: mockQueue },
        { provide: getQueueToken(QUEUE_NAMES.REPORTS), useValue: mockQueue },
      ],
    }).compile();

    service = module.get(QueueService);
  });

  it('enfileira o job e retorna jobId', async () => {
    mockQueue.add.mockResolvedValue({ id: 'job-123' });

    const result = await service.add(QUEUE_NAMES.NOTIFICATIONS, 'send-email', { to: 'u@example.com' });

    expect(result.jobId).toBe('job-123');
    expect(result.queueName).toBe(QUEUE_NAMES.NOTIFICATIONS);
    expect(result.status).toBe('waiting');
  });

  it('lança QueueNotFoundException para fila não registrada', async () => {
    await expect(
      service.add('fila-inexistente' as any, 'tipo', {}),
    ).rejects.toThrow(QueueNotFoundException);
  });

  it('lança InvalidPayloadException para payload não serializável', async () => {
    const circular: Record<string, unknown> = {};
    circular.self = circular;

    await expect(
      service.add(QUEUE_NAMES.NOTIFICATIONS, 'send-email', circular),
    ).rejects.toThrow(InvalidPayloadException);
  });

  it('retorna status delayed quando options.delay > 0', async () => {
    mockQueue.add.mockResolvedValue({ id: 'job-456' });

    const result = await service.add(QUEUE_NAMES.NOTIFICATIONS, 'send-email', {}, { delay: 5000 });

    expect(result.status).toBe('delayed');
  });
});
```

---

## Notas

- `{ name: '...', data: {...} } as Job` é suficiente para testes unitários de Processor; propriedades extras do `Job` (id, timestamp, etc.) não são necessárias salvo se o método `process()` as usar
- `getQueueToken(QUEUE_NAMES.X)` é o token DI registrado pelo `BullModule.registerQueue` — usar este token é obrigatório; `@InjectQueue` internamente resolve pelo mesmo token
- Testes de integração que exigem Redis real devem provisionar uma instância isolada (container Docker ou equivalente) no ambiente de CI; a estratégia de infraestrutura de testes está fora do escopo desta spec
