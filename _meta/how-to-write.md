# Como escrever specs

Este guia define os padrões para criar e editar especificações neste repositório.

O objetivo de cada spec é ser **clara o suficiente para que um Code Agent implemente corretamente na primeira tentativa**, sem decisões implícitas.

---

## Princípios fundamentais

### 1. Seja explícito, não implícito
Se um comportamento parece óbvio para você, ainda assim escreva. O agente não tem o seu contexto.

❌ "Trate erros adequadamente."
✅ "Em caso de entrada inválida, retorne um erro indicando os campos com problema e a mensagem de cada um."

### 2. Escreva o que NÃO deve acontecer
Restrições negativas eliminam uma classe inteira de erros.

✅ "Não revele se o e-mail existe ou não — retorne a mesma mensagem para e-mail inexistente e senha incorreta."

### 3. Edge cases são obrigatórios, não opcionais
Se você consegue imaginar o caso, escreva. Se o agente não encontrar a regra, vai inventar uma.

### 4. Seja agnóstico de implementação
A spec descreve **comportamento**, não **como implementar**.

❌ "Use bcrypt com 12 rounds para hashear a senha."
✅ "A senha deve ser armazenada como hash seguro. Nunca em texto puro."

### 5. Referências entre módulos, não duplicação
Se um comportamento pertence a outro módulo, referencie — não copie.

✅ "O evento de login deve ser registrado conforme `audit/event-log`."

---

## Criando um novo módulo

**Domínio** é o agrupamento funcional de primeiro nível (ex: `auth`, `audit`, `rbac`). Consulte o glossário para a definição completa e exemplos de nomes.

```bash
# 1. Defina o domínio e o nome do módulo
#    Formato: [domínio]/[módulo]
#    Exemplos: auth/login, audit/event-log, common/pagination

# 2. Copie o template
cp -r _template/ [domínio]/[módulo]/

# 3. Adicione ao index.md com status "🚧 template"

# 4. Preencha os arquivos na ordem:
#    README.md → context.md → behavior.md → contracts.md
#    → integration.md → acceptance.md → decisions.md → CHANGELOG.md
#    → examples/ (adicione um arquivo por cenário relevante)
#    Ao começar a preencher, atualize o status para "📝 rascunho"

# 5. Após revisão e merge em main, atualize para "✅ pronto"
```

---

## Guia por arquivo

### README.md
- O que é este módulo em 2-3 frases
- Quando usar e quando NÃO usar
- Links para módulos relacionados
- Versão e data da última revisão

### context.md
- Plataformas suportadas (API REST, Web, Mobile, etc.)
- Pré-requisitos (o que deve existir antes deste módulo)
- Escopo (o que está dentro e fora desta spec)
- Variações de comportamento por plataforma

### behavior.md
- Fluxo principal (passo a passo numerado)
- Regras de negócio (lista clara)
- Edge cases (cada um com comportamento esperado)
- O que esta spec NÃO cobre (com referência para onde está)

### contracts.md
- Entidades envolvidas (campos, tipos, restrições)
- Formato de entrada esperado
- Formato de saída esperado
- Códigos de status ou resposta possíveis

### integration.md
- Dependências (quais outros módulos este usa)
- Dependentes (quais módulos dependem deste)
- Eventos emitidos (se aplicável)
- Eventos consumidos (se aplicável)

### acceptance.md
- Critérios de aceitação: "Dado X, quando Y, então Z"
- Cenários de sucesso
- Cenários de falha esperada
- O que caracteriza uma implementação incorreta

### decisions.md
- Decisões arquiteturais tomadas (formato ADR)
- Para cada decisão: contexto, opções consideradas, decisão tomada, motivo
- Decisões que parecem erradas mas têm razão de existir

### CHANGELOG.md
- Histórico de versões da spec
- O que mudou em cada versão
- Impacto em implementações existentes

### examples/
- Um arquivo por cenário relevante
- Nomenclatura obrigatória: `[ação]-[resultado].md`
- Exemplos: `login-success.md`, `login-account-locked.md`, `create-user-duplicate-email.md`
- Conteúdo: contexto do cenário, entrada, saída esperada (sucesso ou falha)
- No `_template/`, o arquivo `examples/template.md` é apenas um molde. Renomeie-o
  para um cenário real, ou duplique-o, antes de considerar a spec pronta.

> **Regra de sincronização:** ao criar ou atualizar uma entrada no `CHANGELOG.md`, o campo `Spec version` no `README.md` do módulo deve ser atualizado para o mesmo valor. São a mesma informação em dois lugares — mantê-los em sincronia é obrigatório.

---

## Qualidade mínima para "✅ pronto"

- [ ] Todos os arquivos preenchidos (sem seções em branco)
- [ ] `decisions.md` contém ao menos uma entrada ADR
- [ ] Pelo menos 3 edge cases documentados no `behavior.md`
- [ ] Critérios de aceitação cobrem fluxo feliz e pelo menos 2 falhas
- [ ] Dependências identificadas no `integration.md`
- [ ] `index.md` atualizado com o módulo e status correto
- [ ] Versão definida e sincronizada entre `README.md` e `CHANGELOG.md`
- [ ] Revisão registrada no campo `Revisado por` do `README.md`

> **Sobre revisão:** o campo `Revisado por` deve ser sempre preenchido com nome e data — mesmo em times de uma pessoa. Nesse caso, registre uma auto-revisão após intervalo mínimo de um dia. O objetivo é tornar o critério auditável, não burocrático.
