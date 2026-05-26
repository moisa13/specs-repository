# Context — Processamento Assíncrono com Fila

---

## Plataformas suportadas

| Plataforma | Suportada | Observações |
|------------|-----------|-------------|
| API REST | ✅ | Produção de jobs a partir de endpoints |
| Backend (serviço interno) | ✅ | Produção e consumo entre serviços |
| Web SPA | ❌ | Filas operam no lado do servidor |
| Mobile (iOS/Android) | ❌ | Filas operam no lado do servidor |

## Pré-requisitos

Esta spec descreve comportamento agnóstico de tecnologia — não há pré-requisitos de implementação. A infraestrutura necessária (broker, conexão, credenciais) é responsabilidade da spec de implementação correspondente.

## Escopo

**Dentro do escopo:**
- Definição de jobs: estrutura, status, opções de enfileiramento
- Comportamento do produtor: enfileirar, agendar, priorizar
- Comportamento do consumidor: processar, confirmar, falhar
- Retry automático com backoff configurável (exponencial ou fixo)
- Dead letter queue (DLQ) para jobs que esgotaram tentativas
- Concorrência: limite de workers simultâneos por fila

**Fora do escopo:**
- Configuração da infraestrutura de fila (broker, conexão, credenciais) → ver spec de implementação
- Monitoramento e observabilidade de filas → ver `observability/nestjs-metrics` (a criar)
- Comunicação entre serviços via mensageria pub/sub → escopo distinto de filas de trabalho

## Considerações de ambiente

- **Desenvolvimento:** DLQ deve estar ativa; comportamento idêntico ao de produção
- **Produção:** retry e DLQ obrigatoriamente ativos; concorrência configurada conforme carga esperada
