# Behavior — Registro Estruturado de Logs

---

## Objetivo

Garantir que toda atividade relevante da aplicação seja registrada de forma estruturada e livre de dados sensíveis, sem impactar o fluxo principal da aplicação.

---

## Fluxo principal — Requisição HTTP

1. O sistema registra o início do processamento da requisição
2. Controllers e services executam normalmente; qualquer chamada ao logger produz uma entrada de log com os campos obrigatórios
3. Ao finalizar, o sistema registra método HTTP, URL, status de resposta e duração total em milissegundos
4. Antes de qualquer escrita, o conteúdo do log é sanitizado conforme as regras desta spec
5. A entrada sanitizada é gravada no(s) arquivo(s) de log correspondente(s)

---

## Fluxo de exceção

1. Uma exceção é capturada pelo handler de exceções
2. O sistema classifica pela faixa HTTP do status de resposta:
   - **4xx** → registra no nível `warn`; sem stack trace
   - **5xx** → registra no nível `error`; inclui stack trace completo
3. O contexto da requisição e o corpo da exceção são sanitizados antes do log
4. A entrada é gravada nos arquivos correspondentes

---

## Regras de negócio

- Se um `correlationId` estiver disponível no contexto da requisição, ele deve ser incluído em todas as entradas de log geradas durante aquela requisição — sem que o código de negócio precise passá-lo explicitamente
- Toda entrada de log deve conter os campos obrigatórios definidos em `contracts.md`
- O conteúdo de qualquer entrada de log deve ser sanitizado antes de ser escrito
- Em produção, nenhuma saída de log deve ir para o console
- Dois arquivos de log são mantidos simultaneamente:
  - `YYYY-MM-DD.log` — todos os níveis
  - `YYYY-MM-DD-error.log` — apenas `error`
- Arquivos de log são rotacionados diariamente e comprimidos com gzip após a rotação
- Arquivos mais antigos que `LOG_MAX_FILES` são removidos automaticamente
- Quando o arquivo ativo atingir `LOG_MAX_SIZE`, uma rotação imediata é realizada
- Uma falha no processo de log nunca interrompe ou afeta o fluxo da requisição

---

## Sanitização

A sanitização ocorre antes de qualquer escrita e é aplicada recursivamente até 5 níveis de profundidade. Um campo é redijido para `[REDACTED]` quando:

- Seu nome corresponde exatamente a um dos nomes protegidos: `password`, `cpf`, `token`, `accessToken`, `refreshToken`, `secret`, `cvv`, `otp`, `pin`, `ssn`, `privateKey`
- Seu nome contém um dos padrões (case-insensitive): `auth`, `token`, `secret`, `credential`, `apikey`, `api_key`
- Aparece como query param de uma URL e seu nome se enquadra em qualquer uma das regras acima

---

## Edge cases

| Situação | Comportamento esperado |
|----------|----------------------|
| Requisição sem contexto HTTP (ex: job assíncrono) | Log gravado sem `correlationId`; demais campos obrigatórios mantidos |
| `correlationId` indisponível no contexto | Campo `correlationId` omitido da entrada; sem erro lançado |
| Requisições concorrentes com `correlationId` ativo | Cada entrada carrega apenas o `correlationId` da sua própria requisição; sem vazamento entre contextos |
| Erro durante a sanitização de um campo | Campo omitido da entrada de log; o log prossegue normalmente |
| Arquivo atinge `LOG_MAX_SIZE` antes da rotação diária | Rotação imediata; novo arquivo criado no mesmo dia |
| Campo sensível aninhado além do 5º nível | Não sanitizado; trade-off documentado em `decisions.md` |
| Falha ao gravar no arquivo de log | Falha ignorada silenciosamente; fluxo da requisição não é interrompido |
| Campo adicional não previsto na spec | Incluído na entrada sem modificação, desde que não seja sensível |

---

## Restrições

- Nunca escrever dados sensíveis em texto claro em qualquer transporte de log
- Nunca ativar o transporte de console em ambiente de produção (`NODE_ENV=production`)
- Nunca propagar uma falha de escrita de log para o fluxo da requisição
- Nunca incluir stack trace em logs de exceções 4xx

---

## O que esta spec NÃO cobre

- Geração, propagação e rastreamento por `correlationId` → ver `observability/correlation-id` (a criar)
- Captura de erros em serviços externos → ver `observability/error-tracking` (a criar)
- Configuração de biblioteca ou framework de logging → ver spec de implementação (ex: `setup/nestjs/nestjs-logging`)
- Tracing distribuído entre serviços → ver `observability/tracing` (a criar)
- Formato de log específico de uma linguagem ou framework → ver spec de implementação
