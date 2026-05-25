# Contracts — NestJS — Inicialização de Projeto

> Contratos de estrutura e configuração. O agente deve seguir estes contratos ao gerar o projeto.

---

## Estrutura de diretórios

```
<nome-do-projeto>/
├── src/
│   ├── app.module.ts
│   ├── main.ts
│   ├── modules/
│   │   └── app-info/
│   │       ├── app-info.module.ts
│   │       └── app-info.controller.ts
│   ├── common/
│   │   └── filters/
│   │       └── all-exceptions.filter.ts
│   └── config/
│       └── env.validation.ts
├── agents.md             ← contexto vivo do projeto para agentes
├── .env                  ← não commitado
├── .env.example
├── .gitignore
├── nest-cli.json
├── package.json
├── pnpm-lock.yaml
└── tsconfig.json
```

O arquivo `agents.md` é criado na raiz do projeto junto com a inicialização. Ele registra convenções, decisões e padrões do projeto em linguagem direta para agentes. Cada spec aplicada ao projeto acrescenta sua própria seção. Ver conteúdo inicial em `examples/gerar-agents-md.md`.

Todos os módulos da aplicação — sejam de infraestrutura (ex: `app-info`, `health`) ou de domínio (ex: `users`, `orders`) — residem em `src/modules/`. Os diretórios `src/common/`, `src/config/` e `src/database/` são transversais e ficam fora de `src/modules/`.

## Conteúdo mínimo do .gitignore

O NestJS CLI gera o `.gitignore` apenas quando `--skip-git` **não** é usado. Como o pré-requisito desta spec é que o git já esteja inicializado no projeto, o flag `--skip-git` é obrigatório e o `.gitignore` deve ser criado manualmente com ao menos as seguintes entradas:

```
/dist
/node_modules
.env
.env.local
.env.*.local
/coverage
```

---

## Configuração mínima do tsconfig.json

As opções abaixo devem estar presentes além das geradas pelo NestJS CLI:

| Opção | Valor | Motivo |
|-------|-------|--------|
| `resolveJsonModule` | `true` | Permite importar `package.json` para ler a versão da API no Swagger |
| `rootDir` | `"src"` | Torna explícito que toda saída compilada parte de `src/`, garantindo `dist/main.js` independente de efeito colateral das exclusões do `tsconfig.build.json` |

O `rootDir: "src"` no `tsconfig.json` exige que a propriedade `include` também seja definida para restringir o escopo ao diretório `src/` — sem ela, o TypeScript inclui `test/` pelo padrão `**/*` e emite erro de diagnóstico porque `test/` fica fora do `rootDir`:

```json
{
  "compilerOptions": {
    "rootDir": "src"
  },
  "include": ["src/**/*"]
}
```

A opção `baseUrl` gerada pelo NestJS CLI deve ser **removida** — está depreciada no TypeScript 6.0+ com `moduleResolution: nodenext` e não é necessária enquanto não houver path aliases (`paths`) configurados.

A opção `incremental` gerada pelo NestJS CLI deve ser **removida** — o `nest-cli.json` gerado pelo CLI tem `deleteOutDir: true`, que limpa apenas `dist/` a cada build. Os arquivos `.tsbuildinfo` gerados pelo `incremental` ficam na raiz do projeto, sobrevivem ao `deleteOutDir` e fazem o TypeScript concluir que nada mudou, pulando a emissão dos arquivos. O resultado é `nest build` e `npm run start:dev` saindo com código 0 sem gerar `dist/`.

## Configuração de linting e formatação

O projeto usa ESLint com `typescript-eslint` e Prettier integrado via `eslint-plugin-prettier`. A configuração é gerada pelo NestJS CLI e não deve ser removida.

O array `ignores` do `eslint.config.mjs` gerado pelo CLI não inclui `dist/**` por padrão. Após o primeiro `pnpm build`, o ESLint falha ao tentar analisar os arquivos compilados. A entrada `'dist/**'` deve ser adicionada ao `ignores`:

```js
{
  ignores: ['eslint.config.mjs', 'dist/**'],
},
```

O `include: ["src/**/*"]` adicionado ao `tsconfig.json` exclui `test/` do TypeScript project service. O ruleset `recommendedTypeChecked` depende do project service para type-checking — sem acesso aos arquivos de `test/`, o ESLint falha ao analisá-los. A propriedade `allowDefaultProject` deve ser adicionada ao bloco `languageOptions.parserOptions` do `eslint.config.mjs`:

```js
languageOptions: {
  parserOptions: {
    projectService: {
      allowDefaultProject: ['test/*.ts'],
    },
  },
},
```

### Prettier

| Opção | Valor |
|-------|-------|
| `singleQuote` | `true` |
| `trailingComma` | `"all"` |

### Regras ESLint relevantes para implementação

| Regra | Nível | Impacto |
|-------|-------|---------|
| `@typescript-eslint/no-require-imports` | error | Usar `import` em vez de `require()` |
| `@typescript-eslint/require-await` | error | Funções `async` devem ter ao menos um `await` |
| `@typescript-eslint/no-floating-promises` | warn | Promises devem ser aguardadas, encadeadas com `.catch` ou marcadas com `void` |
| `@typescript-eslint/no-explicit-any` | off | Desabilitado — a restrição de não usar `any` em retornos é por convenção (ver `behavior.md`) |
| `@typescript-eslint/no-unsafe-argument` | warn | Alerta ao passar valor de tipo `any` como argumento de função tipada |

---

## Campos obrigatórios do package.json

Os campos abaixo devem ser preenchidos na criação do projeto com os valores informados:

