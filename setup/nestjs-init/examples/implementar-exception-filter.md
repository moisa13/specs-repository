# Exemplo — Implementar AllExceptionsFilter

## Contexto

Filtro global que intercepta todas as exceções da aplicação e normaliza a resposta para o formato definido em `contracts.md`. Registrado no bootstrap antes do `ValidationPipe`.

Caminho do arquivo: `src/common/filters/all-exceptions.filter.ts`

---

## Código

```typescript
import {
  ArgumentsHost,
  Catch,
  ExceptionFilter,
  HttpException,
  HttpStatus,
} from '@nestjs/common';
import { Response } from 'express';

const HTTP_STATUS_TO_ERROR: Record<number, string> = {
  400: 'BAD_REQUEST',
  401: 'UNAUTHORIZED',
  403: 'FORBIDDEN',
  404: 'NOT_FOUND',
  409: 'CONFLICT',
  415: 'UNSUPPORTED_MEDIA_TYPE',
  422: 'UNPROCESSABLE_ENTITY',
  429: 'TOO_MANY_REQUESTS',
  500: 'INTERNAL_SERVER_ERROR',
};

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();

    let status = HttpStatus.INTERNAL_SERVER_ERROR;
    let message = 'Internal server error';
    let details: unknown[] = [];

    if (exception instanceof HttpException) {
      status = exception.getStatus();
      const body = exception.getResponse();

      if (typeof body === 'string') {
        message = body;
      } else if (typeof body === 'object' && body !== null) {
        const bodyObj = body as Record<string, unknown>;
        message = Array.isArray(bodyObj.message)
          ? 'Validation failed'
          : (bodyObj.message as string) ?? message;
        details = Array.isArray(bodyObj.details) ? bodyObj.details : [];
      }
    }

    response.status(status).json({
      statusCode: status,
      error: HTTP_STATUS_TO_ERROR[status] ?? 'INTERNAL_SERVER_ERROR',
      message,
      details,
    });
  }
}
```

---

## Saída — erro de validação (gerado pelo ValidationPipe com exceptionFactory)

```json
{
  "statusCode": 400,
  "error": "BAD_REQUEST",
  "message": "Validation failed",
  "details": [
    { "field": "email", "message": "email must be an email" },
    { "field": "role", "message": "property role should not exist" }
  ]
}
```

## Saída — exceção não tratada

```json
{
  "statusCode": 500,
  "error": "INTERNAL_SERVER_ERROR",
  "message": "Internal server error",
  "details": []
}
```

---

## Notas

- O `details` vindo do `ValidationPipe` é populado via `exceptionFactory` configurado no bootstrap — ver `configurar-bootstrap.md`
- Stack traces nunca aparecem na resposta — erros internos retornam apenas a mensagem genérica
- Códigos HTTP não mapeados na tabela `HTTP_STATUS_TO_ERROR` caem em `'INTERNAL_SERVER_ERROR'`
