# Behavior — Canal de Comunicação em Tempo Real

---

## Objetivo

Garantir que clientes autenticados possam estabelecer uma conexão persistente com o servidor, receber eventos entregues pelo servidor e enviar eventos ao servidor, com ciclo de vida bem definido e falhas tratadas de forma explícita.

---

## Fluxo de conexão

1. O cliente inicia a conexão apresentando um token de autenticação
2. O servidor valida o token antes de aceitar a conexão
3. Se o token for inválido ou ausente, a conexão é rejeitada com código `UNAUTHORIZED` e encerrada imediatamente
4. Se o token for válido, a conexão é aceita e associada à identidade do cliente (ex: `userId`)
5. A conexão permanece ativa até que o cliente a encerre ou o servidor a feche explicitamente

---

## Fluxo de entrega — servidor para cliente

1. Uma parte do sistema solicita a entrega de um evento a um cliente identificado
2. O servidor verifica se o cliente tem ao menos uma conexão ativa
3. Se tiver, o evento é entregue a todas as conexões ativas daquele cliente
4. Se não tiver, o evento é descartado silenciosamente — não há buffering nem persistência por padrão

---

## Fluxo de entrega — cliente para servidor

1. O cliente envia um evento com nome e payload
2. O servidor recebe o evento e o despacha para o handler registrado para aquele nome de evento
3. Se não houver handler registrado, o evento é ignorado silenciosamente
4. O handler é responsável por processar o evento — esta spec não define o que ocorre após o despacho

---

## Fluxo de desconexão

1. O cliente encerra a conexão (voluntariamente ou por perda de rede)
2. O servidor detecta o encerramento
3. A conexão é removida do conjunto de conexões ativas do cliente
4. Se o cliente não tiver mais conexões ativas, o registro associado à sua identidade é removido

---

## Regras de negócio

- A autenticação ocorre uma única vez, no handshake — não há revalidação por evento
- Um cliente pode ter múltiplas conexões simultâneas (ex: múltiplas abas ou dispositivos)
- Cada conexão é individualmente identificada, mas agrupada pela identidade do cliente
- A entrega de um evento a um cliente deve atingir todas as suas conexões ativas, não apenas uma
- Eventos têm um nome e um payload — ambos são obrigatórios
- O payload deve ser serializável (JSON)
- O servidor não aguarda confirmação de entrega por padrão — a entrega é fire-and-forget

---

## Edge cases

| Situação | Comportamento esperado |
|----------|----------------------|
| Token ausente no handshake | Conexão rejeitada com `UNAUTHORIZED`; encerrada imediatamente |
| Token expirado no handshake | Conexão rejeitada com `UNAUTHORIZED`; encerrada imediatamente |
| Token expira após a conexão ser aceita | A conexão permanece ativa até ser encerrada pelo cliente ou pelo servidor — esta spec não define revalidação periódica |
| Evento enviado a cliente sem conexão ativa | Evento descartado silenciosamente; nenhum erro é retornado ao chamador |
| Cliente com múltiplas conexões ativas | O evento é entregue a todas as conexões ativas |
| Cliente reenvia o mesmo evento múltiplas vezes | Cada envio é tratado de forma independente; não há deduplicação por padrão |
| Payload não serializável em JSON | O evento é ignorado silenciosamente; a conexão não é encerrada e nenhum erro é retornado ao cliente |
| Perda de rede durante conexão ativa | O servidor detecta o encerramento e executa o fluxo de desconexão; reconexão é responsabilidade do cliente |
| Evento enviado com nome sem handler registrado | O evento é ignorado silenciosamente; a conexão não é encerrada e nenhum erro é retornado ao cliente |

---

## Restrições

- Nunca aceitar uma conexão sem validar o token de autenticação
- Nunca entregar um evento a uma conexão não autenticada
- Nunca persistir ou enfileirar eventos para clientes offline — comportamentos de persistência são definidos pela spec de feature
- Nunca fechar a conexão de um cliente em resposta a um evento malformado — apenas ignorar o evento
- Nunca expor a identidade ou o estado de conexão de um cliente para outro cliente

---

## O que esta spec NÃO cobre

- Qual evento enviar, quando e para quem → responsabilidade da spec de feature
- Salas, canais e broadcast para grupos de clientes → responsabilidade da spec de feature
- Persistência ou enfileiramento de eventos para clientes offline → responsabilidade da spec de feature
- Configuração do protocolo de transporte → ver `setup/nestjs/nestjs-realtime`
- Mecanismo de autenticação e geração de tokens → ver spec de autenticação do projeto
