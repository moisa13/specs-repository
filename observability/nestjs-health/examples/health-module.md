# Exemplo — HealthModule

## Contexto

Módulo de observabilidade que registra o controller de health check. Não exporta nenhum provider e não é global.

Caminho do arquivo: `src/health/health.module.ts`

---

## Código

```typescript
import { Module } from '@nestjs/common';
import { TerminusModule } from '@nestjs/terminus';
import { HealthController } from './health.controller';

@Module({
  imports: [TerminusModule],
  controllers: [HealthController],
})
export class HealthModule {}
```
