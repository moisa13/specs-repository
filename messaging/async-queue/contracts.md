# Contracts — Processamento Assíncrono com Fila

> Contratos de dados agnósticos de linguagem. O agente deve mapear estes tipos para os equivalentes da stack utilizada.

---

## Entidades

### Job

| Campo | Tipo | Obrigatório | Restrições | Descrição |
|-------|------|-------------|------------|-----------|
| `id` | string | ✅ | único por fila | Identificador do job; gerado automaticamente se não fornecido |
| `queueName` | string | ✅ | non-empty | Nome da fila à qual o job pertence |
| `jobType` | string | ✅ | non-empty | Tipo do processamento a ser executado |
| `payload` | object | ✅ | serializável em JSON | Dados necessários para o processamento |
| `status` | enum | ✅ | ver `JobStatus` | Estado atual do job |
| `attempts` | integer | ✅ | >= 0 | Número de tentativas realizadas até o momento |
| `createdAt` | datetime | ✅ | UTC | Momento em que o job foi enfileirado |
| `processedAt` | datetime | ❌ | UTC, nullable | Momento em que o processamento foi concluído (sucesso ou falha definitiva) |
| `failReason` | string | ❌ | nullable | Mensagem de erro registrada na última falha |

### JobStatus

| Valor | Descrição |
|-------|-----------|
| `waiting` | Aguardando um worker disponível |
| `active` | Em processamento por um worker |
| `completed` | Processado com sucesso |
| `failed` | Esgotou o número de tentativas; movido para DLQ |
| `delayed` | Aguardando o delay antes de entrar em `waiting` |

---

## Entrada — Produção de Job

| Campo | Tipo | Obrigatório | Validação |
|-------|------|-------------|-----------|
| `queueName` | string | ✅ | non-empty; fila deve estar registrada |
| `jobType` | string | ✅ | non-empty |
| `payload` | object | ✅ | serializável em JSON |
| `options.jobId` | string | ❌ | se omitido, gerado automaticamente |
| `options.delay` | integer | ❌ | milissegundos; >= 0; padrão: 0 |
| `options.priority` | integer | ❌ | maior valor = maior prioridade; padrão: 0 |
| `options.attempts` | integer | ❌ | >= 1; padrão: 3 |
| `options.backoff.type` | enum | ❌ | `exponential` \| `fixed`; padrão: `exponential` |
| `options.backoff.delay` | integer | ❌ | delay base em ms; padrão: 1000 |

---

## Saída — Confirmação de Enfileiramento

| Campo | Tipo | Sempre presente | Descrição |
|-------|------|-----------------|-----------|
| `jobId` | string | ✅ | ID do job criado na fila |
| `queueName` | string | ✅ | Nome da fila em que o job foi inserido |
| `status` | string | ✅ | `waiting` ou `delayed` conforme o `delay` informado |

---

## Erros do Produtor

| Código de erro | Situação |
|----------------|----------|
| `QUEUE_NOT_FOUND` | A fila informada não está registrada |
| `INVALID_PAYLOAD` | O payload não é serializável em JSON |
| `QUEUE_UNAVAILABLE` | O broker de filas está inacessível no momento da produção |

---

## Resultado do Consumidor

O consumidor não retorna dados ao produtor. Ao concluir o processamento, deve:

- **Sucesso:** finalizar a execução normalmente; o sistema marca o job como `completed`
- **Falha recuperável:** lançar um erro; o sistema aplica retry conforme configuração
- **Falha definitiva:** o sistema move o job para DLQ após esgotar as tentativas
