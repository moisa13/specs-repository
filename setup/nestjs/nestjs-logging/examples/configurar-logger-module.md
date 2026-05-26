# Exemplo — Configurar LoggerModule

## Contexto

`LoggerModule` é decorado com `@Global()` para que `LoggerService` e `LogSanitizer` sejam injetáveis em qualquer parte da aplicação — incluindo filtros e guards globais — sem que cada módulo precise importá-lo explicitamente. É importado uma única vez no `AppModule`.

---

## src/logger/logger.module.ts

```typescript
import { Global, Module } from '@nestjs/common';
import { LoggerService } from './logger.service';
import { LogSanitizer } from './log-sanitizer';
import { LoggingInterceptor } from './logging.interceptor';

@Global()
@Module({
  providers: [LoggerService, LogSanitizer, LoggingInterceptor],
  exports: [LoggerService, LogSanitizer],
})
export class LoggerModule {}
```

> `LoggingInterceptor` está em `providers` mas não em `exports` — ele é registrado pelo `AppModule` via `APP_INTERCEPTOR`, não injetado diretamente por outros módulos.

---

## Registro no AppModule

```typescript
import { Module } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';
import { LoggerModule } from './logger/logger.module';
import { LoggingInterceptor } from './logger/logging.interceptor';

@Module({
  imports: [LoggerModule],
  providers: [
    { provide: APP_INTERCEPTOR, useClass: LoggingInterceptor },
  ],
})
export class AppModule {}
```

> Com `@Global()`, qualquer módulo de feature pode injetar `LoggerService` ou `LogSanitizer` sem declarar `LoggerModule` em seus próprios `imports`.
