# Behavior — NestJS — Health Check (Liveness + Readiness)

---

## Objetivo

Garantir que todo projeto NestJS exponha endpoints padronizados de liveness e readiness, permitindo que orquestradores de container e load balancers tomem decisões corretas sobre o ciclo de vida e o roteamento de tráfego da aplicação.

---

## Fluxo principal

1. O pacote `@nestjs/terminus` é instalado via pnpm
2. As variáveis de ambiente de memória são adicionadas ao schema Joi em `src/config/env.validation.ts` e ao `.env.example`
3. O `HealthModule` é criado em `src/modules/health/health.module.ts`
4. O `HealthExceptionFilter` é criado em `src/modules/health/health.exception-filter.ts` e o `HealthController` é criado em `src/modules/health/health.controller.ts` decorado com `@UseFilters(HealthExceptionFilter)`, expondo `GET /health/live` e `GET /health/ready`
5. O `HealthModule` é registrado no `AppModule`
6. A aplicação passa a responder em `/health/live` (liveness) e `/health/ready` (readiness)

---

## Regras de negócio

- `/health/live` nunca verifica dependências externas — sua única responsabilidade é confirmar que o processo Node.js está em execução e respondendo
- `/health/ready` verifica banco de dados e memória (heap e RSS) antes de sinalizar que a aplicação está apta a receber tráfego
- Quando todos os indicadores de `/health/ready` passam, a resposta é HTTP 200; quando qualquer indicador falha, a resposta é HTTP 503
- `/health/live` responde sempre HTTP 200 enquanto o processo estiver no ar — uma falha no banco não altera seu comportamento
- Os limites de memória são lidos do `ConfigService` — os valores padrão são 150 MB para heap e 300 MB para RSS
- Ambos os endpoints retornam JSON no formato padrão do Terminus (ver `contracts.md`)
- O `HealthModule` não é global e não exporta nenhum provider — nenhum outro módulo depende dele
- O `AllExceptionsFilter` global do `setup/nestjs-init` intercepta toda exceção, incluindo `HealthCheckError` (que o Terminus lança e que extends `HttpException`) — sem tratamento específico, o corpo Terminus é destruído e substituído pelo formato de erro padrão da aplicação
- O `HealthController` deve ser decorado com `@UseFilters(HealthExceptionFilter)`, um filtro específico que captura `HealthCheckError` antes do filtro global e devolve o corpo original do Terminus com HTTP 503; os demais erros não capturados pelo filtro continuam sendo tratados pelo `AllExceptionsFilter`
- O `HealthController` é documentado no Swagger com `@ApiTags('health')` e `@ApiExtraModels(HealthIndicatorResultDto, HealthCheckResponseDto)` no nível do controller; cada endpoint recebe `@ApiOperation`, `@ApiOkResponse({ type: HealthCheckResponseDto })` e, onde aplicável, `@ApiServiceUnavailableResponse`

---

## Edge cases

| Situação | Comportamento esperado |
|----------|----------------------|
| Banco indisponível | `/health/live` retorna 200; `/health/ready` retorna 503 com `database` marcado como `down` |
| Banco se recupera | `/health/ready` volta a retornar 200 automaticamente na próxima requisição |
| Memória heap excede o limite | `/health/ready` retorna 503 com `memory_heap` marcado como `down` |
| Memória RSS excede o limite | `/health/ready` retorna 503 com `memory_rss` marcado como `down` |
| Múltiplos indicadores falhando simultaneamente | `/health/ready` retorna 503 com todos os indicadores com falha listados em `error` |
| Aplicação em inicialização (bootstrap ainda em andamento) | `/health/live` pode retornar 503 brevemente até o servidor HTTP estar pronto — comportamento do NestJS, não desta spec |
| Variável de limite de memória ausente | O schema Joi usa o valor padrão configurado — não impede a inicialização |

---

## Restrições

- Não adicionar autenticação nos endpoints de health — eles devem ser acessíveis por sistemas de infraestrutura sem credenciais
- Não verificar dependências externas em `/health/live` — qualquer verificação que possa falhar vai para `/health/ready`
- Não registrar o `HealthModule` em módulos de feature — apenas no `AppModule`
- Não suprimir nem reformatar erros do Terminus — o `HealthExceptionFilter` captura `HealthCheckError` exclusivamente para preservar o corpo original do Terminus; não deve transformar, logar ou silenciar a resposta

---

## O que esta spec NÃO cobre

- Métricas de desempenho → ver `observability/nestjs-metrics` (a criar)
- Logging estruturado → ver `observability/nestjs-logging` (a criar)
- Verificação de dependências externas além do banco (ex: Redis, filas, APIs externas) → estender esta spec conforme necessário e documentar em `decisions.md`
- Autenticação em endpoints de health → decisão arquitetural registrada em ADR-001
