# Decisions — Setup NestJS — Logging Estruturado

> Decisões que podem parecer questionáveis sem contexto — evita que implementadores "melhorem" comportamentos deliberadamente escolhidos.

---

## ADR-001 — Winston como biblioteca de logging

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
O NestJS não impõe uma biblioteca de logging. É necessário escolher uma que suporte múltiplos transportes, rotação de arquivos e formatação JSON.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Pino | Alta performance; JSON nativo | Integração com NestJS menos direta; rotação de arquivo depende de ferramentas externas |
| Winston — **escolhida** | Madura; suporta múltiplos transportes; `winston-daily-rotate-file` disponível; integração direta implementando `LoggerService` | Levemente mais lenta que Pino em benchmarks sintéticos |

### Decisão
Usar Winston com `winston-daily-rotate-file`.

### Consequências
- Toda a configuração de transporte fica centralizada no `LoggerService`
- A diferença de performance em relação ao Pino é irrelevante para o volume de logs esperado em APIs REST típicas

---

## ADR-002 — LoggerService implementa NestLoggerService e substitui o logger padrão

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
O NestJS usa um logger interno para emitir mensagens de framework (bootstrap, erros de módulo, etc.). Se `LoggerService` não substituir o logger padrão, esses logs são emitidos sem formatação JSON.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Logger paralelo (sem substituir o padrão) | Mais simples | Logs internos do NestJS ficam fora do formato padronizado |
| Implementar `LoggerService` e chamar `app.useLogger()` — **escolhida** | Todos os logs, incluindo os do framework, passam pelo mesmo pipeline | Exige implementar a interface completa de `NestLoggerService` |

### Decisão
`LoggerService` implementa a interface `LoggerService` de `@nestjs/common` e é registrado via `app.useLogger(app.get(LoggerService))` no `bootstrap()`.

### Consequências
- Todos os logs da aplicação (framework + código de negócio) têm o mesmo formato JSON

---

## ADR-003 — LoggerModule decorado com @Global()

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
`LogSanitizer` e `LoggerService` precisam ser injetáveis em qualquer ponto da aplicação — incluindo `HttpExceptionFilter`, que é um provider global definido em `setup/nestjs/nestjs-init` sem pertencer a nenhum módulo de feature. Sem `@Global()`, cada módulo de feature que precisar de `LogSanitizer` teria que importar `LoggerModule` explicitamente.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Sem `@Global()` — importar `LoggerModule` onde necessário | Dependências explícitas; testabilidade por módulo | Filtros e guards globais não conseguem injetar `LogSanitizer`; boilerplate de importação em cada feature module |
| `@Global()` — **escolhida** | Disponível em toda a aplicação com uma única importação em `AppModule`; filtros globais resolvem providers corretamente | Dependências de `LoggerService` e `LogSanitizer` ficam implícitas nos módulos de feature |

### Decisão
`LoggerModule` é decorado com `@Global()`. É importado uma única vez no `AppModule`.

### Consequências
- `LogSanitizer` e `LoggerService` são injetáveis em qualquer classe sem importação adicional de `LoggerModule`
- `HttpExceptionFilter` e outros providers globais resolvem `LogSanitizer` corretamente
- Módulos de feature não precisam declarar dependência de `LoggerModule` mesmo que usem o logger
