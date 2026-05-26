# Integration — Caching — Cache de Dados

---

## Dependências

| Módulo | Obrigatório | O que usa |
|--------|-------------|-----------|
| Redis | ✅ | Backend de armazenamento; banco dedicado separado de filas e sessões |

---

## Dependentes

| Módulo | Como usa este módulo |
|--------|----------------------|
| `setup/nestjs/nestjs-cache` | Implementa os padrões e contratos definidos nesta spec |
| Qualquer módulo de domínio | Implementa os padrões `remember` e `deletePattern` desta spec para cachear seus recursos |

---

## Eventos emitidos

| Canal | Quando | Payload |
|-------|--------|---------|
| `cache-events:{recurso}` | Após invalidação de chaves de um recurso | `{ resource, keys, timestamp }` — ver `contracts.md` |

---

## Eventos consumidos

Esta spec não consome eventos de outros módulos.

---

## Impacto em outros sistemas

- Operações de `deletePattern` em alto volume impactam o Redis de forma incremental — não bloqueante, mas gera tráfego de rede
- O aquecimento de cache no boot aumenta o tempo de inicialização da aplicação proporcionalmente ao volume de dados pré-carregados
- Eventos publicados em `cache-events:{recurso}` podem ser consumidos por gateways WebSocket para notificação em tempo real — não é responsabilidade desta spec
