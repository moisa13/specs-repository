# Changelog — Setup NestJS — Logging Estruturado

> Versões seguem Semantic Versioning: MAJOR.MINOR.PATCH
>
> - **MAJOR** — mudança que quebra implementações existentes
> - **MINOR** — nova regra ou edge case (implementações existentes precisam de revisão)
> - **PATCH** — correção de texto, sem mudança de comportamento

---

## [Não lançado]

### Adicionado
-

### Alterado
-

### Removido
-

---

## [0.1.1] — 2026-05

### Corrigido
- `contracts.md`, `examples/`: caminhos `src/logger/` atualizados para `src/infrastructure/logger/` — reflete a reestruturação do diretório de infraestrutura no projeto

---

## [0.1.0] — 2026-05

### Adicionado
- Versão inicial da spec
- Descrição dos componentes: `LoggerService`, `LoggingInterceptor`, `LogSanitizer`
- Fluxo de inicialização e fluxo de requisição
- Contratos de variáveis de ambiente e schema Joi
- Convenções de nomenclatura e localização de arquivos
- Adições ao `agents.md`
- ADR-001, ADR-002 e ADR-003
