# Exemplo — Configurar data-source.ts

## Contexto

Arquivo usado exclusivamente pelo TypeORM CLI para gerar e executar migrations fora do contexto NestJS. Não é importado pela aplicação.

Caminho do arquivo: `src/infrastructure/database/data-source.ts`

---

## Código

```typescript
import { DataSource } from 'typeorm';
import * as dotenv from 'dotenv';

dotenv.config();

export default new DataSource({
  type: 'postgres',
  host: process.env.DB_HOST,
  port: Number(process.env.DB_PORT),
  username: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  ssl:
    process.env.DB_SSL === 'true' ? { rejectUnauthorized: false } : false,
  entities: ['src/**/*.entity.ts'],
  migrations: ['src/infrastructure/database/migrations/*.ts'],
  synchronize: false,
});
```

---

## Notas

- `dotenv.config()` carrega o `.env` manualmente — fora do NestJS, o `ConfigModule` não está disponível
- `entities` aponta para fontes TypeScript (`*.entity.ts`) — o CLI opera no código-fonte, não no compilado
- `migrations` aponta para fontes TypeScript (`*.ts`) — o CLI gera e lê arquivos `.ts`, não `.js`
- Este arquivo não deve ser importado por nenhum módulo NestJS
