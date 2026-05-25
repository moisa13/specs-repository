# NestJS — Inicialização de Projeto

> Define a estrutura base e a configuração global obrigatória para um projeto NestJS destinado a APIs REST.

---

**Spec version:** 0.1.0
**Status:** ✅ pronto
**Última revisão:** 2026-05
**Revisado por:** Moiséis Almeida

---

## O que é

Descreve o conjunto mínimo de instalações, estrutura de diretórios e configurações globais que todo projeto NestJS para APIs deve ter antes de qualquer feature ser implementada. Cobre instalação via pnpm, organização de pastas, `ConfigModule` com validação de variáveis de ambiente via Joi, `ValidationPipe` global, filtro de exceção global e Swagger habilitado apenas em desenvolvimento.

## Quando usar

- Ao iniciar um novo projeto NestJS do zero para uso como API REST
- Ao auditar se um projeto existente segue a estrutura base esperada pelo boilerplate

## Quando NÃO usar

- Para adicionar features a um projeto já inicializado — use a spec da feature em questão
- Para configurar banco de dados — ver `setup/database` (a criar)
- Para configurar autenticação — ver `auth/` (a criar)

## Módulos relacionados

- `setup/database` — configuração de banco de dados (a criar)
- `auth/` — autenticação e autorização (a criar)

## Arquivos desta spec

| Arquivo | Conteúdo |
|---------|----------|
| [`context.md`](./context.md) | Plataformas, pré-requisitos, escopo |
| [`behavior.md`](./behavior.md) | Regras, fluxos, edge cases |
| [`contracts.md`](./contracts.md) | Estrutura de diretórios, variáveis de ambiente, formato de erro |
| [`integration.md`](./integration.md) | Dependências e integrações |
| [`acceptance.md`](./acceptance.md) | Critérios de aceitação |
| [`decisions.md`](./decisions.md) | Decisões arquiteturais (ADRs) |
| [`CHANGELOG.md`](./CHANGELOG.md) | Histórico de versões |
| [`examples/`](./examples/) | Exemplos concretos |
