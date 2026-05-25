# Exemplo — HealthController

## Contexto

Controller com dois endpoints: `/health/live` (liveness) e `/health/ready` (readiness). Lê os limites de memória do `ConfigService`.

Caminho do arquivo: `src/modules/health/health.controller.ts`

---

## Código

```typescript
import { Controller, Get, UseFilters } from '@nestjs/common';
import {
  HealthCheck,
  HealthCheckService,
  MemoryHealthIndicator,
  TypeOrmHealthIndicator,
} from '@nestjs/terminus';
import { ConfigService } from '@nestjs/config';
import {
  ApiExtraModels,
  ApiOkResponse,
  ApiOperation,
  ApiProperty,
  ApiServiceUnavailableResponse,
  ApiTags,
  getSchemaPath,
} from '@nestjs/swagger';
import { HealthExceptionFilter } from './health.exception-filter';

class HealthIndicatorResultDto {
  @ApiProperty({ example: 'up' })
  status: string;

  @ApiProperty({ required: false, example: 'connect ECONNREFUSED 127.0.0.1:5432' })
  message?: string;
}

class HealthCheckResponseDto {
  @ApiProperty({ example: 'ok' })
  status: string;

  @ApiProperty({
    type: 'object',
    additionalProperties: { $ref: getSchemaPath(HealthIndicatorResultDto) },
  })
  info: Record<string, HealthIndicatorResultDto>;

  @ApiProperty({
    type: 'object',
    additionalProperties: { $ref: getSchemaPath(HealthIndicatorResultDto) },
  })
  error: Record<string, HealthIndicatorResultDto>;

  @ApiProperty({
    type: 'object',
    additionalProperties: { $ref: getSchemaPath(HealthIndicatorResultDto) },
  })
  details: Record<string, HealthIndicatorResultDto>;
}

@ApiTags('health')
@UseFilters(HealthExceptionFilter)
@ApiExtraModels(HealthIndicatorResultDto, HealthCheckResponseDto)
@Controller('health')
export class HealthController {
  constructor(
    private readonly health: HealthCheckService,
    private readonly db: TypeOrmHealthIndicator,
    private readonly memory: MemoryHealthIndicator,
    private readonly config: ConfigService,
  ) {}

  @Get('live')
  @HealthCheck()
  @ApiOperation({ summary: 'Liveness probe' })
  @ApiOkResponse({ type: HealthCheckResponseDto, description: 'Processo ativo' })
  live() {
    return this.health.check([
      () => Promise.resolve({ live: { status: 'up' as const } }),
    ]);
  }

  @Get('ready')
  @HealthCheck()
  @ApiOperation({ summary: 'Readiness probe' })
  @ApiOkResponse({ type: HealthCheckResponseDto, description: 'Todos os indicadores saudáveis' })
  @ApiServiceUnavailableResponse({ description: 'Um ou mais indicadores com falha' })
  ready() {
    const heapThreshold =
      this.config.getOrThrow<number>('HEALTH_MEMORY_HEAP_THRESHOLD_MB') * 1024 * 1024;
    const rssThreshold =
      this.config.getOrThrow<number>('HEALTH_MEMORY_RSS_THRESHOLD_MB') * 1024 * 1024;

    return this.health.check([
      () => this.db.pingCheck('database'),
      () => this.memory.checkHeap('memory_heap', heapThreshold),
      () => this.memory.checkRSS('memory_rss', rssThreshold),
    ]);
  }
}
```

---

## Notas

- `@UseFilters(HealthExceptionFilter)` aplicado no controller captura `HealthCheckError` antes do `AllExceptionsFilter` global, preservando o formato Terminus na resposta 503
- `status: 'up' as const` previne o widening do literal para `string` — sem `as const`, o TypeScript infere `string`, que não é atribuível ao tipo `'up' | 'down'` esperado pelo Terminus
- O endpoint `/health/live` retorna um indicador estático — não verifica nenhuma dependência externa
- `pingCheck('database')` reutiliza a conexão existente do `DatabaseModule` — não abre nova conexão
- Os thresholds são convertidos de MB para bytes na multiplicação por `1024 * 1024`
- Os decorators Swagger não alteram o comportamento do endpoint — são apenas metadados para geração da documentação
