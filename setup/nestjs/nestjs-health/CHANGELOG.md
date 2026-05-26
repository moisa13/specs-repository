# Changelog — NestJS — Health Check (Liveness + Readiness)

> Versões seguem Semantic Versioning: MAJOR.MINOR.PATCH
>
> - **MAJOR** — mudança que quebra implementações existentes
> - **MINOR** — nova regra ou edge case (implementações existentes precisam de revisão)
> - **PATCH** — correção de texto, sem mudança de comportamento

---

## [0.3.0] — 2026-05

### Adicionado
- `behavior.md`: indicador Redis condicional ao `/health/ready` quando `REDIS_HOST` está presente; edge cases de Redis indisponível e recuperação automática
- `contracts.md`: indicador `redis` na tabela de indicadores com coluna de condição de ativação; instrução no `agents.md` atualizada
- `acceptance.md`: AC-09 (Redis up quando acessível), AC-10 (503 quando Redis indisponível); AC-09 anterior renumerado para AC-11
- `decisions.md`: ADR-007 — indicador Redis condicional baseado em `REDIS_HOST`
- `integration.md`: dependência opcional de `setup/nestjs/nestjs-queue`
- `examples/health-redis-indicator.md`: exemplo de implementação do `RedisHealthIndicator`

---

## [0.2.0] — 2026-05

### Adicionado
- `contracts.md`: seção "Adições ao agents.md" com 6 convenções — separação liveness/readiness com HTTP status explícitos (200/503), verificação paralela de indicadores com reporte individual, limites de memória padrão via Joi, endpoints sem autenticação e preservação do `HealthExceptionFilter`
- `acceptance.md`: seção de cobertura mínima de testes com 6 cenários e2e/unitários cobrindo liveness, readiness, banco down e limites de memória

### Corrigido
- `contracts.md`: `HealthIndicatorResultDto` atualizado para `{ status: string; message?: string }` — campo `message` estava ausente e é enviado pelo Terminus em indicadores com falha (HTTP 503)
- `examples/health-controller.md`: `HealthIndicatorResultDto` recebe `message?: string` com `@ApiProperty({ required: false })` alinhando o exemplo ao contrato
- `examples/registrar-health-no-app-module.md`: corrige ordem — `AppInfoModule` deve preceder `HealthModule` no `AppModule`, respeitando a progressão de aplicação das specs
- `behavior.md`, `examples/health-exception-filter.md`, `examples/health-module.md`: caminhos atualizados de `src/health/` para `src/modules/health/`
- `README.md`: corrige escape de markdown na linha `Status`; preenche campo `Revisado por`

---

## [0.1.0] — 2026-05

### Adicionado
- Versão inicial da spec: `HealthModule` com endpoints `/health/live` e `/health/ready`, indicadores de banco de dados (TypeORM), memória heap e RSS, variáveis de ambiente para limites de memória, formato de resposta Terminus e quatro ADRs de decisão arquitetural
