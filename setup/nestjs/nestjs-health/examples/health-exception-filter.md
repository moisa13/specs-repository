# Exemplo — HealthExceptionFilter

## Contexto

Filtro de exceção específico para o `HealthController`. Captura `ServiceUnavailableException` (lançada pelo `@nestjs/terminus` 11 quando qualquer indicador falha) antes que o `AllExceptionsFilter` global do `setup/nestjs/nestjs-init` o intercepte — sem este filtro, o payload Terminus seria substituído pelo formato de erro padrão da aplicação.

Caminho do arquivo: `src/modules/health/health.exception-filter.ts`

---

## Código

```typescript
import { ArgumentsHost, Catch, ExceptionFilter, ServiceUnavailableException } from '@nestjs/common';
import { Response } from 'express';

@Catch(ServiceUnavailableException)
export class HealthExceptionFilter implements ExceptionFilter {
  catch(exception: ServiceUnavailableException, host: ArgumentsHost): void {
    const response = host.switchToHttp().getResponse<Response>();
    response.status(503).json(exception.getResponse());
  }
}
```

---

## Notas

- Em `@nestjs/terminus` 11, `HealthCheckService` lança `ServiceUnavailableException` com o `HealthCheckResult` do Terminus como corpo — `exception.getResponse()` retorna esse objeto intacto, com os campos `status`, `info`, `error` e `details` exatamente no formato documentado em `contracts.md`
- Filtros aplicados via `@UseFilters()` no controller têm precedência sobre o filtro global — o `AllExceptionsFilter` não é executado para exceções capturadas aqui
- Como este filtro é registrado apenas no `HealthController`, capturar `ServiceUnavailableException` não afeta outros controllers — erros 503 de outras origens são tratados normalmente pelo `AllExceptionsFilter`
- Exceções que não sejam `ServiceUnavailableException` (ex: erros inesperados no controller) continuam sendo capturadas pelo `AllExceptionsFilter` global normalmente
