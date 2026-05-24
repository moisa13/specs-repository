# agents.md

> Este arquivo contém instruções para Code Agents (Cursor, Claude Code, Codex, etc.).
> Leia este arquivo integralmente antes de qualquer ação neste repositório.

---

## O que é este repositório

Este é um repositório de especificações de comportamento reutilizáveis. Cada módulo descreve **o que um sistema deve fazer**, de forma agnóstica de linguagem e framework.

Você não vai encontrar código de implementação aqui. Apenas specs.

---

## Sua função ao usar este repositório

Você será acionado com uma ou mais specs deste repositório e uma instrução de implementação. Seu papel é implementar o comportamento descrito fielmente — sem adicionar, remover ou alterar comportamento não especificado.

---

## Regras obrigatórias

### 1. A spec é a fonte de verdade
Se a spec diz X, implemente X. Não "melhore" a spec. Não assuma que algo foi esquecido. Se um comportamento não está na spec, não o implemente.

### 2. Nunca invente comportamento não especificado
Se a spec não cobre um caso e você precisaria tomar uma decisão de comportamento para prosseguir, **pare e pergunte**. Não assuma. Não escolha o que parece mais razoável.

### 3. Decisões já foram tomadas
O arquivo `decisions.md` de cada módulo registra decisões arquiteturais. Se você discordar de uma decisão ou achar que existe uma abordagem melhor, não altere o comportamento. Você pode mencionar sua observação, mas deve implementar conforme a spec.

### 4. Referências entre módulos são intencionais
Quando a spec diz "conforme `módulo/x`", significa que aquele módulo já existe ou será fornecido. Não reimplemente o comportamento do módulo referenciado. Apenas consuma-o.

### 5. Restrições negativas têm peso igual às regras positivas
"Nunca retorne mensagens de erro diferentes para e-mail inexistente e senha incorreta" é tão obrigatório quanto qualquer fluxo descrito. Trate restrições negativas com a mesma prioridade.

---

## Como ler uma spec

Cada módulo contém os seguintes arquivos. Leia nesta ordem:

| Ordem | Arquivo | O que você vai encontrar |
|-------|---------|--------------------------|
| 1 | `README.md` | Visão geral, quando usar, quando não usar |
| 2 | `context.md` | Plataforma alvo, pré-requisitos, escopo |
| 3 | `behavior.md` | Fluxo principal, regras, edge cases, restrições |
| 4 | `contracts.md` | Entidades, campos, tipos, validações, respostas |
| 5 | `integration.md` | Dependências e módulos que dependem deste |
| 6 | `acceptance.md` | Critérios para validar sua implementação |
| 7 | `decisions.md` | Por que foi feito assim — leia antes de questionar |
| 8 | `CHANGELOG.md` | Histórico de versões — verifique se a spec está atualizada |
| 9 | `examples/` | Exemplos concretos de entrada e saída |

---

## O que fazer antes de implementar

1. Leia todos os arquivos da spec fornecida
2. Identifique as dependências em `integration.md`
3. Confirme que as specs das dependências também foram fornecidas
4. Se alguma spec de dependência estiver faltando, sinalize antes de prosseguir
5. Identifique a plataforma alvo em `context.md` e aplique as variações correspondentes

---

## O que fazer ao encontrar ambiguidade

Se a spec for ambígua em algum ponto:

1. Cite o trecho ambíguo exatamente como está escrito
2. Descreva as interpretações possíveis
3. Pergunte qual interpretação é a correta

Não escolha uma interpretação e prossiga silenciosamente.

---

## O que não fazer

- Não adicione funcionalidades além do especificado
- Não refatore ou reorganize o que não foi pedido
- Não altere comportamento de módulos dependentes, mesmo que pareça necessário
- Não ignore edge cases por parecerem improváveis
- Não substitua restrições de segurança por alternativas que pareçam equivalentes
- Não consulte documentação externa para tomar decisões de comportamento — a spec é suficiente

---

## Validando sua implementação

Antes de considerar a implementação concluída, verifique cada critério em `acceptance.md`:

- Cada critério de aceitação foi atendido?
- Cada edge case documentado tem comportamento correto?
- Cada restrição negativa está sendo respeitada?
- Os contratos de entrada e saída em `contracts.md` estão sendo seguidos?

Se algum critério não puder ser atendido com as informações disponíveis, sinalize antes de entregar.

---

## Estrutura do repositório

```
specs-repo/
├── agents.md           ← este arquivo
├── README.md           ← visão geral para humanos
├── index.md            ← catálogo de todos os módulos disponíveis
├── contributing.md     ← fluxo Git Flow para colaboradores (não relevante para sua execução)
├── _meta/              ← guias para humanos (não relevante para sua execução)
└── _template/          ← template base de cada módulo (não é uma spec real)
```

Os módulos de domínio ficam organizados por área:

```
[domínio]/[módulo]/
├── README.md
├── context.md
├── behavior.md
├── contracts.md
├── integration.md
├── acceptance.md
├── decisions.md
├── CHANGELOG.md
└── examples/
    └── [ação]-[resultado].md
```

---

## Glossário rápido

| Termo | Significado |
|-------|-------------|
| Spec | Especificação de comportamento de um módulo |
| Fluxo principal | O caminho feliz — quando tudo ocorre conforme esperado |
| Edge case | Situação fora do fluxo principal com comportamento definido |
| Restrição | O que a implementação nunca deve fazer |
| ADR | Registro de decisão arquitetural em `decisions.md` |
| Agnóstico de stack | Comportamento independente de linguagem ou framework |

Para o glossário completo, consulte `_meta/glossary.md`.
