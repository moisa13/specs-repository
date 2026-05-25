# Acceptance Criteria — NestJS — Configuração de Banco de Dados

> Formato: **Dado** [contexto] **Quando** [ação] **Então** [resultado esperado]

---

## Fluxo principal

**AC-01 — Aplicação conecta ao banco com variáveis válidas**
- **Dado** que todas as variáveis de banco estão definidas corretamente no `.env` e o PostgreSQL está acessível
- **Quando** `pnpm start:dev` é executado
- **Então** a aplicação sobe sem erros e o log confirma a conexão com o banco

**AC-02 — Migrations executadas automaticamente em desenvolvimento**
- **Dado** que `NODE_ENV=development` e há migrations pendentes
- **Quando** `pnpm start:dev` é executado
- **Então** as migrations são executadas antes da aplicação aceitar requisições e o log confirma cada migration aplicada

**AC-03 — Migrations não executadas automaticamente em produção**
- **Dado** que `NODE_ENV=production` e há migrations pendentes
- **Quando** a aplicação sobe
- **Então** as migrations não são executadas automaticamente

**AC-04 — CLI executa migrations em produção**
- **Dado** que `NODE_ENV=production` e há migrations pendentes
- **Quando** `pnpm migration:run` é executado
- **Então** as migrations pendentes são aplicadas e o log confirma cada migration executada

---

## Cenários de falha

**AC-05 — Banco indisponível impede a inicialização**
- **Dado** que o PostgreSQL não está acessível (host errado, porta errada ou serviço parado)
- **Quando** `pnpm start:dev` é executado
- **Então** a aplicação encerra com exit code 1 e exibe erro de conexão com o banco

**AC-06 — Variável de banco ausente impede a inicialização**
- **Dado** que `DB_HOST` não está definido no `.env`
- **Quando** `pnpm start:dev` é executado
- **Então** a aplicação encerra com exit code 1 antes de tentar conectar, com mensagem Joi indicando qual variável está ausente

**AC-07 — Migration com erro interrompe a inicialização em desenvolvimento**
- **Dado** que `NODE_ENV=development` e uma migration contém SQL inválido
- **Quando** `pnpm start:dev` é executado
- **Então** a aplicação encerra com exit code 1 e o log exibe o erro da migration — as demais migrations pendentes não são executadas

---

## Edge cases

**AC-08 — Nenhuma migration pendente não causa erro**
- **Dado** que todas as migrations já foram aplicadas
- **Quando** a aplicação sobe em desenvolvimento
- **Então** o TypeORM loga que não há migrations pendentes e a aplicação continua normalmente

**AC-09 — `synchronize` nunca altera o schema automaticamente**
- **Dado** que uma entidade é adicionada ou modificada sem migration correspondente
- **Quando** a aplicação sobe em qualquer ambiente
- **Então** o schema do banco não é alterado

---

## Qualidade de código

**AC-10 — Lint passa sem erros após a configuração**
- **Dado** que o `DatabaseModule` e o `data-source.ts` foram criados
- **Quando** `pnpm lint` é executado
- **Então** nenhum erro de ESLint é retornado

---

## O que caracteriza uma implementação incorreta

- `synchronize: true` em qualquer ambiente
- `migrationsRun: true` em produção
- Credenciais de banco hardcoded no código
- `DatabaseModule` registrado em módulo de feature em vez do `AppModule`
- `data-source.ts` importado pela aplicação NestJS (deve ser usado apenas pelo CLI)
- Scripts de migration ausentes no `package.json`

---

## Cobertura mínima de testes

> **Framework padrão:** Jest — instalado pelo NestJS CLI. Configuração completa de testes em `setup/testing` (a criar).

- [ ] Aplicação conecta ao banco com variáveis válidas e sobe sem erros (e2e)
- [ ] Aplicação encerra com variável de banco ausente antes de tentar conectar (teste unitário no schema Joi)
- [ ] Migrations são executadas automaticamente em `NODE_ENV=development` (e2e com banco real)
- [ ] Migrations não são executadas automaticamente em `NODE_ENV=production` (e2e)
- [ ] `pnpm migration:run` executa migrations pendentes com sucesso (e2e com banco real)
- [ ] Schema do banco não é alterado automaticamente quando `synchronize: false` (e2e com banco real)
