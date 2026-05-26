# Context — Registro Estruturado de Logs

---

## Plataformas suportadas

| Plataforma | Suportada | Observações |
|------------|-----------|-------------|
| API REST | ✅ | Comportamento principal da spec |
| Web SPA | ❌ | Logs de browser fora do escopo |
| Mobile (iOS/Android) | ❌ | |
| Backend (serviço interno) | ✅ | Inclui workers e jobs assíncronos |

## Pré-requisitos

- [ ] Aplicação de servidor capaz de interceptar o ciclo de vida de requisições HTTP
- [ ] Sistema de arquivos com permissão de escrita no diretório de logs

## Escopo

**Dentro do escopo:**
- Estrutura e campos obrigatórios de uma entrada de log
- Regras de sanitização de dados sensíveis
- Classificação de exceções por faixa HTTP (4xx / 5xx)
- Retenção, rotação e compressão de arquivos de log
- Diferença de comportamento entre ambientes (produção / não-produção)

**Fora do escopo:**
- Geração, propagação e rastreamento por `correlationId` → ver `observability/correlation-id` (a criar)
- Captura de erros em serviços externos (ex: Sentry) → ver `observability/error-tracking` (a criar)
- Tracing distribuído entre serviços → ver `observability/tracing` (a criar)
- Logs de auditoria com persistência em banco → escopo distinto
- Configuração de infraestrutura de coleta de logs (ELK, Loki, etc.) → escopo de infraestrutura
- Configuração de biblioteca ou framework → ver spec de implementação (ex: `setup/nestjs/nestjs-logging`)

## Considerações de ambiente

- **Desenvolvimento:** console ativo com formatação colorida; nível padrão `debug`
- **Produção:** console desativado; apenas transportes de arquivo ativos; nível padrão `warn`
