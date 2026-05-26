# Exemplo — Envio de evento do cliente para o servidor

## Contexto

Cliente com conexão ativa envia um evento ao servidor. O servidor despacha para o handler registrado para aquele nome de evento.

---

## Entrada (cliente → servidor)

```json
{
  "event": "cursor.movido",
  "payload": { "x": 120, "y": 340 }
}
```

---

## Saída esperada

O servidor identifica o handler registrado para `"cursor.movido"` e o invoca com o payload recebido. Nenhuma resposta é enviada de volta ao cliente por padrão — o handler decide se e o que retornar.

---

## Variação — sem handler registrado para o nome do evento

```json
{
  "event": "evento.desconhecido",
  "payload": { "foo": "bar" }
}
```

O servidor não encontra handler para `"evento.desconhecido"`. O evento é ignorado silenciosamente. A conexão permanece ativa e nenhum erro é retornado ao cliente.

---

## Variação — payload não serializável em JSON

O cliente tenta enviar um evento cujo payload não pode ser representado como JSON (ex: referência circular, função, binário não codificado). O evento é ignorado silenciosamente. A conexão permanece ativa e nenhum erro é retornado ao cliente.
