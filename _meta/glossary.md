# Glossário

Termos utilizados nas specs deste repositório. Quando um termo aparecer nas specs, seu significado é este — independente do projeto ou stack.

---

## Termos gerais

**Spec (Especificação)**
Documento que descreve o comportamento esperado de um módulo. É a fonte de verdade — a implementação deve se conformar à spec, não o contrário.

**Domínio**
Agrupamento de módulos relacionados por área funcional. Define a pasta de primeiro nível no repositório. Exemplos:

| Domínio | Módulos típicos |
|---------|-----------------|
| `auth` | login, logout, session, mfa, oauth2 |
| `audit` | event-log, change-tracking |
| `rbac` | roles, permissions |
| `notifications` | email, push, in-app |
| `common` | pagination, soft-delete, error-handling |

Regras para nomear um domínio: use substantivos no singular, em inglês, sem espaços (use hífen se necessário). Prefira nomes funcionais (`auth`) a técnicos (`jwt`) ou genéricos (`utils`).

**Módulo**
Unidade de funcionalidade autônoma com spec própria, sempre dentro de um domínio. Formato: `[domínio]/[módulo]`. Ex: `auth/login`, `audit/event-log`.

Regras para nomear um módulo: mesmas do domínio — substantivos no singular, em inglês, sem espaços (use hífen se necessário). Prefira nomes funcionais e descritivos (`event-log`) a genéricos (`log`) ou técnicos (`kafka-consumer`).

**Fluxo principal**
O caminho feliz — a sequência de passos quando tudo ocorre conforme o esperado.

**Edge case**
Situação fora do fluxo principal que requer comportamento específico.

**ADR (Architecture Decision Record)**
Registro de uma decisão arquitetural: o contexto, as opções consideradas, a decisão tomada e o motivo. Documentado em `decisions.md`.

**Agnóstico de stack**
Comportamento descrito sem dependência de linguagem, framework ou banco de dados específico.

---

## Termos de contratos

**Entidade**
Estrutura de dados com campos definidos. Descrita em `contracts.md`.

**Campo obrigatório**
Campo que deve estar presente e não-nulo em toda operação.

**Campo opcional**
Campo que pode estar ausente. Sua ausência não deve causar erro.

**Normalização**
Transformação aplicada a um valor antes de processá-lo. Ex: e-mail convertido para lowercase.

---

## Termos de plataforma

**API REST**
Sistema que expõe comportamento via endpoints HTTP. Respostas em JSON.

**Web SPA**
Aplicação web de página única rodando no navegador.

**Mobile**
Aplicação nativa ou híbrida para iOS e/ou Android.

**Backend**
Serviço que processa lógica de negócio e persiste dados.

---

## Tipos agnósticos de dados

| Tipo agnóstico | Equivalentes comuns |
|----------------|---------------------|
| string | string, varchar, str |
| integer | int, number (sem decimal) |
| float | number, decimal, float |
| boolean | bool, boolean |
| datetime | timestamp, Date, datetime |
| uuid | UUID, GUID, string com formato UUID |
| object | map, dict, JSON object |
| array | list, array, [] |
| nullable | null, None, nil, undefined |

---

## Status de spec

| Status | Significado |
|--------|-------------|
| 🚧 template | Estrutura criada, conteúdo a preencher |
| 📝 rascunho | Em elaboração, não usar em produção |
| ✅ pronto | Revisado e aprovado para uso |
| ⚠️ desatualizado | Implementações existentes podem estar desalinhadas |
| 🗄️ depreciado | Substituído por outro módulo, não usar em novos projetos |
