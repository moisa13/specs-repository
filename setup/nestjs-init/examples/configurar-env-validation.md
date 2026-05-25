# Exemplo — Configurar schema de validação de variáveis de ambiente

## Contexto

Schema Joi que valida todas as variáveis de ambiente obrigatórias na inicialização. A aplicação encerra com exit code 1 se qualquer variável estiver ausente ou com formato inválido.

Caminho do arquivo: `src/config/env.validation.ts`

---

## Código

```typescript
import * as Joi from 'joi';

export const envValidationSchema = Joi.object({
  NODE_ENV: Joi.string()
    .valid('development', 'production', 'test')
    .required(),
  PORT: Joi.number().integer().min(1).max(65535).required(),
  CORS_ORIGIN: Joi.string().required(),
  APP_NAME: Joi.string().required(),
  APP_DESCRIPTION: Joi.string().required(),
});
```

---

## Saída — inicialização com todas as variáveis presentes

Nenhum erro. A aplicação continua o bootstrap normalmente.

## Saída — variável ausente

```
Error: Config validation error: "PORT" is required
```

Processo encerra com **exit code 1** antes de qualquer módulo ser carregado.

## Saída — variável com formato inválido

```
Error: Config validation error: "PORT" must be a number
```

---

## Notas

- Novas variáveis de ambiente adicionadas ao projeto devem ser acrescentadas aqui antes de serem usadas
- A validação é executada pelo `ConfigModule` no momento do `forRoot()` — ver `configurar-app-module.md`
