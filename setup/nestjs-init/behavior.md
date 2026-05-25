# Behavior — NestJS — Inicialização de Projeto

---

## Objetivo

Garantir que todo projeto NestJS para APIs parta de uma estrutura consistente, com módulos globais configurados corretamente antes de qualquer feature ser desenvolvida.

---

## Fluxo principal

1. O projeto é criado com `nest new <nome> --package-manager pnpm --skip-git --strict`; os arquivos `src/app.controller.ts`, `src/app.service.ts` e `src/app.controller.spec.ts` gerados pelo CLI são deletados imediatamente — eles não fazem parte da estrutura da spec
2. As variáveis de ambiente são validadas por schema na inicialização, conforme `contracts.md`; a aplicação não sobe se alguma obrigatória estiver ausente ou inválida
3. A validação de dados de entrada é aplicada globalmente a todos os endpoints: campos não declarados nos contratos são rejeitados e tipos são convertidos automaticamente
4. Todas as respostas de erro são interceptadas e normalizadas para o formato definido em `contracts.md`, sem expor internals do sistema
5. CORS é configurado com as origens lidas de `CORS_ORIGIN`, separadas por vírgula e convertidas em array antes de serem passadas ao `enableCors`
6. A rota `GET /` retorna `APP_NAME`, `APP_DESCRIPTION`, versão do `package.json`, timestamp e uptime em tempo real
7. A documentação da API está disponível em `/docs` apenas quando `NODE_ENV` não for `production`
8. A aplicação sobe sem erros com todas as variáveis obrigatórias presentes

---

## Regras de negócio

- O módulo raiz registra dois tipos de módulos: (1) **infraestrutura** — configuração, banco de dados, logging, cache e outros serviços transversais; (2) **domínio** — módulos de feature que encapsulam seus próprios controllers, services e providers. O módulo raiz nunca deve definir controllers, services ou providers diretamente nele
- O ponto de entrada da aplicação não deve conter lógica inline além do bootstrap — configurações como Swagger e CORS devem ser encapsuladas em funções ou módulos dedicados
- Variáveis de ambiente obrigatórias devem ser validadas por schema na inicialização; a aplicação deve encerrar com exit code 1 se alguma estiver ausente ou com formato inválido
- A validação de dados de entrada deve rejeitar campos não declarados nos DTOs e converter tipos automaticamente
- O filtro de exceção global deve normalizar todas as respostas de erro para o formato definido em `contracts.md` — sem exceções
- O filtro de exceção global deve ser registrado **antes** do `ValidationPipe` no bootstrap — a ordem garante que erros lançados pelo pipe sejam capturados e normalizados pelo filtro
- A documentação da API deve ser montada apenas quando `NODE_ENV` não for `production`
- O arquivo com variáveis de ambiente reais nunca deve ser commitado; apenas o arquivo de exemplo com chaves e descrições (sem valores reais)
- Não deve haver providers globais fora do módulo raiz sem justificativa registrada em `decisions.md`

---

## Edge cases

| Situação | Comportamento esperado |
|----------|----------------------|
| Variável de ambiente obrigatória ausente | Aplicação encerra com exit code 1 e mensagem descritiva indicando qual variável está faltando |
| Variável com formato inválido (ex: `PORT=abc`) | Aplicação encerra com exit code 1 e mensagem do erro de validação Joi |
| Request com campos extras não declarados no DTO | `ValidationPipe` rejeita com 400 e lista os campos não permitidos em `details` |
| Request com `Content-Type` não suportado | NestJS retorna 415 — o body não é processado |
| Swagger acessado com `NODE_ENV=production` | Retorna 404 — o Swagger não está montado |
| Porta já em uso ao subir a aplicação | Aplicação encerra com exit code 1 e mensagem de erro da porta |
| `.env` ausente em desenvolvimento | Joi acusa variáveis obrigatórias ausentes e encerra a aplicação |
| Qualquer origem em `CORS_ORIGIN` for `*` com `NODE_ENV=production` | A aplicação sobe, mas o comportamento é inseguro — um aviso deve ser emitido no log de inicialização indicando que CORS aberto não é recomendado em produção |

---

## Restrições

- Não usar `any` como tipo de retorno em controllers, services ou filtros
- Não expor stack traces em respostas de erro — apenas a mensagem normalizada
- Não registrar o `ValidationPipe` ou o filtro de exceção em módulos individuais; apenas globalmente no bootstrap

---

## O que esta spec NÃO cobre

- Configuração de ORM/banco de dados → ver `setup/database` (a criar)
- Estratégias de autenticação → ver `auth/` (a criar)
- Logging estruturado → ver `observability/logging` (a criar)
- Estrutura e configuração de testes → ver `setup/testing` (a criar)
- Containerização com Docker → ver `setup/docker` (a criar)
