# Instruções para Code Agents

> Leia este arquivo integralmente antes de qualquer ação neste repositório.

---

## O que é este repositório

Este é um repositório de especificações de comportamento reutilizáveis. Cada módulo descreve **o que um sistema deve fazer**, de forma agnóstica de stack (quando possível) ou orientada a uma stack específica.

Você encontrará specs de comportamento e, nas specs orientadas a stack, exemplos de código de referência.

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
O arquivo `decisions.md` de cada módulo registra decisões arquiteturais. Se você discordar de uma decisão ou achar que existe uma abordagem melhor, não altere o comportamento nem a spec. Você pode parar e apresentar sua observação ao usuário — mas só prossiga com uma abordagem diferente após confirmação explícita. Na dúvida, pergunte. Nunca altere a spec por conta própria.

### 4. Referências entre módulos são intencionais
Quando a spec referencia outro módulo — por exemplo, "conforme `messaging/async-queue`" — verifique no `specs.json` se ele já está implementado no projeto. Se estiver, utilize o que ele expõe. Se não estiver, ele será implementado separadamente — não reproduza o comportamento dele. Se não estiver claro como integrá-lo, pergunte antes de assumir.

### 5. Restrições negativas têm peso igual às regras positivas
"Nunca exponha stack traces em respostas de erro" é tão obrigatório quanto qualquer fluxo descrito. Trate restrições negativas com a mesma prioridade.

---

## Como ler uma spec

Cada módulo contém os seguintes arquivos. Leia nesta ordem:

| Ordem | Arquivo | O que você vai encontrar |
|-------|---------|--------------------------|
| 1 | `CHANGELOG.md` | Versão atual e mudanças recentes — confirme antes de tudo |
| 2 | `README.md` | Visão geral, quando usar, quando não usar |
| 3 | `context.md` | Plataforma alvo, pré-requisitos, escopo |
| 4 | `integration.md` | Dependências — leia antes do behavior para entender as referências |
| 5 | `behavior.md` | Fluxo principal, regras, edge cases, restrições |
| 6 | `contracts.md` | Entidades, campos, tipos, validações, respostas |
| 7 | `decisions.md` | Por que foi feito assim — leia antes de questionar |
| 8 | `acceptance.md` | Critérios para validar sua implementação |
| 9 | `examples/` | Exemplos concretos de entrada e saída |

---

## O que fazer antes de implementar

1. **Verifique o `specs.json` na raiz do projeto:**
   - Módulo ausente → implementação nova, prossiga
   - Módulo presente com a mesma versão → confirme com o usuário se deve reimplementar
   - Módulo presente com versão anterior → leia o `CHANGELOG.md` e identifique o que mudou antes de continuar
   - Módulo com status `⚠️ desatualizado` → sinalize ao usuário antes de prosseguir. A spec pode ter lacunas ou inconsistências conhecidas
   - Módulo com status `🗄️ depreciado` → não implemente. Informe o usuário e indique se há um módulo substituto referenciado no `CHANGELOG.md`

2. **Leia todos os arquivos da spec na ordem definida em "Como ler uma spec"**

3. **Ao ler `integration.md`, mapeie cada dependência e verifique no `specs.json` se já está implementada no projeto**

4. **Confirme que as specs de todas as dependências foram fornecidas no prompt.** Se alguma estiver faltando:
   - Liste quais dependências estão ausentes
   - Sinalize antes de prosseguir
   - Se mesmo assim for instruído a continuar: implemente o módulo principal, marque todas as tarefas que dependem do módulo ausente como `⏭️ bloqueado` no relatório, e documente explicitamente quais comportamentos não puderam ser verificados

5. **Ao ler `context.md`, identifique a plataforma alvo e anote as variações que se aplicam** — se a plataforma não estiver clara no prompt, pergunte antes de continuar

---

## O que fazer ao encontrar ambiguidade

Se a spec for ambígua em algum ponto, **pare e pergunte**. A pergunta deve conter:

- O trecho ambíguo exatamente como está escrito
- As interpretações possíveis
- Qual informação é necessária para prosseguir

Nunca escolha uma interpretação e prossiga silenciosamente. Implementação feita sobre uma premissa errada gera retrabalho — parar e perguntar é sempre a decisão correta.

---

## Hierarquia de conflito entre arquivos

Se dois arquivos da spec descreverem comportamentos contraditórios para o mesmo caso, aplique esta ordem de prioridade — o arquivo de maior prioridade prevalece:

1. `behavior.md` — define o comportamento, sempre prevalece
2. `contracts.md` — define os contratos de dados
3. `acceptance.md` — define os critérios de validação
4. `decisions.md` — explica o contexto das decisões
5. `context.md`, `integration.md`, `README.md` — informações de suporte

Em qualquer caso de conflito: registre o conflito na seção "Inconsistências encontradas" do relatório final, indique qual arquivo prevaleceu e por quê, e sinalize para revisão humana.

---

## O que não fazer

- Não adicione funcionalidades além do especificado
- Não refatore ou reorganize o que não foi pedido
- Não altere comportamento de módulos dependentes, mesmo que pareça necessário
- Não ignore edge cases por parecerem improváveis
- Não substitua restrições de segurança por alternativas que pareçam equivalentes
- Não consulte documentação externa para tomar decisões de comportamento — a spec é suficiente. Consultar documentação técnica para implementar o comportamento especificado é válido; usá-la para justificar uma mudança de comportamento não é. Se identificar uma divergência entre a spec e a documentação técnica, notifique o usuário e aguarde confirmação expressa antes de prosseguir

---

## Validando sua implementação

A implementação não está concluída até que cada item abaixo tenha sido **testado e confirmado** — não apenas revisado visualmente:

