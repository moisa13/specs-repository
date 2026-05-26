# Decisions — Canal de Comunicação em Tempo Real

> Decisões que podem parecer questionáveis sem contexto — evita que implementadores "melhorem" comportamentos deliberadamente escolhidos.

---

## ADR-001 — Autenticação no handshake, não por evento

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto

Em um canal persistente, a autenticação pode ocorrer no momento da abertura da conexão (handshake) ou em cada evento enviado pelo cliente.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Autenticação por evento | Permite revalidar o token a cada mensagem | Overhead em cada evento; complexidade maior no handler |
| Autenticação no handshake (escolhida) | Simples; overhead único; alinhado com o modelo WebSocket | Token pode expirar após a conexão ser aceita sem revalidação automática |

### Decisão

A autenticação ocorre uma única vez, no handshake. Após a conexão ser aceita, ela permanece ativa independentemente da validade posterior do token. Revalidação periódica, se necessária, é responsabilidade da spec de feature ou de uma decisão de implementação.

### Consequências

- Implementação mais simples e com menor overhead
- O período entre expiração do token e encerramento da conexão é aceito como trade-off
- Specs de feature que precisam de revalidação devem definir esse comportamento explicitamente

---

## ADR-002 — Entrega agrupada por identidade do cliente, não por conexão

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto

Um mesmo usuário pode ter múltiplas conexões ativas simultaneamente (múltiplas abas, dispositivos). A entrega de um evento pode ser endereçada à conexão específica ou à identidade do cliente.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Endereçamento por connectionId | Controle granular por conexão | O chamador precisa conhecer o connectionId; múltiplos envios para múltiplas abas |
| Endereçamento por clientId (escolhida) | O chamador endereça pelo userId; todas as abas recebem automaticamente | Menos controle granular; sempre entrega a todas as conexões |

### Decisão

Eventos são endereçados pela identidade do cliente (`clientId`, ex: `userId`). O servidor é responsável por entregar o evento a todas as conexões ativas daquele cliente. O chamador não precisa conhecer os `connectionId`s individuais.

### Consequências

- API de entrega mais simples para os chamadores
- Comportamento consistente para usuários com múltiplas abas ou dispositivos
- Specs de feature que precisam endereçar uma conexão específica devem estender este comportamento

---

## ADR-003 — Entrega fire-and-forget sem confirmação de recebimento

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto

Ao entregar um evento a um cliente, o servidor pode aguardar uma confirmação de recebimento (acknowledgment) antes de considerar a entrega concluída, ou pode adotar o modelo fire-and-forget sem confirmação.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Entrega com acknowledgment | Garantia de que o cliente processou o evento | Overhead de roundtrip; complexidade no chamador; necessidade de timeout e retry |
| Fire-and-forget (escolhida) | Simples; sem overhead de roundtrip; sem acoplamento com o tempo de processamento do cliente | Sem garantia de que o cliente recebeu ou processou o evento |

### Decisão

O servidor entrega o evento sem aguardar confirmação. A entrega é considerada concluída no momento em que o evento é escrito na conexão. O chamador não recebe retorno sobre o sucesso ou falha da entrega individual.

### Consequências

- API de entrega sem estado; o chamador não precisa gerenciar callbacks ou timeouts
- Specs de feature que precisam de garantia de entrega devem implementar mecanismo próprio (ex: combinação com `messaging/async-queue` e reentrega por polling)

---

## ADR-004 — Descarte silencioso para clientes offline

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto

Quando um evento é emitido para um cliente sem conexão ativa, o sistema precisa decidir entre descartar o evento ou persistir/enfileirar para entrega futura.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Persistência automática | Garantia de entrega eventual | Acoplamento com camada de persistência; aumento de escopo |
| Descarte silencioso (escolhida) | Escopo mínimo; sem dependências adicionais | Eventos podem ser perdidos se o cliente não estiver conectado |

### Decisão

Eventos emitidos para clientes sem conexão ativa são descartados silenciosamente. Nenhum erro é retornado ao chamador. Comportamentos de persistência ou enfileiramento para entrega posterior são definidos pelas specs de feature que os necessitam.

### Consequências

- Escopo desta spec permanece focado no canal em si
- Specs de feature que precisam de garantia de entrega devem combinar este módulo com `messaging/async-queue` ou mecanismo equivalente
