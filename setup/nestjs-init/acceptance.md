# Acceptance Criteria — NestJS — Inicialização de Projeto

> Formato: **Dado** [contexto] **Quando** [ação] **Então** [resultado esperado]

---

## Fluxo principal

**AC-01 — Aplicação sobe com variáveis válidas**
- **Dado** que todas as variáveis obrigatórias (`NODE_ENV`, `PORT`, `CORS_ORIGIN`, `APP_NAME`) estão definidas corretamente no `.env`
- **Quando** `pnpm start:dev` é executado
- **Então** a aplicação sobe sem erros, escuta na porta configurada e o log confirma a inicialização

**AC-02 — Rota raiz retorna informações da aplicação**
- **Dado** que a aplicação está rodando
- **Quando** `GET /` é acessado
- **Então** a resposta é `200` com `name` lido de `APP_NAME`, `version` lido de `package.json`, `description` lido de `APP_DESCRIPTION`, `timestamp` no formato ISO 8601 e `uptime` como número em segundos

**AC-03 — Swagger disponível em desenvolvimento**
- **Dado** que `NODE_ENV=development`
- **Quando** `GET /docs` é acessado no browser
- **Então** a interface do Swagger é retornada com os endpoints documentados

---

## Cenários de falha

**AC-04 — Variável obrigatória ausente impede a inicialização**
- **Dado** que `PORT` não está definido no `.env`
- **Quando** `pnpm start:dev` é executado
- **Então** a aplicação encerra com exit code 1 e exibe no stderr qual variável está ausente

**AC-05 — Variável com formato inválido impede a inicialização**
- **Dado** que `PORT=abc` está definido (valor não numérico)
- **Quando** `pnpm start:dev` é executado
- **Então** a aplicação encerra com exit code 1 e exibe a mensagem de erro do Joi descrevendo a falha de validação

**AC-06 — Request com campo extra é rejeitada**
- **Dado** que um endpoint espera um DTO com `name` e `email`
- **Quando** a request inclui um campo adicional `role` não declarado no DTO
- **Então** a resposta é `400` com `error: "BAD_REQUEST"` e `details` listando o campo não permitido, no formato de `contracts.md`

**AC-07 — Erro interno retorna formato padronizado**
- **Dado** que qualquer endpoint lança uma exceção não tratada
- **Quando** a request é processada
- **Então** a resposta é `500` com `error: "INTERNAL_SERVER_ERROR"` e sem stack trace exposto, no formato de `contracts.md`

---

## Edge cases

**AC-08 — Swagger não disponível em produção**
- **Dado** que `NODE_ENV=production`
- **Quando** `GET /docs` é acessado
- **Então** a resposta é `404` — o Swagger não está montado

**AC-09 — Stack trace não é exposto em erros**
- **Dado** que um erro interno ocorre
- **Quando** a resposta de erro é retornada
- **Então** o campo `details` está vazio e nenhum campo da resposta contém stack trace ou mensagens internas do sistema

---

## Qualidade de código

**AC-10 — Lint e formatação passam sem erros**
- **Dado** que a implementação está completa
- **Quando** `pnpm lint` é executado
- **Então** nenhum erro de ESLint é retornado e o código está formatado conforme a configuração Prettier definida em `contracts.md`

---

## O que caracteriza uma implementação incorreta

- Aplicação sobe mesmo com variáveis obrigatórias ausentes
- Resposta de erro não segue o formato de `contracts.md` (campos ausentes, nomes diferentes)
- `ValidationPipe` não rejeita campos extras não declarados no DTO
- Lógica de negócio presente no `AppModule` ou inline no `main.ts`
- Swagger acessível quando `NODE_ENV=production`
- Stack trace presente em qualquer resposta de erro
- `pnpm lint` retorna erros ou avisos de ESLint

---

## Cobertura mínima de testes

> **Framework padrão:** Jest — instalado pelo NestJS CLI. Configuração completa de testes em `setup/testing` (a criar).

- [ ] Aplicação sobe corretamente com variáveis válidas (e2e)
- [ ] Aplicação encerra com variável obrigatória ausente (teste unitário no schema Joi)
- [ ] Aplicação encerra com variável com formato inválido (teste unitário no schema Joi)
- [ ] `ValidationPipe` rejeita campos extras (e2e ou integration)
- [ ] Resposta de erro segue o formato padrão (e2e)
- [ ] `GET /` retorna os campos `name`, `version`, `description`, `timestamp` e `uptime` (e2e)
- [ ] Swagger retorna 404 em produção (e2e)
- [ ] `pnpm lint` passa sem erros (verificação manual ou CI)
