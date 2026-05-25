# Changelog — NestJS — Configuração de Banco de Dados

> Versões seguem Semantic Versioning: MAJOR.MINOR.PATCH
>
> - **MAJOR** — mudança que quebra implementações existentes
> - **MINOR** — nova regra ou edge case (implementações existentes precisam de revisão)
> - **PATCH** — correção de texto, sem mudança de comportamento

---

## [0.2.0] — 2026-05

### Adicionado
- `contracts.md`: seção "Adições ao agents.md" com 7 convenções — `DatabaseModule` global (não registrar `TypeOrmModule` em módulos de feature), variáveis `DB_*` no schema Joi, `synchronize: false` sem exceções, migrations automáticas apenas em `development`, `pnpm migration:run` em produção, nomenclatura PascalCase e caminho `src/database/migrations/`
- `acceptance.md`: seção de cobertura mínima de testes com 6 cenários e2e/unitários cobrindo conexão, validação Joi, auto-execução de migrations e `synchronize: false`

### Corrigido
- `README.md`: corrige escape de markdown na linha `Spec version` (renderizava `\*\*` em vez de negrito)

---

## [0.1.1] — 2026-05

### Corrigido
- `contracts.md`, `decisions.md`, `examples/configurar-database-module.md`: `config.get<T>()` substituído por `config.getOrThrow<T>()` em todas as referências ao `ConfigService` — alinha ao padrão do repositório e elimina o tipo `T | undefined` que causaria erro de compilação em strict mode
- `README.md`, `behavior.md`, `context.md`, `integration.md`: removido marcador `(a criar)` das referências a `setup/nestjs-health`, que agora está publicada

---

## [0.1.0] — 2026-05

### Adicionado
- Versão inicial da spec: `DatabaseModule` com `TypeOrmModule.forRootAsync`, variáveis de ambiente, `data-source.ts` para o CLI, scripts de migration com `--` para repasse correto de argumentos posicionais via pnpm, comportamento de auto-execução por ambiente, ADRs de decisão arquitetural (TypeORM, `synchronize: false`, migrations automáticas em desenvolvimento, glob de entidades, configuração SSL) e critérios de aceitação
