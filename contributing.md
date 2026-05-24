# Contributing

Este guia define como colaborar neste repositório usando Git Flow.

---

## Branches principais

| Branch | Propósito |
|--------|-----------|
| `main` | Contém specs aprovadas para consumo (`✅ pronto`) e specs mantidas apenas para referência histórica (`🗄️ depreciado`) |
| `develop` | Integração contínua — specs em elaboração ou revisão convergem aqui antes de irem para `main` |

Nunca commite diretamente em `main` ou `develop`.

---

## Fluxo de trabalho

### Criando ou editando uma spec

```bash
# 1. Parta sempre de develop atualizado
git checkout develop
git pull origin develop

# 2. Crie uma branch de feature
git checkout -b feature/[domínio]/[módulo]

# Exemplos:
# feature/auth/login
# feature/audit/event-log
# feature/common/pagination

# 3. Trabalhe na spec normalmente

# 4. Commite seguindo a convenção de mensagens (ver abaixo)

# 5. Abra um Pull Request para develop
```

### Corrigindo uma spec já publicada (hotfix)

Use quando uma spec em `main` tem um erro que afeta implementações em andamento.

```bash
# 1. Parta de main
git checkout main
git pull origin main

# 2. Crie uma branch de hotfix
git checkout -b hotfix/[domínio]/[módulo]-[descrição-curta]

# Exemplo:
# hotfix/auth/login-timing-attack-restriction

# 3. Corrija, commite e abra PR para main E develop
```

### Depreciando uma spec

```bash
# 1. Parta de develop atualizado
git checkout develop
git pull origin develop

# 2. Crie uma branch de feature
git checkout -b feature/[domínio]/[módulo]-deprecate

# 3. Atualize o status no README.md do módulo para 🗄️ depreciado
# 4. Adicione entrada no CHANGELOG.md explicando o motivo e o módulo substituto (se houver)
# 5. Abra PR para develop
# 6. Após merge em develop, abra PR de develop para main
#    A spec entra em main com status 🗄️ depreciado — não a remova.
#    Main deve refletir o estado atual de todas as specs, inclusive as depreciadas.
```

---

## Convenção de mensagens de commit

```
ação(escopo): descrição curta no imperativo
```

### Ações

| Ação | Quando usar |
|------|-------------|
| `feat` | Nova spec ou novo arquivo em spec existente |
| `fix` | Correção de erro em spec (inconsistência, ambiguidade, edge case faltando) |
| `refactor` | Reestruturação de conteúdo sem alterar comportamento especificado |
| `test` | Adição ou correção de exemplos em `examples/` |
| `deprecate` | Marcação de spec como depreciada |
| `chore` | Mudanças estruturais sem impacto em conteúdo (renomear arquivo, reorganizar pasta) |
| `docs` | Mudanças em `_meta/`, `README.md`, `index.md`, `agents.md`, `contributing.md` |

### Exemplos

```bash
git commit -m "feat(auth/login): adiciona spec inicial de login por email e senha"
git commit -m "fix(auth/login): adiciona edge case de conta inativa"
git commit -m "fix(audit/event-log): corrige ambiguidade no campo actor"
git commit -m "refactor(auth/login): reorganiza behavior.md sem alterar comportamento"
git commit -m "test(auth/login): adiciona exemplo login-account-locked"
git commit -m "deprecate(auth/session): substituído por auth/token"
git commit -m "docs(agents): adiciona instrução sobre módulos depreciados"
git commit -m "chore: renomeia INDEX.md para index.md"
```

### Regras

- Descrição em português, no imperativo: "adiciona", "corrige", "remove" — não "adicionado" ou "adicionando"
- Máximo 72 caracteres na primeira linha
- Se necessário, adicione detalhes no corpo do commit separado por linha em branco

---

## Pull Requests

### Título

Siga o mesmo padrão dos commits:
```
feat(auth/login): spec inicial de login por email e senha
```

### Descrição mínima

```markdown
## O que muda
[descreva o que foi adicionado, alterado ou corrigido]

## Checklist
- [ ] Todos os arquivos do módulo estão preenchidos (sem seções em branco)
- [ ] `decisions.md` contém ao menos uma entrada ADR
- [ ] Pelo menos 3 edge cases documentados no `behavior.md`
- [ ] Critérios de aceitação cobrem fluxo feliz e pelo menos 2 falhas
- [ ] Dependências identificadas no `integration.md`
- [ ] `index.md` atualizado com o módulo e status correto
- [ ] `CHANGELOG.md` e `Spec version` no `README.md` estão sincronizados
- [ ] Revisão registrada no campo "Revisado por" do `README.md`
```

### Regras de aprovação

- PRs para `develop`: revisão de ao menos uma pessoa (ou auto-revisão documentada após um dia, em times solo)
- PRs para `main`: apenas de `develop` ou `hotfix/*`, após todos os itens do checklist marcados; cada módulo incluído no PR deve ter status `✅ pronto` ou `🗄️ depreciado`

---

## Atualizando o status de uma spec

O status no `index.md` e no `README.md` do módulo devem sempre estar em sincronia:

```bash
# Ao copiar o _template/ (estrutura criada, arquivos ainda vazios)
# → status: 🚧 template

# Ao começar a preencher os arquivos
# → status: 📝 rascunho (na branch de feature, ainda não em develop)

# Ao abrir PR para develop
# → status: 📝 rascunho (visível em develop para revisão)

# Ao aprovar e mergear em main
# → status: ✅ pronto

# Ao identificar desatualização
# → status: ⚠️ desatualizado (mantenha em `develop` até a revisão ficar pronta)

# Ao depreciar
# → status: 🗄️ depreciado
```

---

## Inicializando o repositório (primeira vez)

```bash
git init
git remote add origin [url-do-repositório]

git checkout -b develop
git add .
git commit -m "chore: estrutura inicial do repositório de specs"
git push -u origin develop

git checkout -b main
git push -u origin main

# Defina develop como branch padrão no GitHub/GitLab
```