- [ ] Cada critério de aceitação de `acceptance.md` foi coberto por um teste ou verificação explícita
- [ ] Cada edge case documentado tem comportamento correto e verificável
- [ ] Cada restrição negativa está sendo respeitada e não pode ser violada por nenhum caminho de execução
- [ ] Os contratos de entrada e saída em `contracts.md` estão sendo seguidos em todos os cenários

Se algum critério não puder ser testado com as informações disponíveis, sinalize antes de entregar — não marque como concluído.

---

## Protocolo de execução

Toda implementação segue este protocolo obrigatório em duas fases.

### Fase 1 — Implementação por tarefas

Antes de escrever qualquer código, extraia as tarefas da spec nesta ordem:

1. Cada passo do **fluxo principal** em `behavior.md` → uma tarefa
2. Cada **regra de negócio** → uma tarefa
3. Cada **edge case** → uma tarefa
4. Cada **restrição** → uma tarefa de verificação
5. Cada **critério de aceitação** em `acceptance.md` → uma tarefa de validação

Para cada tarefa:
- Execute a implementação
- Registre internamente o status: `✅ concluído` / `⚠️ parcial` / `❌ falhou` / `⏭️ bloqueado`
- Se falhar ou encontrar inconsistência, **não pare** — registre e continue para a próxima tarefa

Nunca pule uma tarefa silenciosamente. Se não puder executar, marque como `⏭️ bloqueado` com o motivo.

### Fase 2 — Relatório final obrigatório

Ao concluir todas as tarefas, emita o relatório abaixo **sempre** — mesmo que tudo tenha passado.

O relatório é a última coisa emitida. Nada deve vir depois dele.

---

## Formato do relatório final

~~~
## Relatório de Execução — [nome do módulo]

**Data:** [data]
**Spec version:** [versão lida do CHANGELOG.md do módulo]
**Status geral:** ✅ Concluído / ⚠️ Concluído com ressalvas / ❌ Falhou

---

### Resumo

| Total de tarefas | Concluídas | Parciais | Falhas | Bloqueadas |
|-----------------|------------|----------|--------|------------|
| N               | N          | N        | N      | N          |

---

### Tarefas

| # | Origem | Tarefa | Status | Observação |
|---|--------|--------|--------|------------|
| 1 | behavior.md › fluxo principal › passo 1 | [descrição curta] | ✅ | — |
| 2 | behavior.md › regras › [nome da regra] | [descrição curta] | ⚠️ | [o que foi encontrado] |
| 3 | behavior.md › edge cases › [situação] | [descrição curta] | ❌ | [o que falhou e por quê] |
| 4 | acceptance.md › AC-01 | [descrição curta] | ⏭️ | [motivo do bloqueio] |

---

### Inconsistências encontradas

> Liste problemas na spec ou na implementação que não se encaixam como falha de tarefa — ambiguidades, contradições entre arquivos, comportamento não coberto.

| # | Arquivo | Trecho | Problema | Sugestão |
|---|---------|--------|----------|----------|
| 1 | behavior.md | "..." | [descrição] | [sugestão] |

Se não houver: `Nenhuma inconsistência encontrada.`

---

### Itens que requerem atenção humana

> Decisões que o agente não pode tomar sozinho, ambiguidades implementadas com uma interpretação específica, ou itens bloqueados aguardando informação.

- [item — descrição e o que precisa ser decidido]

Se não houver: `Nenhum item requer atenção.`
~~~

---

## Legenda de status de tarefas

| Status | Significado |
|--------|-------------|
| ✅ concluído | Implementada e verificada conforme a spec |
| ⚠️ parcial | Implementada parcialmente — limitação documentada |
| ❌ falhou | Não foi possível implementar conforme a spec |
| ⏭️ bloqueado | Não executada — dependência ausente, ambiguidade ou informação faltando |

---

## Após implementar

Após confirmar que todos os critérios de `acceptance.md` foram atendidos, crie ou atualize o arquivo `specs.json` na raiz do projeto:

**Primeira aplicação do módulo:**
```json
{
  "name": "[nome do projeto]",
  "specs": {
    "[caminho/da/spec]": {
      "version": "[versão do README.md da spec]",
      "appliedAt": "[data de hoje em YYYY-MM-DD]",
      "appliedBy": "[nome do responsável, se disponível]",
      "history": []
    }
  }
}
```

**Atualização de versão de um módulo já existente:**
```json
{
  "name": "[nome do projeto]",
  "specs": {
    "[caminho/da/spec]": {
      "version": "[nova versão]",
      "appliedAt": "[data de hoje em YYYY-MM-DD]",
      "appliedBy": "[nome do responsável, se disponível]",
      "history": [
        {
          "version": "[versão anterior]",
          "appliedAt": "[data da aplicação anterior]",
          "appliedBy": "[responsável anterior]"
        }
      ]
    }
  }
}
```

A entrada raiz sempre reflete a versão atual. Ao atualizar, mova os dados anteriores para `history` — nunca os remova. Não remova entradas de outros módulos já existentes no arquivo.

---

## Estrutura do repositório

```
specs-repo/
├── agents.md           ← este arquivo
├── contributing.md     ← fluxo Git Flow para colaboradores
├── index.md            ← catálogo de todos os módulos disponíveis
├── README.md           ← visão geral para humanos
├── _meta/              ← guias para humanos
└── _template/          ← template base de cada módulo (não é uma spec real)
```

Os módulos de domínio ficam organizados por área:

```
[domínio]/[módulo]/
├── CHANGELOG.md
├── README.md
├── context.md
├── integration.md
├── behavior.md
├── contracts.md
├── decisions.md
├── acceptance.md
└── examples/
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
