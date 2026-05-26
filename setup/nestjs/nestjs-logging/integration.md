# Integration — Setup NestJS — Logging Estruturado

---

## Dependências

| Módulo | Obrigatório | O que usa |
|--------|-------------|-----------|
| `setup/nestjs/nestjs-init` | ✅ | Estrutura base do projeto, schema de validação de env, `HttpExceptionFilter` |
| `observability/structured-log` | ✅ | Comportamento que esta spec implementa |

---

## Dependentes

| Módulo | Como usa este módulo |
|--------|----------------------|
| Qualquer módulo da aplicação | Usa `new Logger(NomeDaClasse.name)` para logar; delega para `LoggerService` automaticamente |

---

## Pontos de integração no projeto

| Ponto | Integração |
|-------|-----------|
| `AppModule` | Importa `LoggerModule`; registra `LoggingInterceptor` como `APP_INTERCEPTOR` |
| `bootstrap()` | Chama `app.useLogger(app.get(LoggerService))` — requer `LoggerModule` já importado no `AppModule` |
| `HttpExceptionFilter` | Injeta `LoggerService` e `LogSanitizer`; chama `sanitizer.sanitize()` antes de `logger.warn()` ou `logger.error()` |
| Schema de env | Adiciona `LOG_LEVEL`, `LOG_MAX_SIZE`, `LOG_MAX_FILES`, `LOG_DIR` ao schema de validação do projeto |

---

## Eventos emitidos

Esta spec não define eventos de domínio.

---

## Eventos consumidos

Esta spec não consome eventos de outros módulos.

---

## Impacto em outros sistemas

- Após `app.useLogger(app.get(LoggerService))`, todos os logs internos do NestJS (bootstrap, providers, erros de módulo) passam pelo `LoggerService` e ganham formatação JSON
