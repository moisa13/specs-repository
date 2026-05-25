# Contracts — NestJS — Configuração de Banco de Dados

> Contratos de estrutura, configuração e variáveis de ambiente. O agente deve seguir estes contratos ao configurar o banco.

---

## Estrutura de diretórios

```
src/
└── database/
    ├── database.module.ts
    ├── data-source.ts
    └── migrations/
```

---

## Pacotes a instalar

| Pacote | Versão mínima | Comando de instalação |
|--------|---------------|-----------------------|
| `@nestjs/typeorm` | ^10.0.0 | `pnpm add @nestjs/typeorm` |
| `typeorm` | ^0.3.0 | `pnpm add typeorm@^0.3.0` |
| `pg` | ^8.0.0 | `pnpm add pg` |
| `dotenv` | ^16.0.0 | `pnpm add dotenv` |

> O `typeorm` resolve para `1.x` se instalado sem pin de versão (`pnpm add typeorm`). A versão `^0.3.0` é obrigatória — usar o comando exato da tabela acima.

> O pnpm usa isolamento estrito de `node_modules`: apenas dependências declaradas diretamente no `package.json` do projeto são importáveis. O `dotenv` é dependência direta do `@nestjs/config`, não do projeto — portanto o `import * as dotenv from 'dotenv'` no `data-source.ts` falha em tempo de execução e no lint sem a instalação explícita.

---

## Variáveis de ambiente

As variáveis abaixo devem ser adicionadas ao schema Joi em `src/config/env.validation.ts` e ao `.env.example`.

| Variável | Tipo | Obrigatória | Validação Joi | Descrição |
|----------|------|-------------|---------------|-----------|
| `DB_HOST` | string | ✅ | `Joi.string().required()` | Host do servidor PostgreSQL |
| `DB_PORT` | number | ✅ | `Joi.number().integer().min(1).max(65535).required()` | Porta do servidor PostgreSQL |
| `DB_USER` | string | ✅ | `Joi.string().required()` | Usuário do banco |
| `DB_PASSWORD` | string | ✅ | `Joi.string().required()` | Senha do banco |
| `DB_NAME` | string | ✅ | `Joi.string().required()` | Nome do banco de dados |
| `DB_SSL` | string | ❌ | `Joi.string().valid('true', 'false').default('false')` | Habilita SSL na conexão; usar `true` em produção |

Adições ao `.env.example`:

```dotenv
# Host do servidor PostgreSQL
DB_HOST=localhost

# Porta do servidor PostgreSQL
DB_PORT=5432

# Usuário do banco de dados
DB_USER=postgres

# Senha do banco de dados
DB_PASSWORD=postgres

# Nome do banco de dados
DB_NAME=minha-api

# Habilita SSL na conexão com o banco (true | false)
DB_SSL=false
```

---

## Configuração do TypeOrmModule

O `TypeOrmModule.forRootAsync` deve ser configurado com as seguintes opções:

| Opção | Valor | Observação |
|-------|-------|------------|
| `type` | `'postgres'` | Driver fixo |
| `host` | `config.getOrThrow<string>('DB_HOST')` | Via `ConfigService` |
| `port` | `config.getOrThrow<number>('DB_PORT')` | Via `ConfigService` |
| `username` | `config.getOrThrow<string>('DB_USER')` | Via `ConfigService` |
| `password` | `config.getOrThrow<string>('DB_PASSWORD')` | Via `ConfigService` |
| `database` | `config.getOrThrow<string>('DB_NAME')` | Via `ConfigService` |
| `ssl` | `config.getOrThrow<string>('DB_SSL') === 'true' ? { rejectUnauthorized: false } : false` | Via `ConfigService` |
| `entities` | `[__dirname + '/../**/*.entity{.ts,.js}']` | Glob que encontra entidades de qualquer módulo de feature; `{.ts,.js}` suporta ts-node e código compilado |
| `migrations` | `[__dirname + '/migrations/*{.ts,.js}']` | Encontra `.ts` com ts-node (desenvolvimento) e `.js` no código compilado (produção) |
| `migrationsRun` | `config.getOrThrow<string>('NODE_ENV') === 'development'` | Auto-executa apenas em desenvolvimento |
| `synchronize` | `false` | Sempre desabilitado |

