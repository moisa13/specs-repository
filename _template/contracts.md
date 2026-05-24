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
