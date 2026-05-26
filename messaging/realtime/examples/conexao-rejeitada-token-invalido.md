# Exemplo — Conexão rejeitada por token inválido

## Contexto

Cliente tenta iniciar uma conexão com um token expirado, adulterado ou ausente.

---

## Entrada

```
Handshake sem token  (ou com token: "token.invalido.aqui")
```

---

## Saída esperada

A conexão é rejeitada antes de ser estabelecida. O servidor retorna o código de erro `UNAUTHORIZED` e encerra a tentativa de conexão imediatamente.

---

## Estado resultante

- Nenhuma `Connection` é criada
- Nenhum evento pode ser entregue ao cliente
- O cliente deve tratar o erro e decidir se tenta reconectar com um token renovado
