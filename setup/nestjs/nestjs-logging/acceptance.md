# Acceptance Criteria — Setup NestJS — Logging Estruturado

> Formato: **Dado** [contexto] **Quando** [ação] **Então** [resultado esperado]

---

## LoggerService

**AC-01 — LoggerService substitui o logger padrão do NestJS**
- **Dado** que `app.useLogger(app.get(LoggerService))` está no `bootstrap()`
- **Quando** o NestJS emite um log interno (ex: "Application is running on port 3000")
- **Então** a entrada é formatada e gravada pelo `LoggerService`, não pelo logger padrão

---

## LoggingInterceptor

**AC-02 — Metadados de requisição registrados**
- **Dado** que uma requisição `GET /users/42` retorna 200
- **Quando** o `LoggingInterceptor` registra a conclusão
- **Então** a entrada contém `method: GET`, `url: /users/42`, `statusCode: 200` e `duration` em ms

---

## LogSanitizer

**AC-03 — Objeto original não modificado**
- **Dado** que `LogSanitizer.sanitize(obj)` é chamado com um objeto contendo `password`
- **Quando** o método retorna
- **Então** o objeto original permanece inalterado
- **E** o objeto retornado tem `password: '[REDACTED]'`

---

## Transportes Winston

**AC-04 — Arquivos de log criados corretamente**
- **Dado** que a aplicação está rodando
- **Quando** uma entrada de log é gerada
- **Então** o arquivo `logs/YYYY-MM-DD.log` existe e contém a entrada em formato JSON

**AC-05 — Arquivo de erros separado**
- **Dado** que uma entrada de nível `error` é gerada
- **Quando** gravada
- **Então** aparece em `logs/YYYY-MM-DD.log` e em `logs/YYYY-MM-DD-error.log`
- **E** uma entrada `warn` aparece apenas em `logs/YYYY-MM-DD.log`

**AC-06 — Console desativado em produção**
- **Dado** que `NODE_ENV=production`
- **Quando** a aplicação inicia
- **Então** nenhum transporte de console está registrado no Winston

---

## LoggerModule

**AC-07 — LogSanitizer injetável em filtros globais sem importação adicional**
- **Dado** que `LoggerModule` está importado no `AppModule` e decorado com `@Global()`
- **Quando** `HttpExceptionFilter` injeta `LogSanitizer` sem que nenhum módulo de feature declare `LoggerModule` em seus `imports`
- **Então** a injeção é resolvida corretamente pelo container NestJS na inicialização

---

## O que caracteriza uma implementação incorreta

- `new Logger()` do NestJS não delega para `LoggerService` (falta `app.useLogger()`)
- `LogSanitizer` modifica o objeto original em vez de retornar uma cópia
- Transporte de console ativo em produção
- `LoggerModule` sem `@Global()` — filtros e guards globais falham na resolução de `LogSanitizer`

---

## Cobertura mínima de testes

- [ ] `LogSanitizer`: sanitização por nome exato, por padrão regex e em URL
- [ ] `LogSanitizer`: recursividade até 5 níveis; campo no 6º nível não sanitizado
- [ ] `LogSanitizer`: objeto original não modificado
- [ ] `LoggingInterceptor`: entrada de log contém método, URL, status e duração
