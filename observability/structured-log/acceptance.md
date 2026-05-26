# Acceptance Criteria — Registro Estruturado de Logs

> Formato: **Dado** [contexto] **Quando** [ação] **Então** [resultado esperado]

---

## Fluxo principal

**AC-01 — Campos obrigatórios presentes**
- **Dado** que qualquer entrada de log é gerada
- **Quando** a entrada é gravada no arquivo
- **Então** contém `level`, `message`, `timestamp` (UTC, ISO 8601) e `context`

**AC-02 — Requisição finalizada registra metadados**
- **Dado** que uma requisição HTTP é concluída com sucesso
- **Quando** a resposta é enviada
- **Então** uma entrada de log registra o método HTTP, a URL, o status de resposta e a duração total em milissegundos

---

## Classificação de exceções

**AC-03 — Exceção 4xx registrada como warn**
- **Dado** que uma exceção com status 4xx é lançada
- **Quando** o handler de exceções processa o erro
- **Então** a entrada de log tem nível `warn`
- **E** não contém stack trace

**AC-04 — Exceção 5xx registrada como error com stack trace**
- **Dado** que uma exceção com status 5xx é lançada
- **Quando** o handler de exceções processa o erro
- **Então** a entrada de log tem nível `error`
- **E** contém o stack trace completo da exceção

---

## Sanitização

**AC-05 — Campo sensível por nome exato é redijido**
- **Dado** que uma entrada de log contém um campo chamado `password`
- **Quando** a entrada é gravada
- **Então** o valor do campo `password` é `[REDACTED]`

**AC-06 — Campo sensível por padrão regex é redijido**
- **Dado** que uma entrada de log contém um campo chamado `authorizationHeader`
- **Quando** a entrada é gravada
- **Então** o valor do campo `authorizationHeader` é `[REDACTED]`

**AC-07 — Sanitização recursiva até 5 níveis**
- **Dado** que uma entrada de log contém um objeto aninhado com um campo `token` no 3º nível
- **Quando** a entrada é gravada
- **Então** o campo `token` aninhado é `[REDACTED]`

**AC-08 — Campo além do 5º nível não é sanitizado**
- **Dado** que uma entrada de log contém um campo sensível aninhado no 6º nível
- **Quando** a entrada é gravada
- **Então** o campo não é redijido (comportamento aceito; documentado em `decisions.md`)

---

## Transportes

**AC-09 — Console inativo em produção**
- **Dado** que o ambiente é produção (`NODE_ENV=production`)
- **Quando** qualquer entrada de log é gerada
- **Então** nenhuma saída aparece no console

**AC-10 — Arquivo de erros separado**
- **Dado** que uma entrada de log com nível `error` é gerada
- **Quando** a entrada é gravada
- **Então** aparece tanto em `YYYY-MM-DD.log` quanto em `YYYY-MM-DD-error.log`
- **E** uma entrada de nível `warn` aparece apenas em `YYYY-MM-DD.log`

**AC-11 — Falha de log não interrompe requisição**
- **Dado** que ocorre um erro ao tentar gravar no arquivo de log
- **Quando** a requisição continua sendo processada
- **Então** a resposta HTTP é enviada normalmente
- **E** nenhuma exceção relacionada ao log é propagada ao cliente

---

## Edge cases

**AC-12 — correlationId ausente é omitido da entrada, sem erro**
- **Dado** que não há `correlationId` disponível no contexto (ex: job assíncrono, contexto não inicializado)
- **Quando** o logger gera uma entrada
- **Então** o campo `correlationId` está ausente do JSON gravado — nunca presente com valor `null`
- **E** os demais campos obrigatórios (`level`, `message`, `timestamp`, `context`) estão presentes
- **E** nenhuma exceção é lançada

**AC-13 — Requisições concorrentes têm correlationIds isolados**
- **Dado** que duas requisições HTTP são processadas simultaneamente, cada uma com seu próprio `correlationId`
- **Quando** ambas geram entradas de log
- **Então** as entradas de cada requisição contêm apenas o `correlationId` da sua própria requisição
- **E** nenhum `correlationId` de uma requisição aparece em entradas da outra

---

## O que caracteriza uma implementação incorreta

- Dados sensíveis (password, token, etc.) em texto claro em qualquer arquivo de log
- Stack trace presente em log de exceção 4xx
- Console ativo em ambiente de produção
- Falha de escrita de log lança exceção para o cliente

---

## Cobertura mínima de testes

- [ ] Fluxo completo de requisição com campos obrigatórios presentes
- [ ] Exceção 4xx → nível `warn`, sem stack trace
- [ ] Exceção 5xx → nível `error`, com stack trace
- [ ] Sanitização de campo sensível por nome exato
- [ ] Sanitização de campo sensível por padrão regex
- [ ] Console desativado em produção
- [ ] Ausência de `correlationId` no contexto → campo omitido do JSON, sem erro, demais campos obrigatórios presentes
- [ ] Requisições concorrentes → `correlationId` isolado por requisição
