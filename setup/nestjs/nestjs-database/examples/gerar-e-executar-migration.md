# Exemplo — Gerar e executar migration

## Contexto

Fluxo completo de criação e execução de uma migration após adicionar ou modificar uma entidade. Inclui também o fluxo para criação manual de migrations vazias via `migration:create`.

---

## Passo a passo

### 1. Gerar a migration (após criar ou modificar uma entidade)

```bash
pnpm migration:generate src/database/migrations/CreateUsersTable
```

O TypeORM compara o estado atual das entidades com o schema do banco e gera um arquivo com os comandos SQL necessários.

Arquivo gerado: `src/database/migrations/1716234567890-CreateUsersTable.ts`

### 2. Revisar o arquivo gerado

Sempre revisar o SQL gerado antes de aplicar. O arquivo contém os métodos `up()` (aplicar) e `down()` (reverter).

### 3. Em desenvolvimento

A migration é aplicada automaticamente no próximo `pnpm start:dev`. O passo de execução manual não é necessário.

### 4. Em produção — executar manualmente antes do deploy

```bash
pnpm migration:run
```

### 5. Reverter a última migration se necessário

```bash
pnpm migration:revert
```

---

## Fluxo alternativo — Criar migration vazia manualmente

Use `migration:create` quando a operação não pode ser inferida pelo CLI a partir das entidades (ex: transformação de dados, seeding pontual).

```bash
pnpm migration:create src/database/migrations/SeedInitialRoles
```

O TypeORM cria um arquivo vazio com os métodos `up()` e `down()` para preenchimento manual.

Arquivo gerado: `src/database/migrations/1716234567890-SeedInitialRoles.ts`

A escrita manual deve ser registrada em `decisions.md` conforme descrito em `behavior.md`.

---

## Notas

- O nome passado ao `migration:generate` ou `migration:create` deve descrever a operação em `PascalCase`: `CreateUsersTable`, `AddEmailIndexToUsers`, `RenameColumnNameToFullName`
- Nunca deletar ou modificar uma migration já aplicada em produção
- `migration:revert` desfaz apenas a última migration — para reverter várias, execute o comando múltiplas vezes
