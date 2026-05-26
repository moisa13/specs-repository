# Decisions — NestJS Canal de Comunicação em Tempo Real

> Decisões que podem parecer questionáveis sem contexto — evita que implementadores "melhorem" comportamentos deliberadamente escolhidos.

---

## ADR-001 — Socket.IO em vez de WebSocket nativo

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto

O NestJS suporta dois adapters para WebSocket: o nativo (`@nestjs/platform-ws`) e o Socket.IO (`@nestjs/platform-socket.io`). A escolha afeta a API de servidor e o que os clientes precisam implementar.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| WebSocket nativo (`ws`) | Sem dependência de biblioteca; protocolo puro | Sem fallback; sem reconexão automática; sem broadcast por sala nativo |
| Socket.IO (escolhida) | Reconexão automática no cliente; fallback para long-polling; salas nativas; SDK maduro para web e mobile | Requer `socket.io-client` no front-end; overhead de protocolo maior |

### Decisão

Usar `@nestjs/platform-socket.io` com o adapter Socket.IO. A reconexão automática e o suporte a salas — mesmo que não usados nesta spec — são primitivos que specs de feature poderão adotar sem mudança de transporte.

### Consequências

- O cliente deve usar `socket.io-client` ou SDK compatível — não é possível conectar com um WebSocket nativo puro
- O gateway é configurado com `cors` e opções de transporte no decorator `@WebSocketGateway`

---

## ADR-002 — RealtimeService como único ponto de emissão

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto

O Socket.IO expõe a API de emissão diretamente nos objetos `Socket` e `Server`. Em NestJS, o `RealtimeGateway` tem acesso ao `Server` via `@WebSocketServer()`. Feature modules poderiam injetar o gateway para emitir eventos diretamente.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Feature modules injetam o gateway | Acesso direto à API Socket.IO; sem indireção | Acoplamento de features à implementação do gateway; difícil auditar quem emite o quê |
| `RealtimeService` facade (escolhida) | Ponto único de emissão; isola o Socket.IO; API simples para features | Camada extra de indireção |

### Decisão

`RealtimeService.sendToUser` é a única API de emissão exposta fora de `src/infrastructure/realtime/`. Features injetam `RealtimeService`, nunca o gateway.

### Consequências

- Trocar Socket.IO por outra biblioteca exige mudanças apenas em `RealtimeGateway` e `RealtimeService`
- A assinatura de `sendToUser` é estável; features não precisam conhecer detalhes de socket

---

## ADR-003 — Token JWT no query param do handshake

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto

O Socket.IO transmite o token de autenticação no handshake HTTP. As opções mais comuns são: query param (`?token=...`), header de autorização, ou cookie. Cada uma tem implicações de acessibilidade pela biblioteca cliente.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Header `Authorization` | Padrão HTTP; não exposto na URL | A maioria dos clientes Socket.IO não suporta headers customizados no handshake WebSocket nativo |
| Cookie | Transparente para o cliente; funciona com `withCredentials` | Requer CORS configurado com `credentials: true`; problemas com SameSite em mobile |
| Query param `?token=...` (escolhida) | Suportado por todos os clientes Socket.IO; simples de implementar | Token visível nos logs de acesso do servidor |

### Decisão

O token é enviado no query param `token` do handshake. O `WsJwtGuard` lê `socket.handshake.query.token`. Logs de acesso devem ser configurados para não registrar query strings em produção.

### Consequências

- Clientes passam o token como `io('url', { query: { token: jwt } })`
- O token aparece nos logs de acesso do servidor — mitigar via configuração de log no proxy

---

## ADR-004 — Map<userId, Set<Socket>> para múltiplas conexões

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto

Um usuário pode ter múltiplas conexões ativas (múltiplas abas, dispositivos). O `RealtimeService` precisa de uma estrutura que permita emitir para todas as conexões de um usuário pelo `userId`.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| `Map<socketId, Socket>` | Simples; acesso por socket | Não agrupa por usuário; `sendToUser` exigiria varredura de todo o mapa |
| `Map<userId, Socket>` (uma conexão) | Simples | Sobrescreve a conexão anterior; penaliza usuários com múltiplas abas |
| `Map<userId, Set<Socket>>` (escolhida) | Agrupa por usuário; suporta múltiplas conexões; `sendToUser` itera apenas o Set do usuário | Leve overhead de estrutura |

### Decisão

Usar `Map<string, Set<Socket>>` indexado pelo `userId`. O `RealtimeService` mantém essa estrutura e expõe `addConnection`, `removeConnection` e `sendToUser`.

### Consequências

- Emitir para um usuário é O(n) onde n é o número de conexões daquele usuário — geralmente 1–3
- Não é necessário Map global de socketId → userId; o `userId` fica em `socket.data.userId`

---

## ADR-005 — RealtimeModule importado no AppModule

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto

O gateway WebSocket precisa estar ativo desde o boot da aplicação, independente de quais features estão carregadas. Sem importação no `AppModule`, o gateway só subiria quando o primeiro feature module que importa `RealtimeModule` fosse carregado.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Cada feature importa `RealtimeModule` | Fiel ao isolamento de módulos | Gateway não sobe se nenhuma feature estiver carregada; ordem de inicialização imprevisível |
| `AppModule` importa `RealtimeModule` (escolhida) | Gateway sempre ativo desde o boot | Feature modules ainda precisam importar `RealtimeModule` para injetar `RealtimeService` |

### Decisão

O `AppModule` importa `RealtimeModule`. Feature modules que precisam de `RealtimeService` também importam `RealtimeModule` — a importação no `AppModule` não os dispensa disso.

### Consequências

- O gateway WebSocket sobe em todos os ambientes, mesmo em testes de integração — use mocks de `RealtimeService` quando necessário
- Feature modules importam `RealtimeModule` para ter `RealtimeService` injetável, sem preocupação com inicialização do gateway
