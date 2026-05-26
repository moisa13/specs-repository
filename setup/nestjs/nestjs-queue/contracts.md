# Contracts — NestJS Filas Assíncronas (BullMQ + Redis)

---

## Variáveis de ambiente

As variáveis abaixo devem ser adicionadas ao schema Joi em `src/config/env.validation.ts` e ao `.env.example`.

| Variável | Tipo | Obrigatório | Validação Joi | Descrição |
|----------|------|-------------|---------------|-----------|
| `REDIS_HOST` | string | ✅* | `Joi.string().optional()` | Host da instância Redis |
| `REDIS_PORT` | number | ❌ | `Joi.number().default(6379)` | Porta da instância Redis |
| `REDIS_PASSWORD` | string | ❌ | `Joi.string().optional()` | Senha do Redis; omitir em ambientes sem autenticação |
| `REDIS_DB` | number | ❌ | `Joi.number().integer().min(0).max(15).default(0)` | Banco lógico do Redis (0–15); usar para isolar as chaves do BullMQ quando a instância é compartilhada |
| `QUEUE_DEFAULT_ATTEMPTS` | number | ❌ | `Joi.number().default(3)` | Número máximo de tentativas para todos os jobs |
| `QUEUE_DEFAULT_BACKOFF_DELAY` | number | ❌ | `Joi.number().default(1000)` | Delay em ms para o backoff padrão: base para `exponential`, constante para `fixed` |
| `BULL_BOARD_USER` | string | ✅* | `Joi.string().optional()` | Usuário para autenticação Basic Auth do painel `/queues` |
| `BULL_BOARD_PASSWORD` | string | ✅* | `Joi.string().optional()` | Senha para autenticação Basic Auth do painel `/queues` |

> **✅\*** Marcado como `optional()` no schema Joi global para que projetos sem Redis possam inicializar sem esta spec aplicada. Quando `QueueModule` está carregado no `AppModule`, `REDIS_HOST`, `BULL_BOARD_USER` e `BULL_BOARD_PASSWORD` são obrigatórios em runtime — o `BullModule.forRootAsync` usa `getOrThrow` internamente e a aplicação falha na inicialização se ausentes.

Adições ao `.env.example`:

```dotenv
# Host da instância Redis
REDIS_HOST=localhost

# Porta da instância Redis
REDIS_PORT=6379

# Senha do Redis (omitir em ambientes sem autenticação)
# REDIS_PASSWORD=

# Banco lógico do Redis (0–15); usar quando a instância Redis é compartilhada com outros serviços
REDIS_DB=0

# Número máximo de tentativas para todos os jobs
QUEUE_DEFAULT_ATTEMPTS=3

# Delay base de backoff em ms (exponential) ou delay fixo (fixed)
QUEUE_DEFAULT_BACKOFF_DELAY=1000

# Usuário para autenticação Basic Auth do painel /queues
BULL_BOARD_USER=admin

# Senha para autenticação Basic Auth do painel /queues
BULL_BOARD_PASSWORD=
```

---

## Estrutura de diretórios

```
src/
└── queue/
    ├── queue.module.ts         ← BullModule.forRootAsync + registerQueue de todas as filas
    ├── queue.service.ts        ← único provider com @InjectQueue; expõe QueueService.add()
    └── queue.constants.ts      ← nomes de fila como constantes exportadas (QUEUE_NAMES)
```

Processors ficam nos módulos de feature:

```
src/
└── [feature]/
    ├── [feature].module.ts     ← importa QueueModule; declara o Processor
    └── processors/
        └── [job-type].processor.ts
```

---

## API do QueueService

### `QueueService.add`

Enfileira um job em uma fila registrada.

**Assinatura:**
```typescript
add(
  queueName: QueueName,
  jobType: string,
  payload: Record<string, unknown>,
  options?: JobOptions
): Promise<{ jobId: string; queueName: string; status: 'waiting' | 'delayed' }>
```

