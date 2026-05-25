# Exemplo — Registrar DatabaseModule no AppModule

## Contexto

O `DatabaseModule` é registrado no `AppModule` como módulo de infraestrutura, após o `ConfigModule`. Substitui a versão sem banco do `AppModule` definida em `setup/nestjs-init`.

Caminho do arquivo: `src/app.module.ts`

---

## Código

```typescript
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { AppInfoModule } from './modules/app-info/app-info.module';
import { DatabaseModule } from './database/database.module';
import { envValidationSchema } from './config/env.validation';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      validationSchema: envValidationSchema,
    }),
    DatabaseModule,
    AppInfoModule,
  ],
})
export class AppModule {}
```

---

## Notas

- A ordem importa: `ConfigModule` deve vir antes de `DatabaseModule` para que o `ConfigService` esteja disponível quando o `TypeOrmModule.forRootAsync` for inicializado
- `DatabaseModule` entra como módulo de infraestrutura — ao lado do `ConfigModule`, antes dos módulos de domínio
