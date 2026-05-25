# Decisions — NestJS — Configuração de Banco de Dados

> Decisões que podem parecer questionáveis sem contexto — evita que implementadores "melhorem" comportamentos deliberadamente escolhidos.

---

## ADR-001 — TypeORM como ORM

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
O ecossistema NestJS oferece suporte oficial a TypeORM, Prisma, Drizzle e Sequelize. A escolha impacta a forma como migrations são escritas, mantidas e aplicadas.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Prisma | Schema-first; tipagem gerada automaticamente; tooling moderno | Migrations geradas automaticamente a partir do schema — sem controle sobre o SQL; `down()` não é suportado nativamente |
| Drizzle | SQL-first; leve; muito controle | Ecosistema menos maduro; menos integrado ao NestJS |
| TypeORM (escolhido) | Integração oficial com NestJS; migrations com `up()` e `down()` explícitos; controle total sobre o SQL; suporte a `migration:revert` | Mais verboso; decorators em entidades |

### Decisão
TypeORM com `@nestjs/typeorm` e driver `pg`.

### Consequências
- Migrations têm `up()` e `down()` explícitos — reversão é suportada via `migration:revert`
- O SQL das migrations é controlado — não há geração automática opaca
- Entidades usam decorators TypeScript

---

## ADR-002 — `synchronize: false` em todos os ambientes

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
O TypeORM oferece `synchronize: true`, que aplica automaticamente as diferenças entre entidades e o schema do banco a cada inicialização. É conveniente em desenvolvimento mas destrói dados em produção.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| `synchronize: true` em desenvolvimento | Sem necessidade de gerar migrations durante o desenvolvimento | Cria hábito errado; diferença de comportamento entre ambientes; risco em produção se variável de ambiente estiver configurada incorretamente |
| `synchronize: false` sempre (escolhida) | Comportamento idêntico em todos os ambientes; migrations são o único mecanismo de mudança de schema | Exige gerar migration para qualquer mudança de schema |

### Decisão
`synchronize: false` em todos os ambientes, sem exceção.

### Consequências
- Qualquer mudança de schema exige uma migration gerada via CLI
- O comportamento é previsível e idêntico em desenvolvimento, staging e produção

---

## ADR-003 — Migrations automáticas apenas em desenvolvimento

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Em produção, aplicar migrations automaticamente no bootstrap pode causar falha irreversível se a migration for incorreta e não houver janela de manutenção. Em desenvolvimento, a execução manual cria atrito desnecessário.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Manual em todos os ambientes | Controle total; comportamento idêntico | Atrito em desenvolvimento; risco de esquecer de rodar as migrations antes de testar |
| Automático em todos os ambientes | Sem atrito | Perigoso em produção — qualquer migration com erro derruba a aplicação sem controle |
| Automático em desenvolvimento, manual em produção (escolhida) | Desenvolvimento fluido; produção controlada | Diferença de comportamento entre ambientes (aceita e documentada) |

### Decisão
`migrationsRun: config.get<string>('NODE_ENV') === 'development'`.

### Consequências
- Em desenvolvimento: migrations são aplicadas automaticamente ao subir a aplicação
- Em produção: `pnpm migration:run` deve ser executado manualmente antes do deploy
- Em test: migrations não são aplicadas automaticamente — gerenciamento de estado definido por `setup/testing` (a criar)

---

## ADR-004 — Glob de entidades em vez de lista explícita

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
O `TypeOrmModule` precisa conhecer todas as entidades da aplicação. Há duas formas de registrá-las: lista explícita no `DatabaseModule` ou glob que encontra arquivos `.entity.ts` automaticamente.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Lista explícita no `DatabaseModule` | Explícito; sem "magia" | Cada nova entidade exige alterar o `DatabaseModule` — cria acoplamento entre infraestrutura e domínio |
| Glob automático (escolhida) | Entidades de qualquer módulo de feature são detectadas sem alterar o `DatabaseModule` | Requer atenção à convenção de nomenclatura (`*.entity.ts`) |

### Decisão
Glob `[__dirname + '/../**/*.entity{.ts,.js}']` no `DatabaseModule` e `['src/**/*.entity.ts']` no `data-source.ts`. O `{.ts,.js}` garante compatibilidade tanto com ts-node (desenvolvimento) quanto com código compilado (produção).

### Consequências
- Toda entidade deve seguir a convenção de nomenclatura `*.entity.ts`
- O `DatabaseModule` nunca precisa ser alterado ao adicionar novas entidades

---

## ADR-005 — Configuração SSL com `rejectUnauthorized: false` como padrão

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Quando `DB_SSL=true`, a conexão com o banco pode ser feita com ou sem verificação do certificado do servidor. Ambientes com certificados auto-assinados (ex: RDS sem ACM, bancos internos) falham com `rejectUnauthorized: true`.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| `rejectUnauthorized: true` (verificação estrita) | Seguro contra ataques MITM | Requer certificado válido e configuração adicional de CA |
| `rejectUnauthorized: false` (escolhida) | Funciona com certificados auto-assinados sem configuração extra | Não verifica a identidade do servidor |

### Decisão
`rejectUnauthorized: false` como padrão ao habilitar SSL. Ambientes que exigem verificação estrita devem estender a configuração SSL (ex: incluir `ca`, `cert`, `key`) e documentar a mudança neste arquivo.

### Consequências
- SSL habilitado protege o tráfego em trânsito, mas não valida o certificado do servidor
- Para verificação estrita, a propriedade `ssl` deve ser substituída por um objeto com `rejectUnauthorized: true` e o certificado da CA — essa mudança deve ser registrada aqui com o contexto do ambiente
