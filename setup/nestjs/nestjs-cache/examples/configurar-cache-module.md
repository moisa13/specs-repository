# Exemplo — Configurar CacheModule

## Contexto

`CacheModule` é `@Global()` e registra a conexão ioredis, o `CacheService`, o `CacheWarmingService` e o `CacheEventsService`. É importado uma única vez no `AppModule`.

---

## src/cache/cache.module.ts

```typescript
import { Global, Module } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import Redis from 'ioredis';
import { CACHE_REDIS_CLIENT } from './cache.constants';
import { CacheService } from './cache.service';
import { CacheEventsService } from './cache-events.service';
import { CacheWarmingService } from './cache-warming.service';

@Global()
@Module({
  providers: [
    {
      provide: CACHE_REDIS_CLIENT,
      inject: [ConfigService],
      useFactory: (config: ConfigService) =>
        new Redis({
          host: config.get('REDIS_HOST'),
          port: config.get('REDIS_PORT'),
          password: config.get('REDIS_PASSWORD'),
          db: config.get('REDIS_CACHE_DB'),
        }),
    },
    CacheService,
    CacheEventsService,
    CacheWarmingService,
  ],
  exports: [CacheService],
})
export class CacheModule {}
```

---

## Registro no AppModule

```typescript
import { Module } from '@nestjs/common';
import { CacheModule } from './cache/cache.module';
import { CACHE_WARMERS } from './cache/cache.constants';
import { UserCacheWarmer } from './users/user-cache.warmer';

@Module({
  imports: [CacheModule],
  providers: [
    { provide: CACHE_WARMERS, useClass: UserCacheWarmer, multi: true },
  ],
})
export class AppModule {}
```

> Warmers são registrados no `AppModule` via token `CACHE_WARMERS` com `multi: true`, permitindo que múltiplos módulos de domínio registrem seus warmers sem que nenhum deles conheça os demais.

---

## src/cache/cache.constants.ts

```typescript
export const CACHE_REDIS_CLIENT = 'CACHE_REDIS_CLIENT';
export const CACHE_WARMERS = 'CACHE_WARMERS';
```
