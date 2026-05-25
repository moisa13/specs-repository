# Decisions — NestJS — Inicialização de Projeto

> Decisões que podem parecer questionáveis sem contexto — evita que implementadores "melhorem" comportamentos deliberadamente escolhidos.

---

## ADR-001 — pnpm como gerenciador de pacotes

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
O ecossistema Node oferece npm, yarn e pnpm. A escolha impacta velocidade de instalação, uso de disco e reprodutibilidade do lockfile entre ambientes.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| npm | Nativo do Node, sem instalação extra | Mais lento, `node_modules` maior |
| yarn | Workspaces maduros | Overhead de configuração, dois formatos de lockfile |
| pnpm (escolhida) | Instalações mais rápidas, hard links economizam disco, lockfile estrito | Requer instalação global prévia |

### Decisão
pnpm como gerenciador de pacotes padrão. Criação do projeto via `nest new --package-manager pnpm`.

### Consequências
- Lockfile `pnpm-lock.yaml` deve ser commitado
- Ambientes de CI e desenvolvedores devem ter pnpm instalado (`npm install -g pnpm`)

---

## ADR-002 — ConfigModule com validação via Joi

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Sem validação de variáveis de ambiente na inicialização, erros de configuração só aparecem em runtime — muitas vezes em produção, em fluxos específicos.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| `class-validator` + DTO de env | Mesma lib dos DTOs da API; tipagem forte | Mais boilerplate; requer instância da classe para validar |
| Joi com `ConfigModule` (escolhida) | Integração nativa com `@nestjs/config`; schema declarativo e conciso | Adiciona dependência `joi` |

### Decisão
`ConfigModule.forRoot({ isGlobal: true, validationSchema: Joi.object({...}) })` registrado no `AppModule`. O schema Joi fica em `src/config/env.validation.ts`.

### Consequências
- Aplicação falha imediatamente na inicialização se qualquer variável obrigatória estiver ausente ou inválida
- `ConfigService` disponível em todos os módulos sem import adicional

---

## ADR-003 — ValidationPipe com whitelist, forbidNonWhitelisted e transform

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Sem `whitelist`, campos extras no body chegam silenciosamente ao service — abre espaço para mass-assignment. Sem `transform`, valores string de query params não são convertidos para os tipos esperados (número, boolean).

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| `whitelist: false` | Menos erros em requests mal-formadas durante desenvolvimento | Risco de mass-assignment; fields extras chegam ao service |
| `whitelist: true, forbidNonWhitelisted: true, transform: true` (escolhida) | Explícito; rejeita campos não declarados; converte tipos automaticamente | DTOs devem ser completos e representar exatamente o contrato |

### Decisão
`ValidationPipe` global com `whitelist: true`, `forbidNonWhitelisted: true` e `transform: true`.

### Consequências
- Qualquer campo não declarado no DTO gera 400
- DTOs devem ser completos — nenhum campo implícito
- Conversão de tipos é automática (ex: `"3"` vira `3` quando o DTO espera `number`)

---

## ADR-004 — Filtro de exceção global normaliza todas as respostas de erro

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Sem um filtro global, diferentes partes da aplicação retornam formatos de erro diferentes: formato padrão do NestJS, erros de ORMs, exceções não tratadas. Isso quebra o contrato com clientes da API.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Tratar erros em cada controller | Controle granular por endpoint | Repetitivo; inconsistência garantida ao escalar o time |
| Filtro de exceção global (escolhida) | Formato único para todos os erros da aplicação | Novos tipos de exceção precisam ser adicionados ao filtro |

### Decisão
`AllExceptionsFilter` registrado globalmente no bootstrap. Normaliza qualquer exceção para o formato definido em `contracts.md`. Stack traces nunca são expostos na resposta.

### Consequências
- Todos os erros da aplicação seguem o mesmo contrato de resposta
- Ao introduzir novos tipos de exceção de domínio, o filtro deve ser atualizado para mapeá-los corretamente

---

