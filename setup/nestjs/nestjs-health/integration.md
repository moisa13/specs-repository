# Integration — NestJS — Health Check (Liveness + Readiness)

---

## Dependências

| Módulo | Obrigatório | O que usa |
|--------|-------------|-----------|
| `setup/nestjs/nestjs-init` | ✅ | `ConfigService` para leitura dos limites de memória |
| `setup/nestjs/nestjs-database` | ✅ | Conexão TypeORM injetada automaticamente pelo `TypeOrmHealthIndicator` |

---

## Dependentes

| Módulo | Como usa este módulo |
|--------|----------------------|
| Kubernetes / orquestrador | Configura liveness probe apontando para `GET /health/live` e readiness probe para `GET /health/ready` |
| Load balancer / proxy reverso | Usa `GET /health/ready` para decidir se o nó recebe tráfego |
| Ferramentas de monitoramento (ex: UptimeRobot, Grafana) | Monitoram `GET /health/ready` para alertas de disponibilidade |

---

## Eventos emitidos

Este módulo não emite eventos de domínio.

---

## Eventos consumidos

Este módulo não consome eventos de domínio.

---

## Impacto em outros sistemas

- Uma falha em `/health/ready` sinaliza ao Kubernetes para remover o pod do balanceamento — nenhuma requisição nova chega ao pod até que o endpoint volte a retornar 200
- Uma falha em `/health/live` sinaliza ao Kubernetes para reiniciar o pod — usar com cautela; verificações que dependem de recursos externos nunca devem estar no endpoint de liveness
- O `TypeOrmHealthIndicator` reutiliza a conexão já estabelecida pelo `DatabaseModule` — não abre uma nova conexão para o health check
