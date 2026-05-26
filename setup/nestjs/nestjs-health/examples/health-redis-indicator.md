# Exemplo — Indicador Redis no readiness (quando setup/nestjs/nestjs-queue aplicado)

## Contexto

Quando `setup/nestjs/nestjs-queue` está aplicado, o Redis é uma dependência crítica. Este exemplo mostra como adicionar o `RedisHealthIndicator` ao `/health/ready` de forma condicional.

---

## `src/modules/health/redis.health-indicator.ts`

```typescript
import { Injectable, OnModuleDestroy } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { HealthCheckError, HealthIndicator, HealthIndicatorResult } from '@nestjs/terminus';
import Redis from 'ioredis';

@Injectable()
export class RedisHealthIndicator extends HealthIndicator implements OnModuleDestroy {
  private readonly client: Redis | null = null;

  constructor(private readonly config: ConfigService) {
    super();
    const host = this.config.get<string>('REDIS_HOST');
    if (host) {
      this.client = new Redis({
        host,
        port: this.config.get<number>('REDIS_PORT', 6379),
        password: this.config.get<string>('REDIS_PASSWORD'),
        db: this.config.get<number>('REDIS_DB', 0),
        lazyConnect: true,
      });
    }
  }

  async pingCheck(key: string): Promise<HealthIndicatorResult> {
    try {
      await this.client!.ping();
      return this.getStatus(key, true);
    } catch (error) {
      const result = this.getStatus(key, false, { message: (error as Error).message });
      throw new HealthCheckError('Redis health check failed', result);
    }
  }

  onModuleDestroy(): void {
    this.client?.disconnect();
  }
}
```

---

## `src/modules/health/health.module.ts` (atualizado)

```typescript
import { Module } from '@nestjs/common';
import { TerminusModule } from '@nestjs/terminus';
import { HealthController } from './health.controller';
import { RedisHealthIndicator } from './redis.health-indicator';

@Module({
  imports: [TerminusModule],
  controllers: [HealthController],
  providers: [RedisHealthIndicator],
})
export class HealthModule {}
```

---

## `src/modules/health/health.controller.ts` — método `ready()` atualizado

```typescript
constructor(
  private readonly health: HealthCheckService,
  private readonly db: TypeOrmHealthIndicator,
  private readonly memory: MemoryHealthIndicator,
  private readonly redis: RedisHealthIndicator,
  private readonly config: ConfigService,
) {}

ready() {
  const heapThreshold = this.config.getOrThrow<number>('HEALTH_MEMORY_HEAP_THRESHOLD_MB') * 1024 * 1024;
  const rssThreshold = this.config.getOrThrow<number>('HEALTH_MEMORY_RSS_THRESHOLD_MB') * 1024 * 1024;

  const indicators = [
    () => this.db.pingCheck('database'),
    () => this.memory.checkHeap('memory_heap', heapThreshold),
    () => this.memory.checkRSS('memory_rss', rssThreshold),
  ];

  if (this.config.get<string>('REDIS_HOST')) {
    indicators.push(() => this.redis.pingCheck('redis'));
  }

  return this.health.check(indicators);
}
```

---

## Notas

- `lazyConnect: true` evita que o ioredis tente conectar imediatamente na inicialização; a conexão é estabelecida no primeiro `ping()`
- `onModuleDestroy` garante que a conexão seja encerrada graciosamente quando a aplicação para
- A condicional `config.get('REDIS_HOST')` no controller determina em runtime se o indicador é incluído — se `REDIS_HOST` não estiver no ambiente, o indicador não é chamado mesmo que `RedisHealthIndicator` esteja registrado
