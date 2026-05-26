# Exemplo — Configurar CacheService

## Contexto

`CacheService` encapsula todas as operações Redis. O prefixo é aplicado internamente; o caller sempre opera sem ele.

---

## src/cache/cache.service.ts

```typescript
import { Inject, Injectable, Logger } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import Redis from 'ioredis';
import { CACHE_REDIS_CLIENT } from './cache.constants';
import { CacheEventsService } from './cache-events.service';

@Injectable()
export class CacheService {
  private readonly logger = new Logger(CacheService.name);
  private readonly enabled: boolean;
  private readonly prefix: string;
  private readonly multiplier: number;
  private readonly lockTtl: number;

  constructor(
    @Inject(CACHE_REDIS_CLIENT) private readonly redis: Redis,
    private readonly config: ConfigService,
    private readonly events: CacheEventsService,
  ) {
    this.enabled = this.config.get<boolean>('CACHE_ENABLED', true);
    this.prefix = this.config.get<string>('CACHE_KEY_PREFIX');
    this.multiplier = this.config.get<number>('CACHE_TTL_MULTIPLIER', 1);
    this.lockTtl = this.config.get<number>('CACHE_LOCK_TTL', 5000);
  }

  private key(k: string): string {
    return `${this.prefix}:${k}`;
  }

  private lockKey(k: string): string {
    return `${this.prefix}:lock:${k}`;
  }

  private effectiveTtl(ttl: number): number {
    return Math.round(ttl * this.multiplier);
  }

  async get<T>(k: string): Promise<T | null> {
    if (!this.enabled) return null;
    try {
      const raw = await this.redis.get(this.key(k));
      return raw ? (JSON.parse(raw) as T) : null;
    } catch (err) {
      this.logger.error({ message: 'Cache get failed', key: k, err });
      return null;
    }
  }

  async set(k: string, value: unknown, ttl: number): Promise<void> {
    if (!this.enabled) return;
    try {
      await this.redis.set(this.key(k), JSON.stringify(value), 'EX', this.effectiveTtl(ttl));
    } catch (err) {
      this.logger.error({ message: 'Cache set failed', key: k, err });
    }
  }

  async delete(k: string): Promise<void> {
    if (!this.enabled) return;
    try {
      await this.redis.del(this.key(k));
      this.events.publish(k);
    } catch (err) {
      this.logger.error({ message: 'Cache delete failed', key: k, err });
    }
  }

  async deletePattern(pattern: string): Promise<void> {
    if (!this.enabled) return;
    const fullPattern = `${this.prefix}:${pattern}`;
    const removed: string[] = [];
    try {
      let cursor = '0';
      do {
        const [next, keys] = await this.redis.scan(cursor, 'MATCH', fullPattern, 'COUNT', 100);
        cursor = next;
        if (keys.length > 0) {
          await this.redis.del(...keys);
          removed.push(...keys.map((k) => k.replace(`${this.prefix}:`, '')));
        }
      } while (cursor !== '0');
      if (removed.length > 0) {
        this.events.publishMany(pattern.split(':')[0], removed);
      }
    } catch (err) {
      this.logger.error({ message: 'Cache deletePattern failed', pattern, err });
    }
  }

  async remember<T>(k: string, ttl: number, fn: () => Promise<T>): Promise<T> {
    if (!this.enabled) return fn();

    const cached = await this.get<T>(k);
    if (cached !== null) return cached;

    let acquired: string | null = null;
    try {
      acquired = await this.redis.set(this.lockKey(k), '1', 'PX', this.lockTtl, 'NX');
    } catch (err) {
      this.logger.error({ message: 'Cache remember failed (lock)', key: k, err });
      return fn();
    }

    if (acquired) {
      try {
        const value = await fn();
        await this.set(k, value, ttl);
        return value;
      } finally {
        await this.redis.del(this.lockKey(k)).catch(() => undefined);
      }
    }

    await new Promise<void>((r) => setTimeout(r, 50));
    const afterWait = await this.get<T>(k);
    return afterWait ?? (await fn());
  }
}
```
