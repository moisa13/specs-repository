# Integration — NestJS Canal de Comunicação em Tempo Real

---

## Dependências

| Módulo | Obrigatório | O que usa |
|--------|-------------|-----------|
| `setup/nestjs/nestjs-init` | ✅ | `ConfigService` para acesso ao `JWT_SECRET`; estrutura global do projeto |
| `messaging/realtime` | ✅ | Spec de comportamento que esta implementação satisfaz |

---

## Dependentes

| Módulo | Como usa este módulo |
|--------|----------------------|
| `setup/nestjs/nestjs-queue` | Independente; ambos podem coexistir no mesmo projeto sem interferência |
| Módulos de feature | Importam `RealtimeModule` e injetam `RealtimeService` para emitir eventos a clientes conectados |

---

## Eventos emitidos

Este módulo não define eventos de domínio próprios. Os eventos emitidos via `RealtimeService.sendToUser` são determinados pelos módulos de feature que o utilizam.

---

## Eventos consumidos

Este módulo não consome eventos internos. O gateway recebe eventos WebSocket diretamente dos clientes conectados via Socket.IO.

---

## Impacto em outros sistemas

- O Socket.IO é montado no mesmo servidor HTTP da API REST — não há porta adicional
- Cada conexão WebSocket ativa mantém um socket aberto no servidor; o dimensionamento deve considerar o número esperado de conexões simultâneas
- O gateway não tem dependência de Redis ou banco de dados — o mapa de conexões é in-memory; ao reiniciar a aplicação, todos os clientes precisam reconectar
