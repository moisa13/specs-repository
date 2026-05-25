# Exemplo — Inicialização bloqueada por variável ausente

## Contexto

Tentativa de subir a aplicação sem a variável `PORT` definida no `.env`. A aplicação deve encerrar antes de abrir qualquer porta, com mensagem descritiva do problema.

---

## Entrada (variáveis de ambiente em `.env`)

```dotenv
NODE_ENV=development
CORS_ORIGIN=http://localhost:5173
# PORT está ausente
```

## Comportamento esperado — stderr

```
Error: Config validation error: "PORT" is required
    at Object.<anonymous> (...)
```

Processo encerra com **exit code 1** imediatamente após a falha de validação do Joi, antes de qualquer módulo ser carregado.

---

## Notas

- A mesma falha ocorre para `NODE_ENV` ou `CORS_ORIGIN` ausentes
- Para formato inválido (ex: `PORT=abc`), a mensagem é: `"PORT" must be a number`
- Nenhuma porta é aberta — a aplicação não chega a escutar requisições
