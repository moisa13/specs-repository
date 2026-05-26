# Integration — Processamento Assíncrono com Fila

---

## Dependências

Esta spec é agnóstica de infraestrutura. Não depende de nenhum outro módulo do repositório.

| Módulo | Obrigatório | O que usa |
|--------|-------------|-----------|
| — | — | — |

---

## Dependentes

| Módulo | Como usa este módulo |
|--------|----------------------|
| `setup/nestjs/nestjs-queue` | Implementa esta spec usando BullMQ + Redis em projetos NestJS |

Specs de domínio que precisam de processamento assíncrono devem referenciar esta spec para definir o comportamento esperado dos seus jobs e referenciar `setup/nestjs/nestjs-queue` para a implementação.

---

## Eventos emitidos

Esta spec não define eventos de domínio. Os jobs em si são a unidade de trabalho; não há eventos publicados para outros módulos.

---

## Eventos consumidos

Esta spec não consome eventos externos. A produção de jobs é iniciada pelo chamador diretamente.

---

## Impacto em outros sistemas

- O broker de filas (ex: Redis) deve estar disponível para que a produção e o consumo funcionem
- Falhas no broker afetam apenas o enfileiramento; o chamador é responsável por tratar o erro de produção
