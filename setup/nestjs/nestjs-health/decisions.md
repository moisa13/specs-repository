# Decisions — NestJS — Health Check (Liveness + Readiness)

> Decisões que podem parecer questionáveis sem contexto — evita que implementadores "melhorem" comportamentos deliberadamente escolhidos.

---

## ADR-001 — Endpoints de health públicos sem autenticação

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Endpoints de health são consumidos por infraestrutura (Kubernetes, load balancers, ferramentas de monitoramento) que normalmente não possuem credenciais de aplicação. Protegê-los exigiria configuração adicional em cada sistema de infraestrutura.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Endpoints públicos (escolhida) | Funciona com qualquer orquestrador sem configuração extra | Expõe informações sobre o estado interno da aplicação |
| Endpoints protegidos por API key | Oculta detalhes internos | Exige configuração de credencial em cada sistema de infraestrutura |
| Endpoint público com resposta reduzida | Limita exposição de detalhes | Reduz a utilidade do endpoint para diagnóstico |

### Decisão
Endpoints públicos. As informações expostas (status de banco e memória) não representam risco de segurança significativo em ambientes típicos de API. Em ambientes com requisitos mais restritivos, proteger com API key e documentar a decisão aqui.

### Consequências
- Qualquer cliente HTTP pode consultar o estado da aplicação sem autenticação
- Orquestradores e load balancers funcionam sem configuração adicional de credenciais

---

## ADR-002 — Separação entre liveness e readiness

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Um único endpoint de health que verifique dependências externas (banco, memória) causa reinicialização em loop do pod pelo Kubernetes quando o banco cai — mesmo que o processo Node.js esteja completamente saudável.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Endpoint único `/health` | Implementação mais simples | Falha no banco causa reinicialização desnecessária do pod |
| Liveness + Readiness separados (escolhida) | Kubernetes toma decisões corretas: remove do balanceamento sem reiniciar | Dois endpoints a manter |

### Decisão
`/health/live` apenas confirma que o processo está vivo; `/health/ready` verifica dependências externas. O Kubernetes usa liveness para decidir reiniciar e readiness para decidir rotear tráfego.

### Consequências
- Falha no banco → pod removido do balanceamento, não reiniciado
- Recuperação do banco → pod reincluído automaticamente no balanceamento
- Processo travado ou em crash loop → liveness falha e Kubernetes reinicia corretamente

---

## ADR-003 — Verificação de memória no readiness, não no liveness

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Verificação de memória poderia estar em liveness (causando reinicialização do pod ao estourar) ou em readiness (removendo do balanceamento). A escolha impacta como o Kubernetes reage a vazamentos de memória.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Memória no liveness | Kubernetes reinicia pod com vazamento | Pod pode entrar em reinicialização contínua se o leak não for corrigido no startup |
| Memória no readiness (escolhida) | Pod para de receber tráfego sem reiniciar — equipe tem tempo para diagnosticar | Processo com memória alta continua rodando sem treinamento |

### Decisão
Verificação de memória no readiness. A reinicialização automática por memória pode mascarar vazamentos e dificultar diagnóstico. Parar de receber tráfego é o comportamento mais seguro e observável.

### Consequências
- Um pod com uso de memória acima do limite para de receber tráfego mas continua acessível para diagnóstico
- Equipe deve monitorar alertas de readiness para identificar vazamentos antes que afetem todos os pods

---

## ADR-004 — `@nestjs/terminus` como biblioteca de health check

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
O ecossistema NestJS tem integração oficial com `@nestjs/terminus`, que já inclui indicadores para TypeORM, memória e HTTP.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Implementação manual | Controle total | Reimplementa o que o Terminus já oferece |
| `@nestjs/terminus` (escolhida) | Integração oficial; indicadores prontos para TypeORM e memória; formato de resposta padronizado | Depende de uma biblioteca adicional |

### Decisão
`@nestjs/terminus` com `TypeOrmHealthIndicator` e `MemoryHealthIndicator`.

### Consequências
- Formato de resposta é o padrão do Terminus (campo `info`, `error`, `details`)
- Novos indicadores podem ser adicionados sem alterar a estrutura do módulo

---

