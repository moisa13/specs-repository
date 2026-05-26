# Acceptance Criteria — Canal de Comunicação em Tempo Real

> Formato: **Dado** [contexto] **Quando** [ação] **Então** [resultado esperado]

---

## Fluxo principal

**AC-01 — Conexão aceita com token válido**
- **Dado** um cliente com token de autenticação válido
- **Quando** o cliente inicia a conexão apresentando o token
- **Então** a conexão é aceita, associada à identidade do cliente e mantida ativa

**AC-02 — Entrega de evento a cliente conectado**
- **Dado** um cliente com ao menos uma conexão ativa
- **Quando** o servidor emite um evento endereçado àquele cliente
- **Então** o evento é entregue a todas as conexões ativas do cliente

**AC-03 — Recebimento de evento do cliente**
- **Dado** um cliente com conexão ativa e um handler registrado para o nome do evento
- **Quando** o cliente envia um evento com nome e payload válidos
- **Então** o servidor despacha o evento para o handler correspondente

---

## Cenários de falha

**AC-04 — Conexão rejeitada por token ausente**
- **Dado** um cliente sem token de autenticação
- **Quando** o cliente tenta iniciar a conexão
- **Então** a conexão é rejeitada com código `UNAUTHORIZED` e encerrada imediatamente

**AC-05 — Conexão rejeitada por token inválido ou expirado**
- **Dado** um cliente com token inválido ou expirado
- **Quando** o cliente tenta iniciar a conexão
- **Então** a conexão é rejeitada com código `UNAUTHORIZED` e encerrada imediatamente

---

## Edge cases

**AC-06 — Evento descartado para cliente sem conexão**
- **Dado** um cliente sem nenhuma conexão ativa no momento
- **Quando** o servidor tenta emitir um evento para aquele cliente
- **Então** o evento é descartado silenciosamente; nenhum erro é retornado ao chamador

**AC-07 — Entrega a múltiplas conexões do mesmo cliente**
- **Dado** um cliente com duas ou mais conexões ativas simultâneas
- **Quando** o servidor emite um evento endereçado àquele cliente
- **Então** o evento é entregue a todas as conexões ativas, não apenas a uma

**AC-08 — Desconexão limpa o estado**
- **Dado** um cliente com conexão ativa
- **Quando** a conexão é encerrada (pelo cliente ou por perda de rede)
- **Então** a conexão é removida do conjunto de conexões ativas do cliente sem efeitos colaterais

**AC-09 — Evento com payload inválido não encerra a conexão**
- **Dado** um cliente com conexão ativa
- **Quando** o cliente envia um evento com payload não serializável em JSON
- **Então** o evento é ignorado silenciosamente; a conexão permanece ativa e nenhum erro é retornado ao cliente

**AC-10 — Evento sem handler é ignorado silenciosamente**
- **Dado** um cliente com conexão ativa
- **Quando** o cliente envia um evento cujo nome não tem handler registrado no servidor
- **Então** o evento é ignorado; a conexão não é encerrada e nenhum erro é retornado ao cliente

---

## O que caracteriza uma implementação incorreta

- Aceitar uma conexão sem validar o token de autenticação
- Entregar um evento apenas à conexão mais recente do cliente, ignorando as demais
- Encerrar a conexão ao receber um evento malformado
- Persistir ou enfileirar eventos para clientes offline sem que uma spec de feature tenha definido esse comportamento
- Expor, a um cliente, informações sobre outros clientes conectados

---

## Cobertura mínima de testes

- [ ] Conexão aceita com token válido (AC-01)
- [ ] Conexão rejeitada com token ausente (AC-04)
- [ ] Conexão rejeitada com token inválido (AC-05)
- [ ] Evento entregue a cliente com conexão ativa (AC-02)
- [ ] Evento entregue a todas as conexões simultâneas do mesmo cliente (AC-07)
- [ ] Evento descartado para cliente sem conexão (AC-06)
- [ ] Evento do cliente despachado para handler registrado (AC-03)
- [ ] Evento com payload inválido não encerra a conexão (AC-09)
- [ ] Evento sem handler registrado ignorado silenciosamente (AC-10)
- [ ] Desconexão limpa o estado sem efeitos colaterais (AC-08)
