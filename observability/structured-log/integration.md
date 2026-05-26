# Integration — Registro Estruturado de Logs

---

## Dependências

| Módulo | Obrigatório | O que usa |
|--------|-------------|-----------|
| `observability/correlation-id` | ❌ | Fornece o `correlationId` incluído no campo opcional de `LogEntry`; sem este módulo, o campo é simplesmente omitido |

---

## Dependentes

| Módulo | Como usa este módulo |
|--------|----------------------|
| `setup/nestjs/nestjs-logging` | Implementa todos os comportamentos desta spec usando NestJS, Winston e nest-cls |

---

## Eventos emitidos

Esta spec não define eventos de domínio. O log em si é o artefato produzido.

---

## Eventos consumidos

Esta spec não consome eventos de outros módulos.

---

## Impacto em outros sistemas

- Qualquer módulo que captura e registra exceções (ex: exception filters) deve seguir as regras de classificação por faixa HTTP definidas aqui
- Qualquer módulo que acessa dados sensíveis de entrada deve garantir que esses dados passem pela sanitização antes de serem logados
