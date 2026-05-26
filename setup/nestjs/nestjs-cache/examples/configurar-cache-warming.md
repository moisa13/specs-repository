# Exemplo — Configurar CacheWarmingService e CacheWarmer

## Contexto

`CacheWarmingService` executa no boot todos os warmers registrados via `CACHE_WARMERS`. Cada módulo de domínio implementa `CacheWarmer` para registrar seus recursos a pré-aquecer.

---

## src/cache/interfaces/cache-warmer.interface.ts

```typescript
export interface CacheWarmer {
  warm(): Promise<void>;
}
```

---

## src/cache/cache-warming.service.ts

```typescript
import { Inject, Injectable, Logger, OnApplicationBootstrap, Optional } from '@nestjs/common';
import { CACHE_WARMERS } from './cache.constants';
import { CacheWarmer } from './interfaces/cache-warmer.interface';

@Injectable()
export class CacheWarmingService implements OnApplicationBootstrap {
  private readonly logger = new Logger(CacheWarmingService.name);

  constructor(
    @Optional() @Inject(CACHE_WARMERS) private readonly warmers: CacheWarmer[] = [],
  ) {}

  async onApplicationBootstrap(): Promise<void> {
    for (const warmer of this.warmers) {
      try {
        await warmer.warm();
      } catch (err) {
        this.logger.error({ message: 'Cache warmer failed', warmer: warmer.constructor.name, err });
      }
    }
  }
}
```

> `@Optional()` garante que a aplicação suba mesmo quando nenhum warmer estiver registrado — o array será `undefined` e o default `[]` é aplicado.

---

## Implementando um warmer em módulo de domínio

```typescript
import { Injectable } from '@nestjs/common';
import { CacheWarmer } from 'src/cache/interfaces/cache-warmer.interface';
import { CacheService } from 'src/cache/cache.service';
import { UserRepository } from './user.repository';

const USER_TTL = 300;

@Injectable()
export class UserCacheWarmer implements CacheWarmer {
  constructor(
    private readonly cache: CacheService,
    private readonly users: UserRepository,
  ) {}

  async warm(): Promise<void> {
    const users = await this.users.findAll();
    await Promise.all(users.map((u) => this.cache.set(`users:${u.id}`, u, USER_TTL)));
  }
}
```