---

## DataSource para o TypeORM CLI

O arquivo `src/database/data-source.ts` é usado exclusivamente pelo TypeORM CLI e não é importado pela aplicação NestJS. Lê variáveis de ambiente diretamente via `process.env`.

| Opção | Valor |
|-------|-------|
| `type` | `'postgres'` |
| `host` | `process.env.DB_HOST` |
| `port` | `Number(process.env.DB_PORT)` |
| `username` | `process.env.DB_USER` |
| `password` | `process.env.DB_PASSWORD` |
| `database` | `process.env.DB_NAME` |
| `ssl` | `process.env.DB_SSL === 'true' ? { rejectUnauthorized: false } : false` |
| `entities` | `['src/**/*.entity.ts']` |
| `migrations` | `['src/database/migrations/*.ts']` |
| `synchronize` | `false` |

---

## Scripts do package.json

Os scripts abaixo devem ser adicionados ao `package.json`:

| Script | Comando | Finalidade |
|--------|---------|------------|
| `migration:generate` | `typeorm-ts-node-commonjs migration:generate -d src/database/data-source.ts --` | Gera migration a partir da diferença entre entidades e banco |
| `migration:create` | `typeorm-ts-node-commonjs migration:create --` | Cria arquivo de migration vazio para escrita manual |
| `migration:run` | `typeorm-ts-node-commonjs migration:run -d src/database/data-source.ts` | Executa migrations pendentes |
| `migration:revert` | `typeorm-ts-node-commonjs migration:revert -d src/database/data-source.ts` | Reverte a última migration aplicada |

> `typeorm-ts-node-commonjs` é um binário incluído no pacote `typeorm` que executa o CLI com suporte a TypeScript via `ts-node`. Não requer instalação separada.

> O `--` ao final dos scripts `migration:generate` e `migration:create` separa as flags do pnpm dos argumentos posicionais do TypeORM CLI, garantindo que o caminho passado na chamada seja interpretado como posicional e não como flag desconhecida.

### Convenção de uso dos scripts

`migration:generate` e `migration:create` exigem o caminho do arquivo como argumento posicional, passado na chamada:

```bash
# Gera migration a partir das entidades (nome descreve a operação)
pnpm migration:generate src/database/migrations/CreateUsersTable

# Cria arquivo de migration vazio
pnpm migration:create src/database/migrations/SeedInitialRoles
```

O caminho deve sempre apontar para `src/database/migrations/<NomeDaMigration>` — sem extensão, pois o TypeORM adiciona o timestamp e o `.ts` automaticamente.

---

## Nomenclatura de migrations

Migrations geradas via CLI seguem o padrão TypeORM: `<timestamp>-<NomeDaMigration>.ts`.

O nome deve descrever a operação no formato `PascalCase`:

- `CreateUsersTable`
- `AddEmailIndexToUsers`
- `RenameColumnNameToFullName`

Exemplo de arquivo gerado: `src/database/migrations/1716234567890-CreateUsersTable.ts`

---

## Adições ao agents.md

Ao aplicar esta spec, acrescentar a seção abaixo ao `agents.md` do projeto:

```markdown
## Banco de dados (setup/nestjs/nestjs-database)

- `DatabaseModule` é global e disponibiliza a conexão TypeORM para todos os módulos — não registrar `TypeOrmModule` em módulos de feature
- Variáveis `DB_*` devem constar no schema Joi em `src/config/env.validation.ts` — atualizar o schema ao introduzir novas variáveis de banco
- `synchronize` é sempre `false` — nunca habilitar, nem em desenvolvimento
- Migrations rodam automaticamente apenas em `NODE_ENV === 'development'`; em produção, executar `pnpm migration:run` manualmente antes de subir a aplicação
- Nomenclatura de migrations: PascalCase descrevendo a operação (`CreateUsersTable`, `AddEmailIndexToUsers`)
- Caminho das migrations: `src/database/migrations/`
- Nunca fazer alterações de schema diretamente no banco — toda mudança passa por migration
```
