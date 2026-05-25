# Como usar este repositório

Este repositório contém especificações de comportamento reutilizáveis. Cada spec descreve **o que um módulo deve fazer**, de forma agnóstica de linguagem e framework.

O objetivo é: você monta um prompt com as specs relevantes, passa para um Code Agent, e ele implementa sem lacunas para inventar comportamento.

---

## Fluxo de uso

```
1. Identificar quais módulos o projeto precisa
        ↓
2. Consultar o index.md para localizar as specs
        ↓
3. Verificar as dependências em integration.md de cada módulo
        ↓
4. Montar o prompt com os arquivos relevantes
        ↓
5. Passar ao Code Agent (Cursor, Claude Code, Codex, etc.)
        ↓
6. Registrar as specs utilizadas no specs.json do projeto
```

---

## Quais arquivos incluir no prompt

Sempre inclua `agents.md` primeiro. Em seguida, os arquivos do módulo:

| Arquivo | Incluir quando... |
|---------|-------------------|
| `agents.md` | **Sempre** — define as regras do agente |
| `README.md` | Sempre — dá o contexto geral |
| `context.md` | Sempre — define plataforma e pré-requisitos |
| `behavior.md` | Sempre — é o coração da spec |
| `contracts.md` | **Sempre** — define os contratos de dados que o agente deve seguir |
| `integration.md` | Sempre — para o agente saber as dependências |
| `acceptance.md` | **Sempre** — o agente deve verificar cada critério antes de entregar |
| `decisions.md` | **Sempre** — sem ele o agente desconhece decisões já tomadas e pode revertê-las |
| `CHANGELOG.md` | **Sempre** — indica a versão da spec e mudanças recentes; toda spec tem ao menos a entrada inicial |
| `examples/` | Quando precisar de precisão em formatos de payload |

---

## Montando o prompt

### Estrutura recomendada

```
Implemente o módulo de [nome].

**Stack:** [linguagem] + [banco] + [framework]
**Plataforma:** [API REST | Web SPA | Mobile | etc.]

**Instruções para o agente:**
[cole o conteúdo de agents.md aqui]

**Specs:**
[cole o conteúdo dos arquivos relevantes aqui]

**Restrições adicionais do projeto:**
[qualquer detalhe específico que não está na spec]
```

### Regra: sempre inclua agents.md

O arquivo `agents.md` define as regras de comportamento do agente ao usar as specs. Inclua-o em todo prompt — ele é o primeiro item depois da instrução principal.

### Dica importante

Sempre informe a **plataforma** explicitamente. Muitas specs têm comportamentos diferentes por contexto (ex: onde armazenar o token varia entre API REST, Web e Mobile).

---

## Dependências entre módulos

Alguns módulos dependem de outros. Sempre verifique `integration.md` e inclua as specs das dependências no prompt.

---

## Quando a spec não cobre seu caso

Se um comportamento necessário não está na spec:

1. **Não implemente** — nem você, nem o agente. Comportamento não especificado pode ter restrições não óbvias.
2. **Abra uma revisão** na spec correspondente e defina o comportamento explicitamente.
3. **Se precisar avançar antes da revisão**, adicione uma restrição explícita no prompt descrevendo o comportamento esperado — nunca deixe o agente decidir sozinho.

---

## Registrando specs no projeto

Após a implementação, declare as specs utilizadas no arquivo `specs.json` na raiz do projeto consumidor. Isso permite saber, no futuro, qual versão foi usada e quando uma revisão pode ser necessária.

```json
{
  "name": "nome-do-projeto",
  "specs": {
    "setup/nestjs-init": {
      "version": "0.1.0",
      "appliedAt": "YYYY-MM-DD",
      "appliedBy": "Nome"
    }
  }
}
```

- A chave de cada spec segue o caminho relativo dentro deste repositório (ex: `setup/nestjs-init`).
- `version` deve corresponder à versão declarada no `README.md` da spec no momento do uso.
- Quando uma spec evoluir para uma versão MAJOR, projetos com a versão anterior registrada devem revisar a implementação em relação ao que mudou.

---

## Consulte também

- [`_meta/prompt-recipes.md`](./prompt-recipes.md) — templates de prompt prontos
- [`_meta/how-to-write.md`](./how-to-write.md) — como criar ou editar specs
- [`_meta/glossary.md`](./glossary.md) — termos usados nas specs
