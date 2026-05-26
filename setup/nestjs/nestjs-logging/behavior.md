# Behavior — Setup NestJS — Logging Estruturado

---

## Objetivo

Implementar o comportamento de `observability/structured-log` em um projeto NestJS, fornecendo os componentes necessários para logging estruturado com sanitização e exportação para arquivo via Winston.

---

## Componentes

### LoggerModule

Módulo NestJS decorado com `@Global()`. Deve:
- Declarar `LoggerService`, `LogSanitizer` e `LoggingInterceptor` em `providers`
- Exportar apenas `LoggerService` e `LogSanitizer` — `LoggingInterceptor` não é exportado pois é registrado via `APP_INTERCEPTOR`, não injetado diretamente
- Ser importado uma única vez no `AppModule`

A anotação `@Global()` garante que `LoggerService` e `LogSanitizer` estejam disponíveis para injeção em qualquer parte da aplicação — incluindo filtros e guards globais — sem que cada módulo precise importar `LoggerModule` explicitamente.

### LogSanitizer

Serviço `@Injectable()`. Aplica as regras de sanitização de `observability/structured-log`.

Responsabilidades:
- Receber um objeto, array ou valor primitivo como entrada
- Aplicar sanitização recursiva até 5 níveis de profundidade
- Retornar a versão sanitizada sem modificar o objeto original (cópia profunda)
- Redijir campos sensíveis para `[REDACTED]` conforme regras de `contracts.md`

### LoggerService

Substitui o logger padrão do NestJS. Deve:
- Implementar a interface `LoggerService` de `@nestjs/common`
- Ser registrado no `bootstrap()` via `app.useLogger(app.get(LoggerService))`
- Injetar `LogSanitizer` para sanitizar entradas antes de gravar
- Configurar os seguintes transportes Winston:
  - `DailyRotateFile` para todos os níveis — arquivo `%DATE%.log`, formato JSON
  - `DailyRotateFile` exclusivo para `error` — arquivo `%DATE%-error.log`, formato JSON
  - `Console` — apenas quando `NODE_ENV !== 'production'`; formato texto colorido com timestamp

### LoggingInterceptor

Interceptor NestJS que registra cada requisição HTTP:
- Implementar `NestInterceptor`
- No início do `intercept()`: capturar `Date.now()` como timestamp de início
- No `pipe(tap(...))`: logar método HTTP, URL, status de resposta e duração em ms
- Usar nível `log` (equivalente a `info`) para o registro da requisição concluída
- Ser registrado como `APP_INTERCEPTOR` no `AppModule`

---

## Fluxo de inicialização

1. `LoggerModule` importado no `AppModule` — expõe `LoggerService` e `LogSanitizer` para resolução pelo container NestJS
2. `LoggingInterceptor` registrado como `APP_INTERCEPTOR` no `AppModule`
3. No `bootstrap()`: `app.useLogger(app.get(LoggerService))` substitui o logger padrão do NestJS

> A importação de `LoggerModule` no `AppModule` é obrigatória antes do passo 3 — sem ela, `app.get(LoggerService)` lança exceção de provider não encontrado no bootstrap.

---

## Fluxo de uma requisição

```
Request
  → LoggingInterceptor (captura Date.now())
  → Controller / Service (Logger(NomeDaClasse.name) delega para LoggerService)
  → LoggingInterceptor (loga método, URL, status, duração)

Se exceção:
  → HttpExceptionFilter (classifica por faixa HTTP, chama LogSanitizer, chama LoggerService)
```

---

## Regras de implementação

- O `LogSanitizer` nunca modifica o objeto original — retorna sempre uma cópia sanitizada
- Nenhum componente desta spec captura ou envia erros para serviços externos — essa responsabilidade pertence ao `HttpExceptionFilter` em integração com `observability/error-tracking`
- O `LoggerModule` é `@Global()` e exporta `LoggerService` e `LogSanitizer` — sem essa anotação, filtros e guards globais que dependam de `LogSanitizer` falham na resolução de provider
- Todo `console.log` em `main.ts` (porta, URL do Swagger, etc.) deve ser substituído por chamadas ao `LoggerService` obtido via `app.get(LoggerService)`, após `app.useLogger()` — garante que todo output de inicialização passe pelo pipeline Winston

---

## O que esta spec NÃO cobre

- Geração e propagação de `correlationId` → ver `observability/correlation-id` (a criar)
- Integração com Sentry ou outro serviço externo → ver `observability/error-tracking` (a criar)
- Implementação do `HttpExceptionFilter` → definido em `setup/nestjs/nestjs-init`
- Configuração de infraestrutura de coleta de logs → escopo de infraestrutura
