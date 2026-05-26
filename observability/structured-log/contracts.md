# Contracts — Registro Estruturado de Logs

> Contratos de dados agnósticos de linguagem. O agente deve mapear estes tipos para os equivalentes da stack utilizada.

---

## Entidades

### LogEntry

| Campo | Tipo | Obrigatório | Restrições | Descrição |
|-------|------|-------------|------------|-----------|
| `level` | LogLevel | ✅ | ver enum | Severidade da entrada |
| `message` | string | ✅ | non-empty | Mensagem descritiva do evento |
| `correlationId` | string | ❌ | omitido quando ausente; nunca `null`; fornecido por `observability/correlation-id` | Identificador da requisição corrente; campo omitido da entrada quando não há contexto de correlação ativo |
| `timestamp` | datetime | ✅ | UTC; ISO 8601 | Momento exato da entrada |
| `context` | string | ✅ | non-empty | Identificador do caller (ex: nome da classe) |
| `stack` | string | ❌ | nullable; apenas para `error` | Stack trace completo da exceção |

> Campos adicionais são permitidos e incluídos na entrada sem modificação, desde que não sejam sensíveis.

### LogLevel

| Valor | Quando usar |
|-------|-------------|
| `debug` | Informação detalhada para diagnóstico em desenvolvimento |
| `verbose` | Informação operacional de baixo nível |
| `info` | Eventos normais de operação |
| `warn` | Situação inesperada mas recuperável; exceções 4xx |
| `error` | Falha que requer atenção; exceções 5xx |

---

## Configuração de ambiente

| Variável | Tipo | Padrão (produção) | Padrão (dev) | Descrição |
|----------|------|-------------------|--------------|-----------|
| `LOG_LEVEL` | LogLevel | `warn` | `debug` | Nível mínimo registrado |
| `LOG_MAX_SIZE` | string | `20m` | `20m` | Tamanho máximo do arquivo antes de rotacionar |
| `LOG_MAX_FILES` | string | `14d` | `14d` | Retenção dos arquivos de log |
| `LOG_DIR` | string | `logs/` | `logs/` | Diretório de destino dos arquivos |

---

## Regras de sanitização

### Por nome exato (case-sensitive)

Os seguintes campos são sempre redijidos para `[REDACTED]`:

`password`, `cpf`, `token`, `accessToken`, `refreshToken`, `secret`, `cvv`, `otp`, `pin`, `ssn`, `privateKey`

### Por padrão (case-insensitive, aplicado ao nome do campo)

Qualquer campo cujo nome contenha um dos seguintes termos:

`auth`, `token`, `secret`, `credential`, `apikey`, `api_key`

### Em URLs

Query params cujo nome corresponda a qualquer uma das regras acima são substituídos por `[REDACTED]` na string da URL.

### Profundidade

A sanitização é aplicada recursivamente. Campos aninhados além do 5º nível não são sanitizados.

---

## Entrada — Chamada ao logger

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `level` | LogLevel | ✅ | Severidade da entrada |
| `message` | string | ✅ | Mensagem descritiva do evento |
| `context` | string | ❌ | Identificador do caller (ex: nome da classe); pode ser inferido pela implementação |
| `[campos extras]` | any | ❌ | Dados adicionais de contexto; passados junto à mensagem e incluídos na entrada sem modificação |

---

## Saída — Arquivos produzidos

| Arquivo | Conteúdo | Formato |
|---------|----------|---------|
| `{LOG_DIR}/YYYY-MM-DD.log` | Todos os níveis | JSON estruturado (um objeto por linha) |
| `{LOG_DIR}/YYYY-MM-DD-error.log` | Apenas `error` | JSON estruturado (um objeto por linha) |

Cada linha dos arquivos corresponde a um `LogEntry` completo conforme a entidade definida acima.

---

## Erros

O logger não lança erros para o caller em nenhuma circunstância. Falhas internas (ex: impossibilidade de gravar no arquivo) são silenciadas. Este é um contrato explícito — o caller nunca precisa tratar exceções de log.

---

## Adições ao agents.md

Ao aplicar esta spec, acrescentar a seção abaixo ao `agents.md` do projeto:

```markdown
## Logging — Registro Estruturado (observability/structured-log)

- Arquivos de log seguem o padrão `YYYY-MM-DD.log` e `YYYY-MM-DD-error.log` no diretório `LOG_DIR` (padrão: `logs/`)
- Toda entrada de log deve conter `level`, `message`, `timestamp` (UTC, ISO 8601) e `context`
- Dados sensíveis devem ser sanitizados antes de qualquer chamada ao logger; campos protegidos: `password`, `cpf`, `token`, `accessToken`, `refreshToken`, `secret`, `cvv`, `otp`, `pin`, `ssn`, `privateKey` e qualquer chave contendo `auth`, `token`, `secret`, `credential`, `apikey`, `api_key`
- O logger nunca lança exceções — o caller não precisa envolver chamadas de log em try/catch
- Variáveis de ambiente: `LOG_LEVEL`, `LOG_MAX_SIZE`, `LOG_MAX_FILES`, `LOG_DIR`
```
