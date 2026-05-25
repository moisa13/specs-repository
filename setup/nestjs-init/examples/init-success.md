# Exemplo — Inicialização bem-sucedida

## Contexto

Projeto NestJS recém-criado com todas as variáveis de ambiente configuradas corretamente. O objetivo é verificar que a aplicação sobe, o Swagger está disponível em `/docs` e o `ValidationPipe` rejeita campos extras.

---

## Entrada (variáveis de ambiente em `.env`)

```dotenv
NODE_ENV=development
PORT=3000
CORS_ORIGIN=http://localhost:5173
APP_NAME=minha-api
APP_DESCRIPTION=API do minha-api
```

## Comportamento esperado — Inicialização

```
[NestFactory] Starting Nest application...
[InstanceLoader] AppModule dependencies initialized
[NestApplication] Nest application successfully started +Xms
Application running on: http://localhost:3000
Swagger available at: http://localhost:3000/docs
```

## Verificação — GET /docs

```
HTTP/1.1 200 OK
Content-Type: text/html
```

Interface do Swagger renderizada no browser com os endpoints disponíveis.

---

## Verificação — POST em endpoint com campo extra

**Request:**
```json
POST /example
Content-Type: application/json

{
  "name": "Moiséis",
  "email": "moiseisalmeida2@gmail.com",
  "role": "admin"
}
```

**Resposta esperada:**
```json
HTTP/1.1 400 Bad Request

{
  "statusCode": 400,
  "error": "BAD_REQUEST",
  "message": "Validation failed",
  "details": [
    { "field": "role", "message": "property role should not exist" }
  ]
}
```

---

## Notas

- `role` foi rejeitado porque não está declarado no DTO — comportamento esperado do `ValidationPipe` com `forbidNonWhitelisted: true`
- Em produção (`NODE_ENV=production`), `GET /docs` retorna `404`
