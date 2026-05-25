# Changelog — NestJS — Inicialização de Projeto

> Versões seguem Semantic Versioning: MAJOR.MINOR.PATCH
>
> - **MAJOR** — mudança que quebra implementações existentes
> - **MINOR** — nova regra ou edge case (implementações existentes precisam de revisão)
> - **PATCH** — correção de texto, sem mudança de comportamento

---

## [Não lançado]

### Corrigido
- `behavior.md`: documenta o comando exato `nest new` com as flags `--skip-git` e `--strict`, e instrui a deletar os arquivos padrão do CLI (`app.controller.ts`, `app.service.ts`, `app.controller.spec.ts`) imediatamente após a criação
- `contracts.md`: documenta que o `eslint.config.mjs` gerado pelo CLI não inclui `dist/**` no `ignores` e que isso causa falha no lint após o primeiro build
- `contracts.md`: documenta o conteúdo mínimo do `.gitignore` e explica que o arquivo não é gerado automaticamente quando `--skip-git` é usado

---

## [0.1.0] — 2026-05

### Adicionado
- Versão inicial da spec: estrutura de diretórios, instalação com pnpm, ConfigModule com Joi, ValidationPipe global, filtro de exceção global, Swagger em desenvolvimento, contratos de resposta de erro e critérios de aceitação