## ADR-005 — Swagger habilitado apenas fora de produção

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Expor a documentação Swagger em produção pode revelar a estrutura interna da API para agentes maliciosos, mesmo que os endpoints exijam autenticação.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Swagger sempre habilitado | Facilita debugging em produção | Exposição desnecessária da estrutura da API |
| Swagger protegido por rota com autenticação | Mantém acesso em produção com controle | Complexidade adicional; não necessário para este boilerplate |
| Swagger apenas em não-produção (escolhida) | Sem exposição em produção; simples de implementar | Indisponível em produção (aceito) |

### Decisão
Swagger montado no bootstrap apenas quando `process.env.NODE_ENV !== 'production'`.

### Consequências
- Não é possível acessar `/docs` em produção — retorna 404
- Para inspeção da API em produção, usar a collection exportada do Swagger em desenvolvimento

---

## ADR-006 — CORS_ORIGIN como string separada por vírgula

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
APIs REST frequentemente precisam aceitar chamadas de mais de uma origem (ex: frontend e painel admin em domínios diferentes). A forma de expressar múltiplas origens via variável de ambiente não é óbvia e há pelo menos três abordagens comuns.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Múltiplas variáveis (`CORS_ORIGIN_1`, `CORS_ORIGIN_2`, ...) | Cada origem é explícita e validável individualmente | Número de origens não é fixo; escala mal; schema Joi não consegue validar quantidade arbitrária de forma limpa |
| JSON array (`CORS_ORIGIN=["https://a.com","https://b.com"]`) | Estruturado e tipado | Propenso a erros de escape em arquivos `.env`; dificulta edição manual |
| String separada por vírgula (escolhida) | Convencional em variáveis de ambiente (padrão Docker, Kubernetes, 12-factor); fácil de editar; um único `split(',').map(s => s.trim())` no bootstrap resolve | Requer parse manual; vírgulas dentro de URLs não são possíveis (não é um caso real) |

### Decisão
`CORS_ORIGIN` recebe uma string com uma ou mais origens separadas por vírgula. O bootstrap faz o parse via `split(',').map(s => s.trim())` antes de passar o array ao `enableCors`.

### Consequências
- Uma única origem continua funcionando sem alteração — `split` em string sem vírgula retorna array de um elemento
- Espaços ao redor das vírgulas são tolerados pelo `trim`
- O schema Joi valida apenas que a variável é uma string não vazia — a validade de cada URL individual não é verificada na inicialização

---

## ADR-007 — ESLint com typescript-eslint (recommendedTypeChecked) e Prettier

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Projetos sem linting consistente acumulam divergências de estilo e permitem erros silenciosos como promises não tratadas, imports via `require()` e funções `async` sem `await`. O NestJS CLI já gera uma configuração de ESLint, mas a escolha do ruleset impacta diretamente como o código deve ser escrito.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| `tseslint.configs.recommended` | Menos restritivo; menos atrito no início | Não usa informação de tipos — perde regras importantes como `no-floating-promises` |
| `tseslint.configs.recommendedTypeChecked` (escolhida) | Usa o type-checker do TypeScript para regras mais precisas; detecta promises não tratadas, `require()` e `async` sem `await` | Requer `parserOptions.projectService`; linting mais lento em projetos grandes |

### Decisão
`tseslint.configs.recommendedTypeChecked` com Prettier integrado via `eslint-plugin-prettier`. Configuração gerada pelo NestJS CLI mantida sem alterações, com três ajustes de nível:
- `no-explicit-any`: desabilitado (a restrição é por convenção, não por regra)
- `no-floating-promises`: `warn` (não bloqueia, mas sinaliza)
- `no-unsafe-argument`: `warn`

### Consequências
- `require()` é proibido — usar sempre `import`
- Funções `async` devem ter ao menos um `await` — remover `async` quando não necessário
- Promises sem tratamento geram aviso — usar `void`, `await` ou `.catch`
- Prettier define aspas simples e trailing comma em todos os contextos
