# Specs Repository

> Catálogo de especificações de comportamento reutilizáveis — agnósticas de stack (quando possível), prontas para uso com Code Agents.

---

## Motivação

Todo sistema tem módulos comuns: login, auditoria, cache, filas, logging. A tendência natural é replanejar esses módulos do zero em cada projeto — com variações sutis que acumulam inconsistência ao longo do tempo.

Este repositório aplica Spec-Driven Development a esse problema: a spec é escrita uma vez, validada, e reutilizada como fonte de verdade em qualquer projeto que precise daquele comportamento. A implementação deriva da spec — não o contrário.

**O resultado:** você para de tomar as mesmas decisões repetidas vezes, e os Code Agents param de inventar comportamento que já foi especificado.

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
├── agents.md             ← instruções para Code Agents (inclua sempre no prompt)
├── contributing.md       ← fluxo Git Flow para criar e editar specs
├── index.md              ← catálogo de todos os módulos
├── README.md             ← visão geral e convenções do repositório
│
├── _meta/                ← como usar e contribuir
│   ├── how-to-use.md
│   ├── how-to-write.md
│   ├── prompt-recipes.md
│   └── glossary.md
│
└── _template/            ← template base para novos módulos
    ├── CHANGELOG.md
    ├── README.md
    ├── context.md
    ├── integration.md
    ├── behavior.md
    ├── contracts.md
    ├── decisions.md
    ├── acceptance.md
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
