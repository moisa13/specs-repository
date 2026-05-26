# Exemplo — Configurar LoggingInterceptor

## Contexto

O interceptor registra método HTTP, URL, status de resposta e duração de cada requisição. É registrado globalmente como `APP_INTERCEPTOR`.

---

## src/infrastructure/logger/logging.interceptor.ts

```typescript
import { CallHandler, ExecutionContext, Injectable, Logger, NestInterceptor } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  private readonly logger = new Logger(LoggingInterceptor.name);

  intercept(context: ExecutionContext, next: CallHandler): Observable<unknown> {
    const req = context.switchToHttp().getRequest();
    const { method, url } = req;
    const start = Date.now();

    return next.handle().pipe(
      tap(() => {
        const res = context.switchToHttp().getResponse();
        const duration = Date.now() - start;
        this.logger.log(`${method} ${url} ${res.statusCode} ${duration}ms`);
      }),
    );
  }
}
```

---

## Registro no AppModule

```typescript
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

> `LoggerModule` deve estar em `imports` para que `app.get(LoggerService)` no `bootstrap()` consiga resolver o provider.
