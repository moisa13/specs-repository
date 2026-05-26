# Caching — Cache de Dados

> Comportamento de armazenamento e recuperação de dados em cache com invalidação controlada e aquecimento no boot.

---

**Spec version:** 0.2.0
**Status:** ✅ pronto
**Última revisão:** 2026-05
**Revisado por:** Moiséis Almeida

---

## O que é

Esta spec define o comportamento de cache de dados em aplicações de servidor: como armazenar resultados computados ou consultados, como recuperá-los com garantia de consistência, como invalidá-los de forma controlada e como pré-aquecê-los no boot para evitar cold start.

## Quando usar

- Ao cachear respostas de queries custosas ou lentas
- Ao precisar de consistência entre instâncias concorrentes (sem cache duplicado ou corrida)
- Ao precisar de invalidação propagada para outros consumidores do cache
- Ao precisar de aquecimento de cache no boot da aplicação

## Quando NÃO usar

- Para sincronização de estado entre serviços — use mensageria
- Para dados que mudam a cada requisição e não têm valor em serem reutilizados
- Para dados que exigem consistência forte em tempo real — use o banco de dados diretamente

## Módulos relacionados

- `setup/nestjs/nestjs-cache` — implementação NestJS desta spec
- `messaging/async-queue` — para invalidação assíncrona baseada em eventos de domínio

## Arquivos desta spec

| Arquivo | Conteúdo |
|---------|----------|
| [`context.md`](./context.md) | Plataforma, pré-requisitos, tecnologias |
| [`behavior.md`](./behavior.md) | Fluxos, padrões e regras de comportamento |
| [`contracts.md`](./contracts.md) | Contratos de entrada, saída e configuração |
| [`integration.md`](./integration.md) | Dependências e pontos de integração |
| [`acceptance.md`](./acceptance.md) | Critérios de aceitação |
| [`decisions.md`](./decisions.md) | Decisões arquiteturais (ADRs) |
| [`CHANGELOG.md`](./CHANGELOG.md) | Histórico de versões |
| [`examples/`](./examples/) | Exemplos de uso dos padrões |
