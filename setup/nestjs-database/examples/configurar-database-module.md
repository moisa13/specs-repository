# Exemplo — Configurar DatabaseModule

## Contexto

Módulo de infraestrutura responsável por configurar a conexão com o PostgreSQL via TypeORM. Registrado no `AppModule` ao lado do `ConfigModule`.

Caminho do arquivo: `src/database/database.module.ts`

---

## Código

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ConfigService } from '@nestjs/config';

@Module({
  imports: [
    TypeOrmModule.forRootAsync({
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        type: 'postgres',
        host: config.getOrThrow<string>('DB_HOST'),
        port: config.getOrThrow<number>('DB_PORT'),
        username: config.getOrThrow<string>('DB_USER'),
        password: config.getOrThrow<string>('DB_PASSWORD'),
        database: config.getOrThrow<string>('DB_NAME'),
        ssl:
          config.getOrThrow<string>('DB_SSL') === 'true'
            ? { rejectUnauthorized: false }
            : false,
        entities: [__dirname + '/../**/*.entity{.ts,.js}'],
        migrations: [__dirname + '/migrations/*{.ts,.js}'],
        migrationsRun: config.getOrThrow<string>('NODE_ENV') === 'development',
        synchronize: false,
      }),
    }),
  ],
})
export class DatabaseModule {}
```

---

## Notas

- `TypeOrmModule.forRootAsync` com `inject: [ConfigService]` garante que as variáveis já foram validadas pelo Joi antes de tentar conectar
- `synchronize: false` sempre — ver ADR-002 em `decisions.md`
- `migrationsRun` habilitado apenas em desenvolvimento — ver ADR-003 em `decisions.md`
- O glob de entidades detecta automaticamente entidades de qualquer módulo de feature — ver ADR-004 em `decisions.md`
