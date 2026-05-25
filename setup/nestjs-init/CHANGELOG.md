# Changelog — NestJS — Inicialização de Projeto

> Versões seguem Semantic Versioning: MAJOR.MINOR.PATCH
>
> - **MAJOR** — mudança que quebra implementações existentes
> - **MINOR** — nova regra ou edge case (implementações existentes precisam de revisão)
> - **PATCH** — correção de texto, sem mudança de comportamento

---

## [0.3.0] — 2026-05

### Adicionado
- `behavior.md`: adiciona passo 8 ao fluxo principal — criação do `agents.md` na raiz do projeto ao aplicar a spec; documenta que esta spec é a única que cria o arquivo, as demais apenas acrescentam seções
- `contracts.md`: documenta `agents.md` na estrutura de diretórios obrigatória com descrição do propósito e referência a `examples/gerar-agents-md.md`; documenta convenção `src/modules/` para módulos de feature e infraestrutura com lista de diretórios transversais (`src/common/`, `src/config/`, `src/database/`)
- `examples/gerar-agents-md.md`: novo exemplo com o conteúdo inicial do `agents.md` — seções de estrutura do projeto, convenções de código, variáveis de ambiente, formato de resposta de erro e Swagger

### Corrigido
- `examples/configurar-app-module.md`: caminho de import do `AppInfoModule` atualizado de `./app-info/` para `./modules/app-info/`
- `examples/implementar-app-info-module.md`: caminhos atualizados para `src/modules/app-info/`; import de `package.json` corrigido de `../../` para `../../../` (um nível a mais por causa do novo caminho)

---

## [0.2.0] — 2026-05

### Adicionado
- `contracts.md`: seção de DTO de resposta (`AppInfoResponseDto`) e decorators Swagger para o endpoint `GET /` — implementações existentes devem converter `interface AppInfoResponse` para `class AppInfoResponseDto` com `@ApiProperty()` em cada campo
- `examples/implementar-app-info-module.md`: exemplo atualizado com `AppInfoResponseDto` como `class` com `@ApiProperty()` e `@ApiOkResponse({ type: AppInfoResponseDto })` no controller

### Corrigido
- `behavior.md`: documenta o comando exato `nest new` com as flags `--skip-git` e `--strict`, e instrui a deletar os arquivos padrão do CLI (`app.controller.ts`, `app.service.ts`, `app.controller.spec.ts`) imediatamente após a criação
- `contracts.md`: documenta que o `eslint.config.mjs` gerado pelo CLI não inclui `dist/**` no `ignores` e que isso causa falha no lint após o primeiro build
- `contracts.md`: documenta o conteúdo mínimo do `.gitignore` e explica que o arquivo não é gerado automaticamente quando `--skip-git` é usado
- `acceptance.md`: adiciona `APP_DESCRIPTION` à lista de variáveis obrigatórias no AC-01
- `README.md`, `behavior.md`, `context.md`, `integration.md`: substitui referência `setup/database (a criar)` por `setup/nestjs-database`

---

## [0.1.0] — 2026-05

### Adicionado
- Versão inicial da spec: estrutura de diretórios, instalação com pnpm, ConfigModule com Joi, ValidationPipe global, filtro de exceção global, Swagger em desenvolvimento, contratos de resposta de erro e critérios de aceitação
