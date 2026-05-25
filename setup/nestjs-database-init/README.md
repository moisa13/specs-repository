# NestJS — Configuração de Banco de Dados (TypeORM + PostgreSQL)

> Configura a conexão com PostgreSQL via TypeORM e disponibiliza o `DatabaseModule` global com suporte a migrations.

---

**Spec version:** 0.2.0
**Status:** ✅ pronto
**Última revisão:** 2026-05
**Revisado por:** Moiséis Almeida

---

## O que é

Descreve como configurar o TypeORM com PostgreSQL em um projeto NestJS já inicializado, expondo um `DatabaseModule` global que disponibiliza a conexão para todos os módulos de feature. Cobre variáveis de ambiente, configuração do `TypeOrmModule.forRootAsync`, estrutura de diretórios para migrations, arquivo `data-source.ts` para o CLI e comportamento de execução automática de migrations em desenvolvimento.

## Quando usar

- Ao adicionar banco de dados a um projeto NestJS já inicializado com `setup/nestjs-init`
- Ao auditar se a configuração de banco de um projeto existente segue o padrão esperado

## Quando NÃO usar

- Para criar entidades ou repositórios — responsabilidade da spec de cada feature
- Para configurar health check de banco — ver `observability/nestjs-health`
- Antes de ter `setup/nestjs-init` aplicado — o `ConfigService` deve estar disponível

## Módulos relacionados

- `setup/nestjs-init` — pré-requisito; disponibiliza `ConfigService` e a estrutura global
- `observability/nestjs-health` — usa a conexão configurada aqui para expor o `TypeOrmHealthIndicator`

## Arquivos desta spec

| Arquivo | Conteúdo |
|---------|----------|
| [`context.md`](./context.md) | Plataformas, pré-requisitos, escopo |
| [`behavior.md`](./behavior.md) | Regras, fluxos, edge cases |
| [`contracts.md`](./contracts.md) | Variáveis de ambiente, estrutura de diretórios, configuração do TypeORM |
| [`integration.md`](./integration.md) | Dependências e integrações |
| [`acceptance.md`](./acceptance.md) | Critérios de aceitação |
| [`decisions.md`](./decisions.md) | Decisões arquiteturais (ADRs) |
| [`CHANGELOG.md`](./CHANGELOG.md) | Histórico de versões |
| [`examples/`](./examples/) | Exemplos concretos |
