# Integration — Canal de Comunicação em Tempo Real

---

## Dependências

| Módulo | Obrigatório | O que usa |
|--------|-------------|-----------|
| Autenticação do projeto | ✅ | Validação do token apresentado no handshake; o mecanismo concreto é definido pelo projeto |

---

## Dependentes

| Módulo | Como usa este módulo |
|--------|----------------------|
| `setup/nestjs/nestjs-realtime` | Implementa o comportamento desta spec em NestJS com Socket.IO |
| Specs de feature | Usam o canal para entregar eventos a clientes conectados |

---

## Eventos emitidos

Este módulo não define eventos de domínio próprios. Os eventos que trafegam pelo canal são definidos pelas specs de feature que o utilizam.

---

## Eventos consumidos

Este módulo não consome eventos de outros módulos. O canal é ativado por chamada direta (ex: `enviarEvento(clientId, event, payload)`), não por subscrição a eventos.

---

## Impacto em outros sistemas

- Requer que a infraestrutura suporte conexões persistentes (WebSocket ou equivalente)
- Cada conexão ativa mantém um recurso alocado no servidor — dimensionamento deve considerar o número esperado de conexões simultâneas
