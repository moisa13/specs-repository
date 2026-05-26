# Exemplo — Configurar GenericCacheService

## Contexto

`GenericCacheService` é uma classe abstrata que módulos de domínio estendem. Padroniza a construção de chaves e delega as operações ao `CacheService`.

---

## src/cache/generic-cache.service.ts

```typescript
import { Inject } from '@nestjs/common';
import { createHash } from 'crypto';
import { CacheService } from './cache.service';

export abstract class GenericCacheService {
  constructor(
    @Inject(CacheService) protected readonly cache: CacheService,
  ) {}

  protected hashParams(params: Record<string, unknown>): string {
    const stable = JSON.stringify(
      Object.keys(params).sort().reduce<Record<string, unknown>>((acc, k) => {
        acc[k] = params[k];
        return acc;
      }, {}),
    );
    return createHash('sha256').update(stable).digest('hex').slice(0, 8);
  }

  async rememberById<T>(resource: string, id: string | number, ttl: number, fn: () => Promise<T>): Promise<T> {
    return this.cache.remember(`${resource}:${id}`, ttl, fn);
  }

  async rememberList<T>(resource: string, params: Record<string, unknown>, ttl: number, fn: () => Promise<T>): Promise<T> {
    const hash = this.hashParams(params);
    return this.cache.remember(`${resource}:list:${hash}`, ttl, fn);
  }

  async invalidateById(resource: string, id: string | number): Promise<void> {
    await this.cache.delete(`${resource}:${id}`);
  }

  async invalidateAll(resource: string): Promise<void> {
    await this.cache.deletePattern(`${resource}:*`);
  }
}
```

---

## Uso em módulo de domínio

```typescript
import { Injectable } from '@nestjs/common';
import { GenericCacheService } from 'src/cache/generic-cache.service';
import { CacheService } from 'src/cache/cache.service';

const USER_TTL = 300;

@Injectable()
export class UserCacheService extends GenericCacheService {
  constructor(cache: CacheService) {
    super(cache);
  }

  async getUser(id: number): Promise<User> {
    return this.rememberById('users', id, USER_TTL, () => this.db.findUser(id));
  }

  async invalidateUser(id: number): Promise<void> {
    await this.invalidateById('users', id);
  }
}
```

> `UserCacheService` define os métodos do seu domínio e delega toda a lógica de chave, TTL e invalidação para `GenericCacheService`. O TTL específico do recurso fica no módulo de domínio.
