# Contracts — Canal de Comunicação em Tempo Real

> Contratos de dados agnósticos de linguagem. O agente deve mapear estes tipos para os equivalentes da stack utilizada.

---

## Entidades

### Event

Estrutura base de qualquer evento que trafega pelo canal, em ambas as direções.

| Campo | Tipo | Obrigatório | Restrições | Descrição |
|-------|------|-------------|------------|-----------|
| `event` | string | ✅ | non-empty | Nome do evento; identifica o tipo e determina o handler que o processa |
| `payload` | object | ✅ | serializável em JSON | Dados transportados pelo evento |

### Connection

Representa uma conexão ativa de um cliente com o servidor.

| Campo | Tipo | Obrigatório | Restrições | Descrição |
|-------|------|-------------|------------|-----------|
| `connectionId` | string | ✅ | único globalmente | Identificador da conexão; gerado pelo servidor no handshake |
| `clientId` | string | ✅ | non-empty | Identidade do cliente autenticado (ex: `userId`) |
| `connectedAt` | datetime | ✅ | UTC | Momento em que a conexão foi aceita |

---

## Entrada — Handshake

| Campo | Tipo | Obrigatório | Validação |
|-------|------|-------------|-----------|
| `token` | string | ✅ | Token de autenticação válido; o mecanismo de validação é definido pelo sistema de autenticação do projeto |

A forma exata de transporte do token (query param, header, cookie) é definida pela spec de implementação.

---

## Entrada — Evento do cliente para o servidor

| Campo | Tipo | Obrigatório | Validação |
|-------|------|-------------|-----------|
| `event` | string | ✅ | non-empty |
| `payload` | object | ✅ | serializável em JSON |

---

## Saída — Evento do servidor para o cliente

| Campo | Tipo | Sempre presente | Descrição |
|-------|------|-----------------|-----------|
| `event` | string | ✅ | Nome do evento |
| `payload` | object | ✅ | Dados do evento |

---

## Erros de conexão

| Código de erro | Situação |
|----------------|----------|
| `UNAUTHORIZED` | Token ausente, inválido ou expirado no handshake |
