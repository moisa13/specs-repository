# Contracts — [Nome do Módulo]

> Contratos de dados agnósticos de linguagem. O agente deve mapear estes tipos para os equivalentes da stack utilizada.

---

## Entidades

### [NomeDaEntidade]

| Campo | Tipo | Obrigatório | Restrições | Descrição |
|-------|------|-------------|------------|-----------|
| `campo` | string | ✅ | max: 255 | [descrição] |
| `campo` | datetime | ✅ | UTC | [descrição] |
| `campo` | string | ❌ | nullable | [descrição] |

---

## Entrada

| Campo | Tipo | Obrigatório | Validação |
|-------|------|-------------|-----------|
| `campo` | string | ✅ | [regra] |

---

## Saída — Sucesso

| Campo | Tipo | Sempre presente | Descrição |
|-------|------|-----------------|-----------|
| `campo` | string | ✅ | [descrição] |

---

## Respostas de erro

> Descreva as respostas de erro no formato adequado à plataforma definida em `context.md`.
> Exemplos por plataforma:
>
> **API REST:** use códigos HTTP (400, 401, 422, 429, etc.)
> **Mobile / Backend interno:** use códigos de erro semânticos (ex: `INVALID_INPUT`, `RULE_VIOLATION`)
> **Misto:** defina um código semântico como primário e mapeie para HTTP quando aplicável

| Código de erro | Situação | Campos adicionais |
|----------------|----------|-------------------|
| [ex: `INVALID_INPUT` / 400] | Entrada inválida | `errors[]` com campos e mensagens |
| [ex: `RULE_VIOLATION` / 422] | Regra de negócio violada | `reason` com descrição |

---

## Adições ao agents.md

> Inclua esta seção quando a spec introduzir pelo menos uma das seguintes convenções duráveis:
> - estrutura de diretórios ou nomenclatura de arquivos que o agente precisa respeitar ao criar novos arquivos
> - regras de runtime que o agente não conseguiria derivar do código existente (ex: ordem de registro de providers, globs de paths)
> - restrições de uso de uma biblioteca ou padrão adotado (ex: sempre usar `getOrThrow`, nunca `synchronize: true` em produção)
>
> Omita esta seção se a spec apenas descreve comportamento de produto (regras de negócio, fluxos de usuário) sem gerar artefatos ou convenções técnicas que o agente precisará replicar em implementações futuras.
>
> **Nota:** `setup/nestjs/nestjs-init` é a única spec que **cria** o `agents.md`. Todas as demais apenas acrescentam seções a ele com `Ao aplicar esta spec, acrescentar a seção abaixo ao agents.md do projeto:`.

Ao aplicar esta spec, acrescentar a seção abaixo ao `agents.md` do projeto:

```markdown
## [Nome do módulo] ([caminho/da/spec])

- [convenção 1]
- [convenção 2]
```