| Campo | Valor inicial |
|-------|---------------|
| `name` | Nome do projeto em kebab-case |
| `description` | Valor de `APP_DESCRIPTION` definido no `.env` |

---

## Rota raiz — GET /

A aplicação expõe uma rota `GET /` que retorna informações básicas sobre si mesma. Esta rota é implementada por um módulo de infraestrutura dedicado (`AppInfoModule`) registrado no `AppModule`.

### Formato de resposta

```json
{
  "name": "nome-do-projeto",
  "version": "1.0.0",
  "description": "API do nome-do-projeto",
  "timestamp": "2026-05-24T21:07:13.856Z",
  "uptime": 4531.328043197
}
```

| Campo | Tipo | Fonte |
|-------|------|-------|
| `name` | string | Variável de ambiente `APP_NAME` |
| `version` | string | Campo `version` do `package.json` |
| `description` | string | Variável de ambiente `APP_DESCRIPTION` (não o campo `description` do `package.json`) |
| `timestamp` | string | ISO 8601 UTC gerado no momento da requisição |
| `uptime` | number | `process.uptime()` em segundos |

### DTO de resposta

| Classe | Finalidade |
|--------|------------|
| `AppInfoResponseDto` | Representa a resposta do `GET /` — deve ser uma `class` com `@ApiProperty()` em cada campo para que o Swagger gere o schema corretamente |

### Decorators Swagger

| Endpoint | Decorators |
|----------|------------|
| Controller | `@ApiTags('app-info')` |
| `GET /` | `@ApiOperation({ summary: 'Retorna metadados da aplicação' })`, `@ApiOkResponse({ type: AppInfoResponseDto })` |

---

## Estrutura de diretórios — crescimento com features

Ao adicionar a primeira feature, a estrutura cresce dentro de `src/modules/`:

```
src/
└── modules/
    └── <módulo>/
        ├── <módulo>.module.ts
        ├── <módulo>.controller.ts
        ├── <módulo>.service.ts
        └── dto/
            └── <ação>-<módulo>.dto.ts
```

---

## Variáveis de ambiente

| Variável | Tipo | Obrigatória | Validação Joi | Descrição |
|----------|------|-------------|---------------|-----------|
| `NODE_ENV` | string | ✅ | `Joi.string().valid('development', 'production', 'test').required()` | Ambiente de execução |
| `PORT` | number | ✅ | `Joi.number().integer().min(1).max(65535).required()` | Porta em que a aplicação escuta |
| `CORS_ORIGIN` | string | ✅ | `Joi.string().required()` | Origens permitidas no CORS, separadas por vírgula (ex: `https://app.exemplo.com,https://admin.exemplo.com`); usar `*` apenas em desenvolvimento |
| `APP_NAME` | string | ✅ | `Joi.string().required()` | Nome da aplicação — fonte de verdade para identificação em e-mails, Swagger e demais integrações |
| `APP_DESCRIPTION` | string | ✅ | `Joi.string().required()` | Descrição curta da aplicação — exibida na rota `GET /` |

O `.env.example` deve conter todas as variáveis acima com valor de exemplo e comentário descritivo:

```dotenv
# Ambiente de execução: development | production | test
NODE_ENV=development

# Porta em que a aplicação vai escutar
PORT=3000

# Origens permitidas para CORS, separadas por vírgula (URL completa ou * para qualquer origem)
CORS_ORIGIN=http://localhost:5173,http://localhost:3001

# Nome da aplicação — usado em e-mails, Swagger e demais integrações
APP_NAME=minha-api

# Descrição curta da aplicação
APP_DESCRIPTION=API do minha-api
```

---

## Formato padrão de resposta de erro

Todas as respostas de erro devem seguir este formato, sem exceção:

```json
{
  "statusCode": 400,
  "error": "BAD_REQUEST",
  "message": "Mensagem legível descrevendo o problema",
  "details": []
}
```

| Campo | Tipo | Sempre presente | Descrição |
|-------|------|-----------------|-----------|
| `statusCode` | number | ✅ | Código HTTP da resposta |
| `error` | string | ✅ | Código semântico em SNAKE_CASE maiúsculo |
| `message` | string | ✅ | Mensagem legível para log ou exibição |
| `details` | array | ✅ | Lista de problemas específicos; pode ser vazia |

### Exemplo — erro de validação de DTO

```json
{
  "statusCode": 400,
  "error": "BAD_REQUEST",
  "message": "Validation failed",
  "details": [
    { "field": "email", "message": "email must be an email" },
    { "field": "role", "message": "property role should not exist" }
  ]
}
```

### Exemplo — erro interno não tratado

```json
{
  "statusCode": 500,
  "error": "INTERNAL_SERVER_ERROR",
  "message": "Internal server error",
  "details": []
}
```

---

## Mapeamento de códigos HTTP para `error`

| HTTP | `error` |
|------|---------|
| 400 | `BAD_REQUEST` |
| 401 | `UNAUTHORIZED` |
| 403 | `FORBIDDEN` |
| 404 | `NOT_FOUND` |
| 409 | `CONFLICT` |
| 415 | `UNSUPPORTED_MEDIA_TYPE` |
| 422 | `UNPROCESSABLE_ENTITY` |
| 429 | `TOO_MANY_REQUESTS` |
| 500 | `INTERNAL_SERVER_ERROR` |

---

## Configuração do Swagger

| Propriedade | Valor |
|-------------|-------|
| Rota | `/docs` |
| Título | Valor de `APP_NAME` |
| Descrição | Valor de `APP_DESCRIPTION` |
| Versão | Lida do campo `version` de `package.json` |
| Habilitado em | `NODE_ENV !== 'production'` |
