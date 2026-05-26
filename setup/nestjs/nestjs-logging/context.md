# Context — Setup NestJS — Logging Estruturado

---

## Plataformas suportadas

| Plataforma | Suportada | Observações |
|------------|-----------|-------------|
| API REST | ✅ | Comportamento principal |
| Web SPA | ❌ | |
| Mobile (iOS/Android) | ❌ | |
| Backend (serviço interno) | ✅ | Inclui workers BullMQ quando integrados com `setup/nestjs/nestjs-queue` |

## Pré-requisitos

- [ ] Projeto NestJS inicializado conforme `setup/nestjs/nestjs-init`
- [ ] `observability/structured-log` lida e compreendida antes desta spec

## Escopo

**Dentro do escopo:**
- Implementação de `LoggerService` substituindo o logger padrão do NestJS
- Implementação de `LoggingInterceptor` para registrar metadados de requisições HTTP
- Implementação de `LogSanitizer` com as regras de `observability/structured-log`
- Configuração dos transportes Winston (arquivo JSON + console colorido)
- Registro e integração dos componentes no `AppModule` e no `bootstrap()`
- Adições ao `agents.md` do projeto

**Fora do escopo:**
- Geração e propagação de `correlationId` → ver `observability/correlation-id` (a criar)
- Captura de erros em serviços externos (Sentry) → ver `observability/error-tracking` (a criar)
- Implementação do `HttpExceptionFilter` → definido em `setup/nestjs/nestjs-init`
- Configuração de infraestrutura de coleta de logs (ELK, Loki, etc.)

## Bibliotecas externas

| Pacote | Versão mínima | Uso |
|--------|--------------|-----|
| `winston` | `^3.0.0` | Logger principal |
| `winston-daily-rotate-file` | `^5.0.0` | Transporte com rotação diária; tipos bundled — nenhum `@types/` adicional necessário |
