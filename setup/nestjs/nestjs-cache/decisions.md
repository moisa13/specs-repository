# Decisions — Setup NestJS — Cache

> Decisões que podem parecer questionáveis sem contexto — evita que implementadores "melhorem" comportamentos deliberadamente escolhidos.

---

## ADR-001 — ioredis diretamente em vez de @nestjs/cache-manager

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
O NestJS oferece `@nestjs/cache-manager` com suporte a Redis via adaptador. É necessário avaliar se ele atende aos requisitos de double-checked locking, SCAN para deletePattern e Pub/Sub para eventos de invalidação.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| `@nestjs/cache-manager` | Integração nativa com NestJS; decorators automáticos | Abstração não expõe SET NX PX, SCAN nem Pub/Sub; double-checked locking não implementável |
| ioredis diretamente — **escolhida** | Acesso completo à API do Redis; SET NX PX, SCAN e Pub/Sub nativos | Sem decorators automáticos; mais código de integração |

### Decisão
Usar ioredis diretamente, encapsulado no `CacheService`. Os decorators do `@nestjs/cache-manager` não são utilizados nesta spec.

### Consequências
- Controle total sobre todas as operações Redis necessárias (locking, SCAN, Pub/Sub)
- Sem cache automático por decorator em rotas — todo cache é explícito via `CacheService` ou `GenericCacheService`

---

## ADR-002 — CacheModule decorado com @Global()

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
`CacheService` precisa ser injetável em qualquer módulo de domínio e em filtros/guards globais sem que cada módulo declare `CacheModule` em seus imports.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Sem `@Global()` — importar onde necessário | Dependências explícitas | Boilerplate em todos os módulos de domínio |
| `@Global()` — **escolhida** | Disponível em toda a aplicação com uma única importação | Dependência de `CacheService` fica implícita nos módulos de feature |

### Decisão
`CacheModule` é decorado com `@Global()`. É importado uma única vez no `AppModule`.

### Consequências
- `CacheService` injetável em qualquer classe sem declarar `CacheModule` nos imports do módulo

---

## ADR-003 — GenericCacheService como classe abstrata, não interface

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Os módulos de domínio precisam de uma forma padronizada de implementar cache dos seus recursos. A escolha é entre interface (contrato apenas) e classe abstrata (contrato + implementação compartilhada).

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Interface | Contrato explícito; implementação livre | Cada módulo repete a lógica de hashing, prefixo e delegação para `CacheService` |
| Classe abstrata — **escolhida** | Implementação compartilhada de `rememberById`, `rememberList`, `invalidateById`, `invalidateAll` | Acoplamento à hierarquia de herança |

### Decisão
`GenericCacheService` é uma classe abstrata. Módulos de domínio a estendem e injetam o `CacheService` pelo construtor da superclasse.

### Consequências
- Lógica de hashing de parâmetros e construção de chaves fica centralizada
- Módulos de domínio definem apenas os métodos específicos dos seus recursos, delegando as operações base para a superclasse

---

## ADR-004 — CacheWarmer como interface com registro via token de injeção

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
O `CacheWarmingService` precisa descobrir quais recursos aquecer sem depender diretamente de cada módulo de domínio — isso criaria acoplamento circular.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Lista hardcoded no CacheWarmingService | Simples | Acoplamento direto; CacheWarmingService precisaria importar módulos de domínio |
| Token de injeção com array de warmers — **escolhida** | Desacoplado; cada módulo registra seu warmer sem que CacheWarmingService o conheça | Requer padrão de registro via `useClass` / `multi: true` ou similar |

### Decisão
`CacheWarmer` é uma interface. Módulos de domínio registram seus warmers no `AppModule` via token `CACHE_WARMERS`. O `CacheWarmingService` injeta o array e itera.

### Consequências
- `CacheWarmingService` não tem dependências de módulos de domínio
- Novos warmers são adicionados sem modificar `CacheWarmingService`
