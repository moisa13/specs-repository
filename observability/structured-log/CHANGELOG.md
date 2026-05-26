# Changelog — Registro Estruturado de Logs

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

## [0.1.0] — 2026-05

### Adicionado
- Versão inicial da spec
- Fluxo de geração e propagação de `correlationId` via CLS
- Regras de sanitização por nome exato, padrão regex e URL
- Classificação de exceções por faixa HTTP (4xx `warn` / 5xx `error`)
- Contratos de `LogEntry`, `LogLevel` e configuração de ambiente
- Critérios de aceitação para fluxo principal, sanitização e transportes
- ADR-001 a ADR-004
