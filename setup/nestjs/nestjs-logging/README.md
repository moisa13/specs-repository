# Setup NestJS — Logging Estruturado

> Implementação de logging estruturado com rastreamento por requisição em projetos NestJS, usando Winston.

---

**Spec version:** 0.1.1
**Status:** ✅ pronto
**Última revisão:** 2026-05
**Revisado por:** Moiséis Almeida

---

## O que é

Esta spec descreve como implementar o comportamento definido em `observability/structured-log` em um projeto NestJS. Cobre os componentes necessários (`LoggerService`, `LoggingInterceptor`, `LogSanitizer`), sua configuração com Winston e a integração com o ciclo de vida do framework.

## Quando usar

- Ao adicionar logging estruturado a um projeto NestJS
- Quando o projeto já segue `setup/nestjs/nestjs-init` como estrutura base

## Quando NÃO usar

- Em projetos com framework diferente de NestJS → usar `observability/structured-log` como guia e adaptar à stack escolhida
- Quando apenas logging básico sem sanitização e interceptor de requisições for necessário → sobrecarga desnecessária

## Módulos relacionados

- `observability/structured-log` — spec de comportamento que esta spec implementa
- `setup/nestjs/nestjs-init` — estrutura base obrigatória

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
