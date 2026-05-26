# Integration — NestJS Filas Assíncronas (BullMQ + Redis)

---

## Dependências

| Módulo | Obrigatório | O que usa |
|--------|-------------|-----------|
| `setup/nestjs/nestjs-init` | ✅ | `ConfigService` para leitura das variáveis de ambiente Redis |
| `messaging/async-queue` | ✅ | Define o comportamento de produção, consumo, retry e DLQ que esta spec implementa |

---

## Dependentes

| Módulo | Como usa este módulo |
|--------|----------------------|
| Qualquer spec de feature que referencie `messaging/async-queue` | Importa `QueueModule` e registra um Processor para processar seus jobs |

---

## Eventos emitidos

Esta spec não define eventos de domínio. A comunicação entre produtor e consumidor ocorre via fila, não via eventos.

---

## Eventos consumidos

Esta spec não consome eventos externos.

---

## Impacto em outros sistemas

- A instância Redis é um ponto de dependência externo; sua indisponibilidade afeta toda a produção e consumo de jobs da aplicação
- O `QueueModule` não é registrado como módulo global do NestJS — cada módulo de feature que declara um Processor ou produz jobs deve importá-lo explicitamente
- **Exceção:** quando o Bull Board está habilitado, o `QueueModule` deve ser importado no `AppModule` para que a rota `/queues` esteja disponível globalmente, independente de quais feature modules estão carregados — ver ADR-006 em `decisions.md`
