# Exemplo — Registrar HealthModule no AppModule

## Contexto

O `HealthModule` é registrado no `AppModule` como módulo de infraestrutura, após o `ConfigModule` e o `DatabaseModule`.

Caminho do arquivo: `src/app.module.ts`

---

## Código

```typescript
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { AppInfoModule } from './modules/app-info/app-info.module';
import { DatabaseModule } from './infrastructure/database/database.module';
import { HealthModule } from './modules/health/health.module';
import { envValidationSchema } from './config/env.validation';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      validationSchema: envValidationSchema,
    }),
    DatabaseModule,
    AppInfoModule,
    HealthModule,
  ],
})
export class AppModule {}
```

---

## Notas

- `HealthModule` entra após `DatabaseModule` — o `TypeOrmHealthIndicator` depende da conexão já registrada
- `AppInfoModule` mantém sua posição anterior à spec de health — `HealthModule` é acrescentado depois, preservando a ordem progressiva de aplicação das specs
- A ordem `ConfigModule → DatabaseModule → AppInfoModule → HealthModule` garante que o `ConfigService` e a conexão TypeORM estejam disponíveis quando o Terminus inicializar os indicadores
