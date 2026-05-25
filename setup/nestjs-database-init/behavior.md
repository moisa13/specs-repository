# Behavior — NestJS — Configuração de Banco de Dados

---

## Objetivo

Garantir que todo projeto NestJS tenha uma configuração de banco de dados consistente, com TypeORM conectado ao PostgreSQL via `ConfigService`, migrations organizadas e comportamento de execução diferenciado por ambiente.

---

## Fluxo principal

1. Os pacotes `@nestjs/typeorm`, `typeorm` e `pg` são instalados via pnpm
2. As variáveis de ambiente de banco são adicionadas ao schema Joi em `src/config/env.validation.ts` e ao `.env.example`
3. O `DatabaseModule` é criado em `src/database/database.module.ts` com `TypeOrmModule.forRootAsync` lendo as configurações do `ConfigService`
4. O `DatabaseModule` é registrado no `AppModule` como módulo de infraestrutura, após o `ConfigModule`
5. O arquivo `src/database/data-source.ts` é criado para uso exclusivo do TypeORM CLI
6. O diretório `src/database/migrations/` é criado
7. Os scripts de migration são adicionados ao `package.json`
8. A aplicação sobe e, em desenvolvimento, executa automaticamente as migrations pendentes antes de aceitar requisições

---

## Regras de negócio

- `synchronize` deve ser sempre `false` em todos os ambientes — sem exceção
- `migrationsRun` deve ser `true` apenas quando `NODE_ENV=development`; em produção e em test deve ser `false`
- A conexão com o banco utiliza o `ConfigService` — nenhuma credencial pode ser hardcoded no código
- O `DatabaseModule` deve ser registrado no `AppModule` como módulo de infraestrutura, após o `ConfigModule`
- O arquivo `data-source.ts` lê variáveis de ambiente diretamente via `process.env` — é usado apenas pelo TypeORM CLI, fora do contexto NestJS
- O glob de entidades no `DatabaseModule` é `*{.ts,.js}` relativo ao `__dirname` — encontra `.ts` com ts-node (desenvolvimento) e `.js` no código compilado (produção); o `data-source.ts` usa o glob estático `src/**/*.entity.ts`, pois o CLI opera sempre no código-fonte TypeScript
- O diretório de migrations no `TypeOrmModule` usa o glob `*{.ts,.js}` relativo ao `__dirname` — em desenvolvimento (ts-node) resolve para `src/database/migrations/*.ts`; em produção (código compilado) resolve para `dist/database/migrations/*.js`; o `data-source.ts` aponta diretamente para os fontes TypeScript (`src/database/migrations/*.ts`)
- Migrations devem ser geradas via CLI (`pnpm migration:generate`) — escrita manual é permitida apenas para operações que o CLI não consegue inferir (ex: transformação de dados), e deve ser registrada em `decisions.md`
- A tabela de controle de migrations gerenciada pelo TypeORM (`migrations`) nunca deve ser alterada ou deletada manualmente

---

## Edge cases

| Situação | Comportamento esperado |
|----------|----------------------|
| Banco indisponível no bootstrap | A aplicação falha na inicialização com erro de conexão explícito e encerra com exit code 1 |
| Migration com erro em desenvolvimento | O TypeORM interrompe a execução, loga o erro e a aplicação encerra — as migrations não são revertidas automaticamente |
| Variável de banco ausente | O schema Joi impede a inicialização antes mesmo de tentar conectar, com mensagem indicando qual variável está faltando |
| `DB_SSL=true` sem certificado verificável | A conexão usa `rejectUnauthorized: false` — adequado para ambientes com certificados auto-assinados; para verificação estrita, a configuração SSL deve ser estendida e documentada em `decisions.md` |
| Migrations já aplicadas são reencontradas | O TypeORM consulta a tabela `migrations` e ignora as já aplicadas — operação idempotente |
| Nenhuma migration pendente | O TypeORM loga que não há migrations a executar e a aplicação continua normalmente |
| `pnpm migration:generate` sem alteração de entidade | O TypeORM não gera arquivo — comportamento esperado |

---

## Restrições

- Não usar `synchronize: true` em nenhum ambiente
- Não hardcodar credenciais de banco — sempre via `ConfigService` ou `process.env` no `data-source.ts`
- Não registrar o `DatabaseModule` em módulos de feature — apenas no `AppModule`
- Não definir entidades no `DatabaseModule` — elas são registradas por cada módulo de feature via `TypeOrmModule.forFeature`

---

## O que esta spec NÃO cobre

- Entidades, repositórios e padrões de acesso a dados → responsabilidade de cada spec de feature
- Health check de banco de dados → ver `observability/nestjs-health`
- Seeding de dados → não coberto por esta spec
- Configuração de testes com banco → ver `setup/testing` (a criar)
- Containerização do banco → ver `setup/docker` (a criar)
