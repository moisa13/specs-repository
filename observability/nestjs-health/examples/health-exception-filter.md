# Exemplo — HealthExceptionFilter

## Contexto

Filtro de exceção específico para o `HealthController`. Captura `HealthCheckError` antes que o `AllExceptionsFilter` global do `setup/nestjs-init` o intercepte — sem este filtro, o corpo Terminus seria substituído pelo formato de erro padrão da aplicação.

Caminho do arquivo: `src/modules/health/health.exception-filter.ts`

---

## Código

```typescript
import { ArgumentsHost, Catch, ExceptionFilter } from '@nestjs/common';
import { HealthCheckError } from '@nestjs/terminus';
import { Response } from 'express';

@Catch(HealthCheckError)
export class HealthExceptionFilter implements ExceptionFilter {
  catch(exception: HealthCheckError, host: ArgumentsHost): void {
    const response = host.switchToHttp().getResponse<Response>();
    response.status(503).json(exception.causes);
  }
}
```

---

## Notas

- `exception.causes` é o objeto `HealthCheckResult` do Terminus — contém os campos `status`, `info`, `error` e `details` exatamente no formato documentado em `contracts.md`
- Filtros aplicados via `@UseFilters()` no controller têm precedência sobre o filtro global — o `AllExceptionsFilter` não é executado para exceções capturadas aqui
- Exceções que não sejam `HealthCheckError` (ex: erros inesperados no controller) continuam sendo capturadas pelo `AllExceptionsFilter` global normalmente
