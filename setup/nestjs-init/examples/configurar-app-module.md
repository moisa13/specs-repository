# Exemplo — Configurar AppModule

## Contexto

Módulo raiz da aplicação. Contém o `ConfigModule` global e o `AppInfoModule` (rota `GET /`). Módulos de infraestrutura adicionais (banco de dados, logging) e módulos de domínio são registrados aqui à medida que o projeto cresce.

Caminho do arquivo: `src/app.module.ts`

---

## Código

```typescript
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { AppInfoModule } from './app-info/app-info.module';
import { envValidationSchema } from './config/env.validation';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      validationSchema: envValidationSchema,
    }),
    AppInfoModule,
  ],
})
export class AppModule {}
```

---

## Notas

- `isGlobal: true` torna o `ConfigService` disponível em todos os módulos sem imports adicionais — ver ADR-002 em `decisions.md`
- O `AppModule` não deve ter `controllers`, `providers` ou `exports` definidos diretamente nele
- `AppInfoModule` é um módulo de infraestrutura — ver `examples/implementar-app-info-module.md`
- Ao adicionar o primeiro módulo de domínio (ex: `UsersModule`), ele entra no array `imports` ao lado dos demais
