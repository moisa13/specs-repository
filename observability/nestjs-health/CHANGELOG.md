# Changelog — NestJS — Health Check (Liveness + Readiness)

> Versões seguem Semantic Versioning: MAJOR.MINOR.PATCH
>
> - **MAJOR** — mudança que quebra implementações existentes
> - **MINOR** — nova regra ou edge case (implementações existentes precisam de revisão)
> - **PATCH** — correção de texto, sem mudança de comportamento

---

## [0.1.0] — 2026-05

### Adicionado
- Versão inicial da spec: `HealthModule` com endpoints `/health/live` e `/health/ready`, indicadores de banco de dados (TypeORM), memória heap e RSS, variáveis de ambiente para limites de memória, formato de resposta Terminus e quatro ADRs de decisão arquitetural
