# Exemplo — Entrega de evento a cliente conectado

## Contexto

O servidor precisa entregar um evento a um cliente que possui duas conexões ativas (duas abas abertas no browser).

---

## Chamada interna (agnóstica de stack)

```
enviarEvento(
  clientId: "usr_abc123",
  event:    "pedido.atualizado",
  payload:  { pedidoId: "ord_456", status: "em_preparo" }
)
```

---

## Saída esperada

O evento é entregue às duas conexões ativas do cliente `usr_abc123`. Cada conexão recebe:

```json
{
  "event": "pedido.atualizado",
  "payload": {
    "pedidoId": "ord_456",
    "status": "em_preparo"
  }
}
```

Nenhum retorno é dado ao chamador — a entrega é fire-and-forget.

---

## Variação — cliente sem conexão ativa

Se `usr_abc123` não tiver nenhuma conexão ativa no momento da chamada, o evento é descartado silenciosamente. O chamador não recebe erro.
