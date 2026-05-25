# Exemplo — Configurar bootstrap da aplicação

## Contexto

Ponto de entrada da aplicação. Registra o filtro de exceção global, o `ValidationPipe` com `exceptionFactory` customizado, CORS e Swagger. Nenhuma lógica de negócio deve existir aqui.

Caminho do arquivo: `src/main.ts`

---

## Código

```typescript
import {
  BadRequestException,
  INestApplication,
  ValidationPipe,
} from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { NestFactory } from '@nestjs/core';
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';
import packageJson from '../package.json';
import { AppModule } from './app.module';
import { AllExceptionsFilter } from './common/filters/all-exceptions.filter';

function setupSwagger(app: INestApplication, configService: ConfigService): void {
  const config = new DocumentBuilder()
    .setTitle(configService.getOrThrow<string>('APP_NAME'))
    .setDescription(configService.getOrThrow<string>('APP_DESCRIPTION'))
    .setVersion(packageJson.version)
    .build();
  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('docs', app, document);
}

async function bootstrap(): Promise<void> {
  const app = await NestFactory.create(AppModule);
  const configService = app.get(ConfigService);

  app.useGlobalFilters(new AllExceptionsFilter());

  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
      exceptionFactory: (errors) => {
        const details = errors.flatMap((error) =>
          Object.entries(error.constraints ?? {}).map(([, message]) => ({
            field: error.property,
            message,
          })),
        );
        return new BadRequestException({
          message: 'Validation failed',
          details,
        });
      },
    }),
  );

  const corsOrigins = configService
    .getOrThrow<string>('CORS_ORIGIN')
    .split(',')
    .map((s) => s.trim());
  if (configService.getOrThrow<string>('NODE_ENV') === 'production' && corsOrigins.includes('*')) {
    console.warn('[WARN] CORS_ORIGIN=* em produção não é recomendado.');
  }
  app.enableCors({ origin: corsOrigins });

  if (configService.getOrThrow<string>('NODE_ENV') !== 'production') {
    setupSwagger(app, configService);
  }

  const port = configService.getOrThrow<number>('PORT');
  await app.listen(port);
  console.log(`Application running on: http://localhost:${port}`);
  if (configService.getOrThrow<string>('NODE_ENV') !== 'production') {
    console.log(`Swagger available at: http://localhost:${port}/docs`);
  }
}

void bootstrap();
```

---

## Saída esperada no console (desenvolvimento)

```
Application running on: http://localhost:3000
Swagger available at: http://localhost:3000/docs
```

---

## Notas

- O `AllExceptionsFilter` deve ser registrado **antes** do `ValidationPipe` para capturar erros do pipe corretamente
- O `exceptionFactory` do `ValidationPipe` popula o campo `details` no formato de `contracts.md`; o `AllExceptionsFilter` apenas repassa esse campo — ver `implementar-exception-filter.md`
- A importação de `package.json` requer `resolveJsonModule: true` no `tsconfig.json` — opção contratada em `contracts.md`
- `configService.getOrThrow` é o padrão para leitura de variáveis de ambiente em todo o projeto — nunca usar `process.env` diretamente fora do bootstrap, e mesmo aqui `ConfigService` é o padrão
