# Setup NestJS — Cache

> Implementação de cache de dados em projetos NestJS usando ioredis, com suporte a cache-aside, invalidação por padrão e aquecimento no boot.

---

**Spec version:** 0.1.0
**Status:** 📝 rascunho
**Última revisão:** 2026-05
**Revisado por:** —

---

## O que é

Esta spec descreve como implementar o comportamento definido em `caching/cache` em um projeto NestJS. Cobre os componentes necessários (`CacheService`, `GenericCacheService`, `CacheWarmingService`, `CacheEventsService`), sua configuração com ioredis e a integração com o ciclo de vida do framework.

## Quando usar

- Ao adicionar cache de dados a um projeto NestJS
- Quando o projeto já segue `setup/nestjs/nestjs-init` como estrutura base

## Quando NÃO usar

- Em projetos com framework diferente de NestJS → usar `caching/cache` como guia e adaptar à stack
- Quando cache simples in-memory for suficiente → sobrecarga desnecessária

## Módulos relacionados

- `caching/cache` — spec de comportamento que esta spec implementa
- `setup/nestjs/nestjs-init` — estrutura base obrigatória
- `setup/nestjs/nestjs-queue` — usa Redis separado para filas; não compartilhar o banco de cache

## Arquivos desta spec

| Arquivo | Conteúdo |
|---------|----------|
| [`context.md`](./context.md) | Plataforma, pré-requisitos, bibliotecas |
| [`behavior.md`](./behavior.md) | Componentes e fluxo de implementação |
| [`contracts.md`](./contracts.md) | Variáveis de ambiente, convenções, schema |
| [`integration.md`](./integration.md) | Dependências e pontos de integração |
| [`acceptance.md`](./acceptance.md) | Critérios de aceitação |
| [`decisions.md`](./decisions.md) | Decisões arquiteturais (ADRs) |
| [`CHANGELOG.md`](./CHANGELOG.md) | Histórico de versões |
| [`examples/`](./examples/) | Exemplos de código dos componentes |
