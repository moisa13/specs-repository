# Specs Repository — Index

> Catálogo de especificações de comportamento reutilizáveis.

---

## Arquivos raiz

| Arquivo | Para quem |
|---------|-----------|
| [`README.md`](./README.md) | Todos — visão geral e convenções do repositório |
| [`agents.md`](./agents.md) | Code Agents — leia antes de qualquer implementação |
| [`contributing.md`](./contributing.md) | Colaboradores — fluxo Git Flow para criar e editar specs |

---

## Como usar este repositório

Consulte [`_meta/how-to-use.md`](./_meta/how-to-use.md) para o guia completo.
Consulte [`_meta/prompt-recipes.md`](./_meta/prompt-recipes.md) para templates de prompt prontos.

---

## Módulos disponíveis

| Módulo | Descrição | Versão | Status |
|--------|-----------|--------|--------|
| [`setup/nestjs/nestjs-init`](./setup/nestjs/nestjs-init/README.md) | Estrutura base e configuração global de um projeto NestJS para APIs | 0.3.0 | ✅ pronto |
| [`setup/nestjs/nestjs-database`](./setup/nestjs/nestjs-database/README.md) | Conexão com PostgreSQL via TypeORM e configuração de migrations em projetos NestJS | 0.2.0 | ✅ pronto |
| [`setup/nestjs/nestjs-health`](./setup/nestjs/nestjs-health/README.md) | Endpoints de liveness e readiness via `@nestjs/terminus` com verificação de banco e memória | 0.3.0 | ✅ pronto |
| [`messaging/async-queue`](./messaging/async-queue/README.md) | Comportamento de produção e consumo de jobs em filas assíncronas, com retry e dead letter queue | 0.1.0 | ✅ pronto |
| [`setup/nestjs/nestjs-queue`](./setup/nestjs/nestjs-queue/README.md) | Implementação de filas assíncronas com BullMQ e Redis em projetos NestJS, com módulo centralizado | 0.1.0 | ✅ pronto |

---

## Legenda de status

| Status | Significado |
|--------|-------------|
| 🚧 template | Estrutura criada, conteúdo a preencher |
| 📝 rascunho | Em elaboração, não usar em produção |
| ✅ pronto | Revisado e aprovado para uso |
| ⚠️ desatualizado | Implementações existentes podem estar desalinhadas |
| 🗄️ depreciado | Substituído por outro módulo, não usar em novos projetos |

---

## Adicionando novos módulos

1. Copie `_template/` para o domínio adequado: `cp -r _template/ [domínio]/[módulo]/`
2. Adicione o módulo neste index.md com status `🚧 template`
3. Preencha todos os arquivos seguindo [`_meta/how-to-write.md`](./_meta/how-to-write.md)
4. Atualize o status conforme o progresso
