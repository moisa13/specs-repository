# Exemplo — Configurar LogSanitizer

## Contexto

`LogSanitizer` é um serviço injetável que aplica as regras de sanitização de `observability/structured-log`. Retorna sempre uma cópia sanitizada sem modificar o objeto original.

---

## src/logger/log-sanitizer.ts

```typescript
import { Injectable } from '@nestjs/common';

const EXACT_NAMES = new Set([
  'password', 'cpf', 'token', 'accessToken', 'refreshToken',
  'secret', 'cvv', 'otp', 'pin', 'ssn', 'privateKey',
]);

const PATTERN_REGEX = /auth|token|secret|credential|apikey|api_key/i;
const URL_QUERY_REGEX = /([?&])([\w-]*(?:auth|token|secret|credential|apikey|api_key)[\w-]*)=([^&]*)/gi;

const REDACTED = '[REDACTED]';
const MAX_DEPTH = 5;

@Injectable()
export class LogSanitizer {
  sanitize<T>(value: T, depth = 0): T {
    if (depth >= MAX_DEPTH || value === null || value === undefined) {
      return value;
    }

    if (typeof value === 'string') {
      return value.replace(URL_QUERY_REGEX, `$1$2=${REDACTED}`) as unknown as T;
    }

    if (Array.isArray(value)) {
      return value.map((item) => this.sanitize(item, depth + 1)) as unknown as T;
    }

    if (typeof value === 'object') {
      const result: Record<string, unknown> = {};
      for (const [key, val] of Object.entries(value as Record<string, unknown>)) {
        if (EXACT_NAMES.has(key) || PATTERN_REGEX.test(key)) {
          result[key] = REDACTED;
        } else {
          result[key] = this.sanitize(val, depth + 1);
        }
      }
      return result as unknown as T;
    }

    return value;
  }
}
```

---

## Uso no HttpExceptionFilter

```typescript
const sanitizedBody = this.sanitizer.sanitize(request.body);
this.logger.warn(
  { message: 'Validation failed', body: sanitizedBody },
  HttpExceptionFilter.name,
);
```

> `message` é `unknown`, portanto aceita um objeto. O campo `body` com o conteúdo sanitizado compõe a entrada de log; `HttpExceptionFilter.name` é passado como segundo argumento — o parâmetro `context` da assinatura `warn(message, context?)`.
