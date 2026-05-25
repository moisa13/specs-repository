# Prompt Recipes

Templates de prompt prontos para os cenários mais comuns de uso das specs com Code Agents.

Substitua os valores entre `[colchetes]` pelo contexto do seu projeto.

> **Regra geral:** todo prompt deve incluir o conteúdo de `agents.md` antes das specs. Os templates abaixo já indicam onde inserí-lo.

---

## Recipe 1 — Implementar um módulo completo

```
Implemente o módulo de [nome do módulo] seguindo as especificações abaixo.

**Stack:** [ex: Node.js + Express + PostgreSQL]
**Plataforma:** [ex: API REST]
**Arquitetura:** [ex: MVC / Clean Architecture / Hexagonal]

**Convenções do projeto:**
- [ex: Use async/await, sem callbacks]
- [ex: Erros lançados como classes que estendem AppError]
- [ex: Respostas sempre no formato { data, error, meta }]

---

**Instruções para o agente (agents.md):**
[cole o conteúdo de agents.md aqui]

---

**Spec:**
[cole aqui o conteúdo dos arquivos da spec]

---

**Entregáveis esperados:**
- Implementação completa do módulo
- Testes unitários cobrindo os critérios de acceptance.md
- `specs.json` criado ou atualizado na raiz do projeto com a spec e versão utilizadas
- Sem decisões de comportamento fora da spec — se algo não estiver coberto, pergunte antes de assumir
```

---

## Recipe 2 — Implementar só uma camada específica

```
Implemente apenas a camada de [ex: validação / repositório / serviço] do módulo de [nome].

A implementação deve seguir a spec abaixo. Não implemente outras camadas.

**Stack:** [stack]
**Interfaces existentes:** [descreva o que já existe para que o agente se integre]

---

**Instruções para o agente (agents.md):**
[cole o conteúdo de agents.md aqui]

---

**Spec:**
[cole README.md, context.md, behavior.md, contracts.md, integration.md, acceptance.md, decisions.md e CHANGELOG.md]

---

**Atenção:**
- Não crie rotas ou controllers
- Não altere o schema do banco
- Apenas a camada de [nome da camada]
```

---

## Recipe 3 — Revisar implementação existente contra a spec

```
Revise a implementação abaixo e verifique se está em conformidade com a spec.

**Para cada divergência encontrada:**
1. Aponte o trecho de código divergente
2. Cite a regra da spec que está sendo violada
3. Sugira a correção

Não corrija automaticamente — apenas aponte e sugira.

---

**Instruções para o agente (agents.md):**
[cole o conteúdo de agents.md aqui]

---

**Spec:**
[cole README.md, context.md, behavior.md, contracts.md, integration.md, acceptance.md, decisions.md e CHANGELOG.md]

---

**Implementação atual:**
[cole o código ou indique os arquivos]
```

---

## Recipe 4 — Implementar módulo com dependências

```
Implemente o módulo de [nome].

Este módulo depende de [módulo dependente], que já está implementado no projeto.
Siga as specs de ambos os módulos abaixo.

**Stack:** [stack]
**Localização do módulo dependente no projeto:** [caminho]

---

**Instruções para o agente (agents.md):**
[cole o conteúdo de agents.md aqui]

---

**Spec do módulo principal:**
[cole os arquivos do módulo principal]

---

**Spec do módulo dependente (referência):**
[cole behavior.md e decisions.md do módulo dependente]

---

**Importante:**
- Não reimplemente o módulo dependente. Apenas consuma-o conforme definido em integration.md.
- Crie ou atualize o `specs.json` na raiz do projeto com a spec principal e versão utilizadas.
```

---

## Recipe 5 — Criar testes a partir da spec

```
Escreva testes para o módulo de [nome] com base nos critérios de aceitação abaixo.

**Framework de testes:** [ex: Jest / Pytest / RSpec]
**Tipo de teste:** [ex: unitário / integração / e2e]
**Estilo:** [ex: BDD com describe/it / AAA (Arrange-Act-Assert)]

---

**Instruções para o agente (agents.md):**
[cole o conteúdo de agents.md aqui]

---

**Critérios de aceitação (acceptance.md):**
[cole acceptance.md]

**Comportamento esperado (behavior.md):**
[cole behavior.md]

**Contratos de dados (contracts.md):**
[cole contracts.md]

**Decisões arquiteturais (decisions.md):**
[cole decisions.md]

**Contexto e plataforma (README.md, context.md, integration.md, CHANGELOG.md):**
[cole README.md, context.md, integration.md e CHANGELOG.md]

---

**Requisitos:**
- Cada critério de aceitação deve ter ao menos um teste
- Edge cases documentados na spec devem ter testes próprios
- Não teste comportamento que não está na spec
```

---

## Recipe 6 — Atualizar implementação após mudança de spec

```
A spec do módulo de [nome] foi atualizada. Atualize a implementação para refletir as mudanças.

**O que mudou (CHANGELOG.md):**
[cole a entrada do changelog com as mudanças]

---

**Instruções para o agente (agents.md):**
[cole o conteúdo de agents.md aqui]

---

**Spec atual completa:**
[cole os arquivos da spec atualizada]

**Implementação atual:**
[indique os arquivos relevantes]

---

**Instruções:**
- Altere apenas o necessário para conformidade com a nova spec
- Não refatore código não relacionado às mudanças
- Atualize os testes afetados
- Aponte qualquer mudança que possa ter impacto em outros módulos
- Atualize o campo `version` da spec no `specs.json` do projeto para a nova versão
```