## ADR-005 — `@UseFilters(HealthExceptionFilter)` no controller em vez de modificar o filtro global

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
O `AllExceptionsFilter` do `setup/nestjs/nestjs-init` é registrado globalmente e intercepta toda exceção, incluindo `HealthCheckError` (que o Terminus lança e que extends `HttpException`). Sem tratamento específico, o filtro destrói o corpo Terminus e retorna o formato de erro padrão da aplicação com status 503 mapeado incorretamente para `INTERNAL_SERVER_ERROR`.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Modificar `AllExceptionsFilter` para detectar `HealthCheckError` | Centraliza o tratamento | Acopla uma spec de infraestrutura (`nestjs-init`) a uma de observabilidade; viola o princípio de responsabilidade única do filtro |
| Adicionar 503 ao mapa de status do `AllExceptionsFilter` | Corrige o código de erro | Não preserva o corpo Terminus — o formato de resposta ainda seria destruído |
| `@UseFilters(HealthExceptionFilter)` no controller (escolhida) | Isolado na spec de health; não altera o `AllExceptionsFilter`; filtros de controller têm precedência sobre o global | Requer um arquivo extra |

### Decisão
`HealthExceptionFilter` aplicado via `@UseFilters` no `HealthController`. O filtro captura exclusivamente `HealthCheckError` e devolve `exception.causes` como corpo da resposta 503, preservando o formato Terminus intacto.

### Consequências
- O `AllExceptionsFilter` global não precisa ser alterado
- Exceções não relacionadas ao health check no controller continuam sendo tratadas pelo filtro global normalmente
- Qualquer nova spec que exponha endpoints com formatos de resposta fora do padrão da aplicação deve adotar a mesma estratégia

---

## ADR-006 — Indicador de banco mandatório nesta versão

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
O readiness probe poderia ser configurado apenas com verificação de memória, sem depender de `setup/nestjs/nestjs-database`. Isso tornaria a spec utilizável em projetos sem banco de dados. No entanto, o banco é a dependência externa mais crítica em projetos NestJS com TypeORM — omiti-la do readiness tornaria o endpoint pouco útil para o caso de uso principal.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Indicador de banco opcional | Spec utilizável sem `nestjs-database` | Aumenta a complexidade da spec; exige configuração condicional |
| Indicador de banco mandatório (escolhida) | Spec simples e direta; cobre o caso de uso padrão do repositório | Não utilizável em projetos sem banco sem adaptação |

### Decisão
`setup/nestjs/nestjs-database` é pré-requisito obrigatório nesta versão. O `TypeOrmHealthIndicator` é sempre incluído no readiness probe.

### Consequências
- Projetos sem banco de dados não podem usar esta spec sem modificação
- Caso surja a necessidade, uma variante `setup/nestjs/nestjs-health-minimal` pode ser criada cobrindo apenas liveness e memória, sem dependência de banco

---

## ADR-007 — Indicador Redis condicional baseado em `REDIS_HOST`

**Data:** 2026-05
**Status:** ✅ Aceito

### Contexto
Quando `setup/nestjs/nestjs-queue` é aplicado, o Redis passa a ser uma dependência crítica da aplicação. Um Redis indisponível impede o enfileiramento e consumo de jobs — a aplicação está degradada. O readiness probe deve refletir esse estado para que o orquestrador pare de rotear tráfego.

### Opções consideradas

| Opção | Vantagens | Desvantagens |
|-------|-----------|--------------|
| Indicador Redis sempre presente | Simples; sem condicional | Requer `ioredis` mesmo em projetos sem fila; falha se `REDIS_HOST` não estiver configurado |
| Indicador Redis condicional por `REDIS_HOST` (escolhida) | Funciona com e sem a spec de filas; sem dependência desnecessária | Condicional em runtime no controller |
| Spec separada `nestjs-health-redis` | Máxima separação de responsabilidades | Sobrecarga de spec para um indicador |

### Decisão
O `RedisHealthIndicator` é registrado no `HealthModule` e injetado no controller. O controller verifica `configService.get('REDIS_HOST')` em runtime para incluir ou não o indicador na lista do `/health/ready`. O indicador cria sua própria conexão `ioredis` — não depende nem reutiliza a conexão do BullMQ.

### Consequências
- `ioredis` deve estar instalado para o indicador funcionar — satisfeito automaticamente quando `setup/nestjs/nestjs-queue` está aplicado
- O `RedisHealthIndicator` mantém uma conexão ioredis singleton durante o ciclo de vida da aplicação
- Projetos sem `REDIS_HOST` nunca instanciam a conexão Redis no indicator
