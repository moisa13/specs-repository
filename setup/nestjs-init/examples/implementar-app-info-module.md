# Exemplo — Implementar AppInfoModule

## Contexto

Módulo de infraestrutura responsável pela rota `GET /`. Retorna metadados da aplicação em tempo real: nome, versão, descrição, timestamp e uptime. Registrado no `AppModule` ao lado do `ConfigModule`.

Caminhos dos arquivos:
- `src/app-info/app-info.module.ts`
- `src/app-info/app-info.controller.ts`

---

## Código — app-info.module.ts

```typescript
import { Module } from '@nestjs/common';
import { AppInfoController } from './app-info.controller';

@Module({
  controllers: [AppInfoController],
})
export class AppInfoModule {}
```

---

## Código — app-info.controller.ts

```typescript
import { Controller, Get } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { ApiOperation, ApiTags } from '@nestjs/swagger';
import packageJson from '../../package.json';

interface AppInfoResponse {
  name: string;
  version: string;
  description: string;
  timestamp: string;
  uptime: number;
}

@ApiTags('app-info')
@Controller()
export class AppInfoController {
  constructor(private readonly configService: ConfigService) {}

  @Get()
  @ApiOperation({ summary: 'Retorna metadados da aplicação' })
  getInfo(): AppInfoResponse {
    return {
      name: this.configService.getOrThrow<string>('APP_NAME'),
      version: packageJson.version,
      description: this.configService.getOrThrow<string>('APP_DESCRIPTION'),
      timestamp: new Date().toISOString(),
      uptime: process.uptime(),
    };
  }
}
```

---

## Registro no AppModule

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

## Saída esperada — GET /

```json
{
  "name": "minha-api",
  "version": "1.0.0",
  "description": "API do minha-api",
  "timestamp": "2026-05-25T14:00:00.000Z",
  "uptime": 12.453821092
}
```

---

## Notas

- `version` vem do campo `version` do `package.json` — requer `resolveJsonModule: true` no `tsconfig.json`
- `description` vem de `APP_DESCRIPTION` (variável de ambiente), não do campo `description` do `package.json`
- `ConfigService` é injetado via construtor — o `ConfigModule` com `isGlobal: true` torna isso possível sem imports adicionais no `AppInfoModule`
- O controller não usa `@Res()` do Express — a resposta é retornada diretamente pelo método, delegando a serialização ao NestJS
