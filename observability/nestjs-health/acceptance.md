# Acceptance Criteria — NestJS — Health Check (Liveness + Readiness)

> Formato: **Dado** [contexto] **Quando** [ação] **Então** [resultado esperado]

---

## Fluxo principal

**AC-01 — Liveness retorna 200 com aplicação no ar**
- **Dado** que a aplicação está em execução
- **Quando** `GET /health/live` é chamado
- **Então** a resposta é HTTP 200 com `status: "ok"` e o indicador `live` com `status: "up"`

**AC-02 — Readiness retorna 200 com todos os indicadores saudáveis**
- **Dado** que a aplicação está em execução, o PostgreSQL está acessível e o uso de memória está abaixo dos limites configurados
- **Quando** `GET /health/ready` é chamado
- **Então** a resposta é HTTP 200 com `status: "ok"` e os indicadores `database`, `memory_heap` e `memory_rss` com `status: "up"`

---

## Cenários de falha

**AC-03 — Readiness retorna 503 quando o banco está indisponível**
- **Dado** que o PostgreSQL não está acessível
- **Quando** `GET /health/ready` é chamado
- **Então** a resposta é HTTP 503 com `status: "error"` e o indicador `database` com `status: "down"` no campo `error`

**AC-04 — Liveness retorna 200 mesmo quando o banco está indisponível**
- **Dado** que o PostgreSQL não está acessível
- **Quando** `GET /health/live` é chamado
- **Então** a resposta é HTTP 200 com `status: "ok"` — a indisponibilidade do banco não afeta o liveness

**AC-05 — Readiness retorna 503 quando a memória heap excede o limite**
- **Dado** que o uso de memória heap da aplicação excede `HEALTH_MEMORY_HEAP_THRESHOLD_MB`
- **Quando** `GET /health/ready` é chamado
- **Então** a resposta é HTTP 503 com `status: "error"` e o indicador `memory_heap` com `status: "down"` no campo `error`

**AC-06 — Readiness retorna 503 quando a memória RSS excede o limite**
- **Dado** que o uso de memória RSS da aplicação excede `HEALTH_MEMORY_RSS_THRESHOLD_MB`
- **Quando** `GET /health/ready` é chamado
- **Então** a resposta é HTTP 503 com `status: "error"` e o indicador `memory_rss` com `status: "down"` no campo `error`

---

## Edge cases

**AC-07 — Readiness se recupera automaticamente após falha no banco**
- **Dado** que o banco estava indisponível e `/health/ready` retornava 503
- **Quando** o banco volta a ficar acessível e `GET /health/ready` é chamado novamente
- **Então** a resposta é HTTP 200 com todos os indicadores com `status: "up"`

**AC-08 — Múltiplos indicadores com falha são reportados simultaneamente**
- **Dado** que o banco está indisponível e o uso de memória heap excede o limite
- **Quando** `GET /health/ready` é chamado
- **Então** a resposta é HTTP 503 com `database` e `memory_heap` ambos listados no campo `error`

**AC-09 — Limites de memória padrão são aplicados quando variáveis não estão definidas**
- **Dado** que `HEALTH_MEMORY_HEAP_THRESHOLD_MB` e `HEALTH_MEMORY_RSS_THRESHOLD_MB` não estão definidas no `.env`
- **Quando** a aplicação sobe
- **Então** os limites de 150 MB (heap) e 300 MB (RSS) são aplicados automaticamente pelo valor padrão do Joi

---

## O que caracteriza uma implementação incorreta

- `/health/live` verificando o banco ou qualquer dependência externa
- `/health/ready` retornando 200 quando o banco está inacessível
- Endpoints de health protegidos por autenticação
- `HealthModule` registrado em módulo de feature em vez do `AppModule`
- Limites de memória hardcoded no código em vez de lidos do `ConfigService`
- Formato de resposta diferente do padrão Terminus

---

## Cobertura mínima de testes

> **Framework padrão:** Jest — instalado pelo NestJS CLI. Configuração completa de testes em `setup/testing` (a criar).

- [ ] `GET /health/live` retorna 200 com aplicação no ar (e2e)
- [ ] `GET /health/ready` retorna 200 com banco acessível e memória dentro dos limites (e2e)
- [ ] `GET /health/ready` retorna 503 com banco indisponível (e2e)
- [ ] `GET /health/live` retorna 200 mesmo com banco indisponível (e2e)
- [ ] Múltiplos indicadores com falha são reportados simultaneamente no campo `error` (e2e)
- [ ] Limites de memória padrão (150 MB heap, 300 MB RSS) são aplicados quando variáveis não estão definidas (teste unitário no schema Joi)
