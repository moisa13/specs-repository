# Changelog — NestJS — Configuração de Banco de Dados

> Versões seguem Semantic Versioning: MAJOR.MINOR.PATCH
>
> - **MAJOR** — mudança que quebra implementações existentes
> - **MINOR** — nova regra ou edge case (implementações existentes precisam de revisão)
> - **PATCH** — correção de texto, sem mudança de comportamento

---

## [0.1.0] — 2026-05

### Adicionado
- Versão inicial da spec: `DatabaseModule` com `TypeOrmModule.forRootAsync`, variáveis de ambiente, `data-source.ts` para o CLI, scripts de migration com `--` para repasse correto de argumentos posicionais via pnpm, comportamento de auto-execução por ambiente, ADRs de decisão arquitetural (TypeORM, `synchronize: false`, migrations automáticas em desenvolvimento, glob de entidades, configuração SSL) e critérios de aceitação
