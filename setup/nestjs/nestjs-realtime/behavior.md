# Behavior — NestJS Canal de Comunicação em Tempo Real

---

## Objetivo

Configurar o Socket.IO em um projeto NestJS de forma que todas as conexões sejam autenticadas no handshake e os eventos sejam emitidos exclusivamente via `RealtimeService`, sem acesso direto ao gateway pelos módulos de feature.

---

## Estado inicial (bootstrap)

Ao aplicar esta spec pela primeira vez, o `RealtimeModule` é importado no `AppModule`. O gateway sobe junto com a aplicação, na mesma porta HTTP, sem nenhuma lógica de feature registrada. Clients podem conectar e autenticar, mas nenhum evento de domínio é emitido até que um módulo de feature injete o `RealtimeService` e o utilize.

---

## Fluxo de configuração (setup)

1. O `RealtimeModule` declara o `RealtimeGateway` e o `RealtimeService` e os exporta
2. O `AppModule` importa o `RealtimeModule` para que o gateway suba junto com a aplicação
3. Módulos de feature que precisam emitir eventos importam o `RealtimeModule` e injetam o `RealtimeService`

---

## Fluxo de conexão

1. O cliente abre uma conexão WebSocket enviando o JWT no query param `token`
2. O `WsJwtGuard` é acionado no `handleConnection` do `RealtimeGateway`
3. O guard extrai o token de `socket.handshake.query.token`
4. O guard valida o token usando `JwtService` com o `JWT_SECRET` configurado no `ConfigService`
5. Se o token for inválido ou ausente, o guard desconecta o socket com o código de erro `UNAUTHORIZED` e encerra
6. Se o token for válido, o `userId` extraído do payload do JWT é associado ao socket
7. O socket é adicionado ao conjunto de conexões ativas do cliente em `RealtimeService`

---

## Fluxo de desconexão

1. O cliente encerra a conexão ou o socket é perdido por perda de rede
2. O `handleDisconnect` do `RealtimeGateway` é acionado
3. O socket é removido do conjunto de conexões ativas do cliente em `RealtimeService`
4. Se o conjunto do cliente ficar vazio, a entrada do cliente é removida do mapa

---

## Fluxo de emissão — servidor para cliente

1. Um service de feature chama `RealtimeService.sendToUser(userId, event, payload)`
2. O `RealtimeService` consulta o mapa interno pelo `userId`
3. Se houver conexões ativas, o evento é emitido via `socket.emit(event, payload)` para cada socket do cliente
4. Se não houver conexões ativas, o método retorna sem erro — silent drop

---

## Fluxo de recebimento — cliente para servidor

1. O cliente envia um evento com nome e payload
2. O `RealtimeGateway` define handlers via `@SubscribeMessage('nome-do-evento')` para eventos esperados
3. O método handler é invocado pelo Socket.IO com o socket e o payload
4. Eventos sem handler registrado são ignorados silenciosamente pelo Socket.IO

---

## Regras de negócio

- O `RealtimeModule` é importado no `AppModule` — o gateway deve subir junto com a aplicação, não sob demanda de uma feature
- O `RealtimeService` é o único provider que acessa os sockets diretamente — módulos de feature nunca injetam o `RealtimeGateway`
- O mapa de conexões é `Map<string, Set<Socket>>`, indexado pelo `userId`
- Um cliente pode ter múltiplos sockets simultâneos; todos ficam no mesmo `Set`
- O `userId` é extraído do payload do JWT no momento da conexão e armazenado no objeto `socket.data` para uso no `handleDisconnect`
- Handlers de eventos cliente→servidor são definidos no `RealtimeGateway` — nunca nos módulos de feature
- A porta do WebSocket é a mesma da API HTTP — o Socket.IO é montado no mesmo servidor Express

---

## Edge cases

| Situação | Comportamento esperado |
|----------|----------------------|
| Token ausente no query param `token` | Guard desconecta o socket com `UNAUTHORIZED`; nenhuma entrada é criada no mapa |
| Token inválido ou expirado | Guard desconecta o socket com `UNAUTHORIZED`; nenhuma entrada é criada no mapa |
| Cliente reconecta após queda de rede | Novo socket é criado e adicionado ao mapa; o socket anterior já foi removido pelo `handleDisconnect` |
| `sendToUser` chamado para `userId` sem conexão | Retorna sem erro; nenhum evento é emitido |
| Cliente com múltiplos sockets ativos | `sendToUser` itera sobre todos os sockets do `Set` e emite para cada um |
| Socket removido do mapa antes do `handleDisconnect` | Não ocorre — o mapa é gerenciado exclusivamente pelo gateway; `handleDisconnect` é sempre o ponto de remoção |
| Feature module importa `RealtimeModule` sem precisar receber eventos | Apenas o `RealtimeService` fica disponível; nenhum handler novo é registrado no gateway |

---

## Restrições

- Nunca injetar `RealtimeGateway` fora do `RealtimeModule`
- Nunca acessar `socket` diretamente fora do `RealtimeGateway` e do `RealtimeService`
- Nunca registrar `@SubscribeMessage` em classes fora do `RealtimeGateway`
- Nunca criar uma segunda instância de gateway para o mesmo namespace
- Nunca armazenar o socket fora do mapa gerenciado pelo `RealtimeService`

---

## O que esta spec NÃO cobre

- O comportamento do canal (regras de entrega, edge cases) → definido em `messaging/realtime`
- Qual evento emitir e quando → responsabilidade da spec de feature
- Salas, namespaces adicionais, broadcast para grupos → responsabilidade da spec de feature
- Geração e renovação de JWT → ver spec de autenticação do projeto