> `QueueName` é o tipo derivado de `QUEUE_NAMES` exportado por `queue.constants.ts` (`type QueueName = (typeof QUEUE_NAMES)[keyof typeof QUEUE_NAMES]`). Usar o tipo em vez de `string` garante que somente filas registradas sejam aceitas em tempo de compilação.

**Parâmetros:**

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| `queueName` | `QueueName` | ✅ | Deve ser uma das constantes de `QUEUE_NAMES` |
| `jobType` | string | ✅ | Tipo do job; deve corresponder ao `job.name` despachado no método `process()` do Processor |
| `payload` | object | ✅ | Dados serializáveis em JSON para o Processor |
| `options.jobId` | string | ❌ | ID customizado; gerado automaticamente se omitido |
| `options.delay` | number | ❌ | Delay em ms antes de entrar em `waiting`; padrão: `0` |
| `options.priority` | number | ❌ | Maior valor = maior prioridade; padrão: `0` |
| `options.attempts` | number | ❌ | Sobrescreve `QUEUE_DEFAULT_ATTEMPTS` para este job |
| `options.backoff.type` | string | ❌ | Tipo de backoff: `'exponential'` ou `'fixed'`; padrão: `'exponential'` |
| `options.backoff.delay` | number | ❌ | Sobrescreve `QUEUE_DEFAULT_BACKOFF_DELAY` para este job |

**Erros lançados:**

| Exceção | Situação |
|---------|----------|
| `QueueNotFoundException` | `queueName` não registrada no `QueueModule` |
| `InvalidPayloadException` | `payload` não é serializável em JSON |
| `QueueUnavailableException` | Redis inacessível no momento da chamada |

---

## Convenção de nomes de fila

O arquivo `queue.constants.ts` deve exportar um objeto `QUEUE_NAMES` com todas as filas registradas:

```typescript
export const QUEUE_NAMES = {
  NOTIFICATIONS: 'notifications',
  REPORTS: 'reports',
} as const;
```

Toda referência a nome de fila no código deve usar `QUEUE_NAMES.[CHAVE]`, nunca uma string literal.

---

## Painel de inspeção de filas

### Pacotes

| Pacote | Finalidade |
|--------|------------|
| `@bull-board/api` | Core do painel; adapters para BullMQ |
| `@bull-board/nestjs` | Integração com o módulo NestJS |
| `@bull-board/express` | Adapter HTTP para Express (runtime padrão do NestJS) |
| `express-basic-auth` | Middleware de Basic Auth para proteger a rota `/queues` |

### Rota padrão

| Configuração | Valor padrão | Descrição |
|--------------|--------------|-----------|
| `route` | `/queues` | Caminho HTTP do painel; configurável em `BullBoardModule.forRoot` |

### Registro de filas no painel

Cada fila registrada em `BullModule.registerQueue` deve ter uma entrada correspondente em `BullBoardModule.forFeature`; caso contrário, a fila não aparece no painel.

---

## Adições ao agents.md

Ao aplicar esta spec, acrescentar a seção abaixo ao `agents.md` do projeto:

```markdown
## Filas assíncronas (setup/nestjs/nestjs-queue)

- Toda fila nova deve ser registrada em `src/queue/queue.module.ts` via `BullModule.registerQueue` e seu nome adicionado ao objeto `QUEUE_NAMES` em `src/queue/queue.constants.ts`; adicionar também em `BullBoardModule.forFeature` no mesmo arquivo
- `BullModule.registerQueue` deve ser chamado em `src/queue/queue.module.ts` (registro canônico) e também no módulo de feature que declara o Processor (requisito do `@nestjs/bullmq`)
- Nunca injetar `Queue` diretamente nos services — usar apenas `QueueService` para produção de jobs
- Processors ficam em `src/[feature]/processors/[job-type].processor.ts` e são declarados no módulo da feature
- Cada fila deve ter exatamente um Processor registrado na aplicação
- Nomes de fila devem ser referenciados via `QUEUE_NAMES` — nunca como string literal
```
