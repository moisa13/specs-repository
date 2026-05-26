# Context — NestJS Filas Assíncronas (BullMQ + Redis)

---

## Plataformas suportadas

| Plataforma | Suportada | Observações |
|------------|-----------|-------------|
| API REST | ✅ | Produção de jobs a partir de controllers e services |
| Backend (serviço interno) | ✅ | Produção e consumo entre módulos do monólito |
| Web SPA | ❌ | Filas operam no lado do servidor |
| Mobile (iOS/Android) | ❌ | Filas operam no lado do servidor |

## Pré-requisitos

- [ ] `setup/nestjs/nestjs-init` aplicado — `ConfigService` e estrutura global disponíveis
- [ ] Instância Redis acessível (local, Docker ou managed)
- [ ] Pacotes instalados: `@nestjs/bullmq`, `bullmq`, `ioredis`, `@bull-board/api`, `@bull-board/nestjs`, `@bull-board/express`, `express-basic-auth`

## Escopo

**Dentro do escopo:**
- Configuração do BullMQ com conexão Redis via `ConfigService`
- `QueueModule` centralizado com registro de todas as filas da aplicação
- `QueueService` como único ponto de produção de jobs
- Definição de Processors nos módulos de feature
- Variáveis de ambiente para conexão Redis e opções padrão de jobs
- Estrutura de diretórios e convenções de nomenclatura
- Painel de inspeção de filas via Bull Board

**Fora do escopo:**
- Comportamento de retry e DLQ → definido em `messaging/async-queue`
- Agendamento de tarefas sem fila persistente → use `@nestjs/schedule`
- Configuração de Redis Cluster ou Redis Sentinel → responsabilidade de infraestrutura
- Métricas de filas para coleta externa (Prometheus, OpenTelemetry)

## Considerações de ambiente

- **Desenvolvimento:** Redis local ou via Docker; `REDIS_PASSWORD` pode ser omitida
- **Produção:** Redis gerenciado com senha obrigatória; `QUEUE_DEFAULT_ATTEMPTS` e `QUEUE_DEFAULT_BACKOFF_DELAY` devem ser definidos explicitamente
