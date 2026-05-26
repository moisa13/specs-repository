# Exemplo — Conexão aceita com token válido

## Contexto

Cliente inicia uma conexão apresentando um token de autenticação válido.

---

## Entrada

```
Handshake com token: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

---

## Saída esperada

A conexão é aceita. O servidor associa a conexão à identidade do cliente extraída do token (ex: `userId: "usr_abc123"`). A partir deste momento, eventos podem ser entregues ao cliente e o cliente pode enviar eventos ao servidor.

Nenhuma mensagem de confirmação é obrigatória — o canal aberto é a confirmação implícita.

---

## Estado resultante

- Uma nova `Connection` registrada internamente com `connectionId` único e `clientId: "usr_abc123"`
- O cliente passa a receber eventos endereçados a `clientId: "usr_abc123"`
