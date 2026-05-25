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
| [`setup/nestjs-init`](./setup/nestjs-init/README.md) | Estrutura base e configuração global de um projeto NestJS para APIs | 0.2.0 | ✅ pronto |
| [`setup/nestjs-database-init`](./setup/nestjs-database-init/README.md) | Conexão com PostgreSQL via TypeORM e configuração de migrations em projetos NestJS | 0.1.1 | ✅ pronto |
| [`observability/nestjs-health`](./observability/nestjs-health/README.md) | Endpoints de liveness e readiness via `@nestjs/terminus` com verificação de banco e memória | 0.2.0 | ✅ pronto |

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
