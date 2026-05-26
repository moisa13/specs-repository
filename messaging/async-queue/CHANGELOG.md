# Changelog — Processamento Assíncrono com Fila

> Versões seguem Semantic Versioning: MAJOR.MINOR.PATCH
>
> - **MAJOR** — mudança que quebra implementações existentes
> - **MINOR** — nova regra ou edge case (implementações existentes precisam de revisão)
> - **PATCH** — correção de texto, sem mudança de comportamento

---

## [0.1.0] — 2026-05

### Adicionado
- Versão inicial da spec
- Fluxo principal de produção e consumo de jobs
- Fluxo de retry com backoff exponencial
- Comportamento de dead letter queue
- Contratos de dados: `Job`, `JobStatus`, entrada do produtor, saída de confirmação
- Edge cases: job duplicado, broker indisponível, sem consumidor registrado, reprocessamento de DLQ
- 12 critérios de aceitação cobrindo fluxo feliz, falhas e edge cases
- 4 ADRs: fire-and-forget, backoff exponencial, DLQ obrigatória, spec agnóstica de tecnologia
