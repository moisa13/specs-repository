# Decisions — NestJS Filas Assíncronas (BullMQ + Redis)

> Decisões que podem parecer questionáveis sem contexto — evita que implementadores "melhorem" comportamentos deliberadamente escolhidos.

---

## ADR-001 — QueueModule centralizado

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Em projetos NestJS sem uma convenção definida, cada módulo de feature tende a registrar suas próprias filas com `BullModule.registerQueue`, criando configurações dispersas, conexões Redis duplicadas e dificuldade para auditar quais filas existem na aplicação.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Cada feature registra suas filas | Autonomia por módulo; menos acoplamento aparente | Configurações dispersas; difícil auditar; conexões duplicadas |
| QueueModule centralizado (escolhida) | Ponto único de verdade; fácil auditoria; uma conexão Redis | Requer disciplina; feature modules devem importar QueueModule |

### Decisão
O `QueueModule` faz o registro canônico de todas as filas — é o único ponto onde as filas são disponibilizadas ao `QueueService`. Módulos de feature que declaram um Processor devem também chamar `BullModule.registerQueue` para a sua fila, pois o `@nestjs/bullmq` exige que o módulo que declara o Processor tenha a fila registrada localmente para que o worker conecte.

### Consequências
- Para adicionar uma fila, o desenvolvedor edita dois arquivos: `queue.module.ts` e `queue.constants.ts`
- O módulo de feature que declara o Processor chama `BullModule.registerQueue` adicionalmente — isso não é duplicação de configuração, é um requisito do framework
- Módulos de feature importam `QueueModule` para ter acesso ao `QueueService`

---

## ADR-002 — QueueService como único ponto de produção

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
O BullMQ permite injetar `Queue` diretamente em qualquer service via `@InjectQueue('nome-da-fila')`. Isso é conveniente mas leva ao mesmo problema de dispersão: qualquer service pode enfileirar jobs sem que haja um contrato centralizado.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| `@InjectQueue` direto nos services | Menos indireção; acesso completo à API BullMQ | Acoplamento direto ao BullMQ; difícil trocar implementação; sem ponto de controle |
| `QueueService` facade (escolhida) | Ponto único de controle; isola o BullMQ; validação centralizada | Uma camada extra de indireção |

### Decisão
`@InjectQueue` só existe em `queue.service.ts`. Services de domínio chamam `QueueService.add()`.

### Consequências
- Validações (fila registrada, payload serializável) ficam em um único lugar
- Trocar BullMQ por outra biblioteca exige mudanças apenas no `QueueService` e no `QueueModule`

---

## ADR-003 — BullMQ em vez de Bull v3

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
O ecossistema NestJS historicamente usa `bull` (v3) via `@nestjs/bull`. O `bullmq` é a versão v4+ com reescrita completa e suporte nativo a TypeScript.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| `bull` (v3) via `@nestjs/bull` | Mais maduro; mais exemplos online | Sem suporte ativo; sem TypeScript nativo; sem grupos de filas |
| `bullmq` via `@nestjs/bullmq` (escolhida) | TypeScript nativo; suporte ativo; melhor API; grupos de filas | Menor base de exemplos para NestJS |

### Decisão
Usar `@nestjs/bullmq` com `bullmq`. O pacote `@nestjs/bull` não deve ser instalado no mesmo projeto.

### Consequências
- Processors usam o padrão `WorkerHost`: a classe estende `WorkerHost` e implementa `process(job: Job)`, despachando pelo `job.name` — não há decorator `@Process` por método
- O pacote `@nestjs/bull` não deve ser instalado no mesmo projeto; os decorators dos dois pacotes são incompatíveis

---

## ADR-004 — Processors nos módulos de feature, não no QueueModule

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Processors poderiam ser declarados no `QueueModule` para centralizar tudo. Mas um Processor contém lógica de negócio e depende de services da feature — declarar no `QueueModule` criaria acoplamento invertido.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Processors no QueueModule | Tudo de fila em um lugar | QueueModule dependeria de todos os módulos de feature; ciclo de dependência |
| Processors nos módulos de feature (escolhida) | Cada feature cuida do próprio Processor; sem ciclo de dependência | Processors dispersos, mas organizados por feature |

### Decisão
Processors ficam nos módulos de feature em `src/[feature]/processors/`. O módulo de feature importa `QueueModule` (não o contrário).

### Consequências
- Para saber quais Processors existem, é preciso buscar nos módulos de feature
- A relação fila → Processor é documentada no `queue.constants.ts` via comentários ou em um mapeamento separado

---

## ADR-005 — removeOnComplete ativo por padrão

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Por padrão, o BullMQ mantém jobs completados no Redis indefinidamente. Em aplicações com alto volume, isso causa crescimento descontrolado de memória.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Manter jobs completados | Útil para auditoria | Crescimento ilimitado de memória Redis |
| Remover após conclusão (escolhida) | Redis limpo; sem acúmulo | Perde histórico de execução |

### Decisão
`removeOnComplete: { count: 100 }` como padrão global — mantém os últimos 100 jobs completados por fila para debugging, remove o restante.

### Consequências
- Auditoria de jobs completados deve ser feita via logs ou banco de dados, não via BullMQ
- O valor `100` pode ser sobrescrito por fila no `QueueModule` quando necessário

---

## ADR-006 — QueueModule importado no AppModule quando Bull Board está habilitado

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
O `QueueModule` não é global — a regra padrão é que cada feature module o importe quando precisar. Porém, o Bull Board é uma rota HTTP transversal (`/queues`) que precisa estar disponível sempre que a aplicação estiver rodando, independente de quais feature modules estão carregados.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Feature module importa QueueModule (padrão) | Fiel ao design de módulos isolados | Dashboard só existe se algum feature module estiver carregado |
| AppModule importa QueueModule (escolhida) | Dashboard sempre disponível; Redis conecta na inicialização | Leve desvio do princípio de isolamento por feature |

### Decisão
Quando o Bull Board está habilitado, o `AppModule` importa o `QueueModule`. Feature modules continuam importando o `QueueModule` normalmente para registrar seus Processors — a importação no `AppModule` não substitui a importação nos feature modules.

### Consequências
- A conexão Redis é estabelecida na inicialização da aplicação, não na primeira fila usada
- O dashboard `/queues` fica disponível desde o boot, independente das features carregadas
- Feature modules ainda precisam importar `QueueModule` para usar `QueueService` e registrar Processors
