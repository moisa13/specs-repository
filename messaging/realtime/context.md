# Context — Canal de Comunicação em Tempo Real

---

## Plataformas suportadas

| Plataforma | Suportada | Observações |
|------------|-----------|-------------|
| API REST | ❌ | Comunicação em tempo real requer conexão persistente |
| Web SPA | ✅ | Cliente primário; mantém conexão ativa durante a sessão |
| Mobile (iOS/Android) | ✅ | Suportado; gestão de reconexão é responsabilidade do cliente |
| Backend (serviço interno) | ✅ | Servidor é sempre o produtor ou consumidor de eventos |

## Pré-requisitos

- [ ] Mecanismo de autenticação disponível — o servidor deve ser capaz de validar a identidade do cliente a partir de um token apresentado no handshake
- [ ] Protocolo de transporte que suporte conexão persistente (WebSocket, SSE ou equivalente) disponível na infraestrutura

## Escopo

**Dentro do escopo:**
- Estabelecimento e autenticação da conexão (handshake)
- Ciclo de vida da conexão: abertura, manutenção e encerramento
- Entrega de eventos do servidor para o cliente
- Entrega de eventos do cliente para o servidor
- Comportamento em falhas de conexão e autenticação

**Fora do escopo:**
- Qual evento enviar, quando e para quem — responsabilidade da spec de feature
- Salas, canais, grupos ou broadcast para múltiplos clientes — responsabilidade da spec de feature
- Persistência ou enfileiramento de eventos para clientes offline — responsabilidade da spec de feature
- Configuração do protocolo de transporte e infraestrutura — ver spec de implementação (ex: `setup/nestjs/nestjs-realtime`)
- Mecanismo de autenticação em si — ver spec de autenticação do projeto

## Variações por plataforma

### Web SPA

O cliente deve apresentar o token de autenticação no momento da abertura da conexão. A forma exata de transporte do token (query param, header, cookie) é definida pela spec de implementação.

### Mobile (iOS/Android)

O comportamento é equivalente ao Web SPA. A gestão de reconexão após perda de rede (background, sono do dispositivo) é responsabilidade do cliente nativo — esta spec não define polling ou heartbeat.

## Considerações de ambiente

- **Desenvolvimento:** conexões sem TLS são aceitáveis; o token pode ser simplificado para fins de teste
- **Produção:** conexões devem ser protegidas por TLS; o token deve ser validado com as mesmas regras do restante da autenticação
