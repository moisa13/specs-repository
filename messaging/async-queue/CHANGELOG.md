# Changelog — Processamento Assíncrono com Fila

> Versões seguem Semantic Versioning: MAJOR.MINOR.PATCH
>
> - **MAJOR** — mudança que quebra implementações existentes
> - **MINOR** — nova regra ou edge case (implementações existentes precisam de revisão)
> - **PATCH** — correção de texto, sem mudança de comportamento

---

## [0.1.0] — 2026-05

### Corrigido
- `behavior.md`: fluxo de retry passo 3 — status durante backoff é `delayed`, não `waiting`; remoção da afirmação incorreta de que `delayed` é exclusivo do delay de produção
- `acceptance.md`: AC-04 atualizado para refletir transição `active → delayed → waiting` durante retry com backoff configurado
- `examples/retry-exhausted-dlq.md`: status das tentativas 1 e 2 corrigido de `waiting` para `delayed`

### Adicionado
- Versão inicial da spec
- Fluxo principal de produção e consumo de jobs
- Fluxo de retry com backoff exponencial
- Comportamento de dead letter queue
- Contratos de dados: `Job`, `JobStatus`, entrada do produtor, saída de confirmação
- Edge cases: job duplicado, broker indisponível, sem consumidor registrado, reprocessamento de DLQ
- 12 critérios de aceitação cobrindo fluxo feliz, falhas e edge cases
- 4 ADRs: fire-and-forget, backoff exponencial, DLQ obrigatória, spec agnóstica de tecnologia
