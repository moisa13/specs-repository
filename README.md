# Specs Repository

> Catálogo de especificações de comportamento reutilizáveis — agnósticas de stack, prontas para uso com Code Agents.

---

## Por onde começar

| Eu sou... | Comece por... |
|-----------|---------------|
| **Code Agent** | [`agents.md`](./agents.md) — leia antes de qualquer ação |
| **Novo no repositório** | [`index.md`](./index.md) → [`_meta/how-to-use.md`](./_meta/how-to-use.md) |
| **Usando specs em um prompt** | [`_meta/how-to-use.md`](./_meta/how-to-use.md) → [`_meta/prompt-recipes.md`](./_meta/prompt-recipes.md) |
| **Criando ou editando uma spec** | [`_meta/how-to-write.md`](./_meta/how-to-write.md) → [`contributing.md`](./contributing.md) |

---

## Estrutura

```
specs-repo/
├── README.md             ← visão geral e convenções do repositório
├── agents.md             ← instruções para Code Agents (inclua sempre no prompt)
├── contributing.md       ← fluxo Git Flow para criar e editar specs
├── index.md              ← catálogo de todos os módulos
│
├── _meta/                ← como usar e contribuir
│   ├── how-to-use.md
│   ├── how-to-write.md
│   ├── prompt-recipes.md
│   └── glossary.md
│
└── _template/            ← template base para novos módulos
    ├── README.md
    ├── context.md
    ├── behavior.md
    ├── contracts.md
    ├── integration.md
    ├── acceptance.md
    ├── decisions.md
    ├── CHANGELOG.md
    └── examples/
        └── template.md       ← renomeie para [ação]-[resultado].md
```

---

## Convenções

| O quê | Convenção |
|-------|-----------|
| Linguagem do conteúdo | Português |
| Linguagem de código, campos e tipos | Inglês |
| Formato | Markdown |
| Estrutura de módulo | Usa `_template/` como base; placeholders devem ser substituídos ou renomeados antes de a spec ficar pronta |
| Versionamento | Atualizado no `README.md` e `CHANGELOG.md` de cada módulo |
