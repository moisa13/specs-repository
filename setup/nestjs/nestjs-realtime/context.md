# Context — NestJS Canal de Comunicação em Tempo Real

---

## Plataformas suportadas

| Plataforma | Suportada | Observações |
|------------|-----------|-------------|
| API REST | ❌ | Esta spec configura o canal WebSocket, não endpoints HTTP |
| Web SPA | ❌ | Cliente; conecta ao gateway definido por esta spec |
| Mobile (iOS/Android) | ❌ | Cliente; conecta ao gateway definido por esta spec |
| Backend (serviço interno) | ✅ | Servidor NestJS que expõe o gateway WebSocket |

## Pré-requisitos

- [ ] `setup/nestjs/nestjs-init` aplicado — `ConfigService` e estrutura global disponíveis
- [ ] Variável de ambiente `JWT_SECRET` configurada — usada pelo `WsJwtGuard` para validar tokens
- [ ] Pacotes instalados: `@nestjs/websockets`, `@nestjs/platform-socket.io`, `socket.io`, `@nestjs/jwt`

## Escopo

**Dentro do escopo:**
- Configuração do `RealtimeModule` com gateway, service e guard
- `RealtimeGateway` com autenticação JWT no handshake via `WsJwtGuard`
- `RealtimeService` como único ponto de emissão de eventos para clientes
- Mapa de conexões ativas por identidade de cliente
- Variáveis de ambiente necessárias
- Estrutura de diretórios e convenções de nomenclatura

**Fora do escopo:**
- O comportamento do canal (regras, fluxos, edge cases) → definido em `messaging/realtime`
- O que emitir, quando e para quem → responsabilidade da spec de feature
- Salas, canais e broadcast para grupos → responsabilidade da spec de feature
- Configuração do mecanismo de autenticação (geração de JWT) → ver spec de autenticação do projeto

## Considerações de ambiente

- **Desenvolvimento:** `JWT_SECRET` pode ser qualquer string; conexões sem TLS são aceitáveis
- **Produção:** `JWT_SECRET` deve ser um segredo forte; o gateway deve operar por trás de proxy com TLS (nginx, load balancer)
