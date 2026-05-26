# Contracts — Setup NestJS — Logging Estruturado

> Contratos específicos para a implementação NestJS. Para os contratos de comportamento (LogEntry, LogLevel, regras de sanitização), ver `observability/structured-log/contracts.md`.

---

## Variáveis de ambiente

| Variável | Tipo | Obrigatório | Padrão (produção) | Padrão (dev) | Validação |
|----------|------|-------------|-------------------|--------------|-----------|
| `LOG_LEVEL` | string | ❌ | `warn` | `debug` | enum: `debug`, `verbose`, `info`, `warn`, `error` |
| `LOG_MAX_SIZE` | string | ❌ | `20m` | `20m` | formato: `{número}{k\|m\|g}` |
| `LOG_MAX_FILES` | string | ❌ | `14d` | `14d` | formato: `{número}d` |
| `LOG_DIR` | string | ❌ | `logs/` | `logs/` | path relativo ou absoluto |

Todas as variáveis devem ser adicionadas ao schema de validação de ambiente do projeto (conforme `setup/nestjs/nestjs-init`).

---

## Schema de validação (Joi)

```typescript
LOG_LEVEL: Joi.string()
  .valid('debug', 'verbose', 'info', 'warn', 'error')
  .default(process.env.NODE_ENV === 'production' ? 'warn' : 'debug'),
LOG_MAX_SIZE: Joi.string().default('20m'),
LOG_MAX_FILES: Joi.string().default('14d'),
LOG_DIR: Joi.string().default('logs/'),
```

> O default de `LOG_LEVEL` é resolvido em tempo de inicialização via `process.env.NODE_ENV`. Em produção o valor é `warn`; em qualquer outro ambiente é `debug`. A variável pode ser sobrescrita explicitamente no `.env`.

---

## Convenções de nomenclatura

| Artefato | Nome da classe | Localização |
|----------|---------------|-------------|
| Módulo | `LoggerModule` | `src/infrastructure/logger/logger.module.ts` |
| Logger service | `LoggerService` | `src/infrastructure/logger/logger.service.ts` |
| Sanitizer | `LogSanitizer` | `src/infrastructure/logger/log-sanitizer.ts` |
| Interceptor | `LoggingInterceptor` | `src/infrastructure/logger/logging.interceptor.ts` |

---

## Adições ao agents.md

Ao aplicar esta spec, acrescentar a seção abaixo ao `agents.md` do projeto:

```markdown
## Logging (setup/nestjs/nestjs-logging)

- O logger central é `LoggerService` em `src/infrastructure/logger/logger.service.ts`; nunca instanciar Winston diretamente fora dele
- Para logar em qualquer classe: `private readonly logger = new Logger(NomeDaClasse.name)` — NestJS delega automaticamente para `LoggerService`
- Nunca logar dados de entrada sem antes passar pelo `LogSanitizer.sanitize()`
```
