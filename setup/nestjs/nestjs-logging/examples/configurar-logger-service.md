# Exemplo — Configurar LoggerService

## Contexto

`LoggerService` implementa a interface `LoggerService` do NestJS e configura os transportes Winston. É singleton e substitui o logger padrão do framework.

---

## src/logger/logger.service.ts

```typescript
import { Injectable, LoggerService as NestLoggerService } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import * as winston from 'winston';
import 'winston-daily-rotate-file';
import { LogSanitizer } from './log-sanitizer';

@Injectable()
export class LoggerService implements NestLoggerService {
  private readonly logger: winston.Logger;

  constructor(
    private readonly sanitizer: LogSanitizer,
    private readonly config: ConfigService,
  ) {
    const logDir = this.config.get<string>('LOG_DIR', 'logs/');
    const logLevel = this.config.get<string>('LOG_LEVEL', 'warn');
    const maxSize = this.config.get<string>('LOG_MAX_SIZE', '20m');
    const maxFiles = this.config.get<string>('LOG_MAX_FILES', '14d');
    const isProduction = process.env.NODE_ENV === 'production';

    const transports: winston.transport[] = [
      new winston.transports.DailyRotateFile({
        dirname: logDir,
        filename: '%DATE%.log',
        datePattern: 'YYYY-MM-DD',
        maxSize,
        maxFiles,
        zippedArchive: true,
        format: winston.format.combine(
          winston.format.timestamp(),
          winston.format.json(),
        ),
      }),
      new winston.transports.DailyRotateFile({
        dirname: logDir,
        filename: '%DATE%-error.log',
        datePattern: 'YYYY-MM-DD',
        level: 'error',
        maxSize,
        maxFiles,
        zippedArchive: true,
        format: winston.format.combine(
          winston.format.timestamp(),
          winston.format.json(),
        ),
      }),
    ];

    if (!isProduction) {
      transports.push(
        new winston.transports.Console({
          format: winston.format.combine(
            winston.format.colorize(),
            winston.format.timestamp(),
            winston.format.simple(),
          ),
        }),
      );
    }

    this.logger = winston.createLogger({ level: logLevel, transports });
  }

  private buildEntry(message: unknown, context?: string): Record<string, unknown> {
    return {
      message: this.sanitizer.sanitize(message),
      ...(context !== undefined ? { context } : {}),
    };
  }

  log(message: unknown, context?: string): void {
    this.logger.info(this.buildEntry(message, context));
  }

  error(message: unknown, stack?: string, context?: string): void {
    this.logger.error({ ...this.buildEntry(message, context), stack });
  }

  warn(message: unknown, context?: string): void {
    this.logger.warn(this.buildEntry(message, context));
  }

  debug(message: unknown, context?: string): void {
    this.logger.debug(this.buildEntry(message, context));
  }

  verbose(message: unknown, context?: string): void {
    this.logger.verbose(this.buildEntry(message, context));
  }
}
```

> O `timestamp` é adicionado pelo `winston.format.timestamp()` antes de `winston.format.json()`, garantindo que o campo esteja presente em toda entrada gravada nos arquivos — conforme contrato de `LogEntry` em `observability/structured-log`.

> O `context` é incluído na entrada apenas quando fornecido pelo caller. Ao usar `new Logger(NomeDaClasse.name)` em qualquer classe, o NestJS passa o nome da classe como `context` automaticamente — garantindo que o campo obrigatório esteja sempre presente no uso convencional.

---

## Registro no bootstrap

```typescript
const app = await NestFactory.create(AppModule, { bufferLogs: true });
app.useLogger(app.get(LoggerService));
```

> `bufferLogs: true` garante que logs emitidos durante o bootstrap sejam armazenados e redirecionados para o `LoggerService` assim que ele for registrado.
