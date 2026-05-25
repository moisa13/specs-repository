# Context — NestJS — Health Check (Liveness + Readiness)

---

## Plataformas suportadas

| Plataforma | Suportada | Observações |
|------------|-----------|-------------|
| API REST | ✅ | Plataforma alvo desta spec |
| Web SPA | ❌ | |
| Mobile (iOS/Android) | ❌ | |
| Backend (serviço interno) | ✅ | Mesma configuração; ajustar limites de memória conforme o ambiente |

---

## Pré-requisitos

- [ ] `setup/nestjs-init` aplicado — `ConfigService` global disponível
- [ ] `setup/nestjs-database` aplicado — conexão TypeORM disponível para o indicador de banco
- [ ] Node.js >= 20
- [ ] pnpm instalado

---

## Escopo

**Dentro do escopo:**
- Instalação do pacote `@nestjs/terminus`
- `HealthModule` com `HealthController` expondo `/health/live` e `/health/ready`
- Indicador de liveness (processo ativo, sem dependências externas)
- Indicador de readiness com verificação de banco de dados via `TypeOrmHealthIndicator`
- Indicador de readiness com verificação de memória heap e RSS via `MemoryHealthIndicator`
- Limites de memória configuráveis via variáveis de ambiente
- Registro do `HealthModule` no `AppModule`

**Fora do escopo:**
- Métricas de desempenho (latência, throughput, taxa de erros) → ver `observability/nestjs-metrics` (a criar)
- Logging estruturado → ver `observability/nestjs-logging` (a criar)
- Verificação de dependências externas além do banco (ex: Redis, serviços HTTP) → estender esta spec conforme necessário
- Autenticação nos endpoints de health → endpoints são públicos por design (ver ADR-001)

---

## Considerações de ambiente

- **Desenvolvimento:** ambos os endpoints funcionam normalmente; limites de memória podem ser ajustados via `.env` para refletir a realidade local.
- **Produção:** `/health/live` é usado pelo Kubernetes como liveness probe; `/health/ready` como readiness probe. Limites de memória devem refletir os recursos alocados ao container.
- **Test:** os critérios mínimos de cobertura desta spec estão em `acceptance.md` — a infraestrutura de testes (configuração do Jest, helpers e setup de banco para e2e) é definida em `setup/testing` (a criar).
