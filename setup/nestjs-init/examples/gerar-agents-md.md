# Exemplo — Gerar agents.md

## Contexto

O `agents.md` é criado na raiz do projeto durante a inicialização. Registra convenções, padrões e decisões em linguagem direta para agentes. Cada spec aplicada ao projeto acrescenta sua própria seção.

Caminho do arquivo: `agents.md`

---

## Conteúdo inicial

```markdown
# agents.md — <nome-do-projeto>

Contexto para agentes que trabalham neste projeto. Cada seção foi adicionada pela spec que a originou.

---

## Estrutura do projeto (setup/nestjs-init)

- Todos os módulos da aplicação — de infraestrutura ou de domínio — vivem em `src/modules/`
- Diretórios transversais ficam fora de `src/modules/`: `src/common/`, `src/config/`, `src/database/`
- O `AppModule` não define controllers, providers ou services diretamente — apenas importa módulos

## Convenções de código (setup/nestjs-init)

- Não usar `any` como tipo de retorno em controllers, services ou filtros
- Não expor stack traces em respostas de erro
- Não registrar `ValidationPipe` ou filtros de exceção em módulos individuais — apenas globalmente no bootstrap
- O filtro de exceção global é registrado **antes** do `ValidationPipe` no bootstrap

## Variáveis de ambiente (setup/nestjs-init)

- Toda variável nova deve ser adicionada ao schema Joi em `src/config/env.validation.ts` e ao `.env.example`
- Variáveis obrigatórias usam `.required()`; opcionais usam `.default(valor)`
- O `.env` nunca é commitado — apenas o `.env.example` com chaves e descrições

## Formato de resposta de erro (setup/nestjs-init)

Todas as respostas de erro seguem este formato:

\`\`\`json
{
  "statusCode": 400,
  "error": "BAD_REQUEST",
  "message": "Mensagem legível",
  "details": []
}
\`\`\`

O campo `error` é sempre SNAKE_CASE maiúsculo. Ver mapeamento completo em `setup/nestjs-init/contracts.md`.

## Swagger (setup/nestjs-init)

- Disponível apenas quando `NODE_ENV !== 'production'`, na rota `/docs`
- Título vem de `APP_NAME`, descrição de `APP_DESCRIPTION`, versão do `package.json`
- Todo controller deve ter `@ApiTags`; todo endpoint deve ter `@ApiOperation` e os decorators de resposta correspondentes
```

---

## Notas

- Substituir `<nome-do-projeto>` pelo nome real do projeto
- Manter o atributo de autoria de cada seção (`setup/nestjs-init`, `setup/nestjs-database`, etc.) para rastrear qual spec originou cada convenção
- Ao aplicar uma nova spec, acrescentar a seção correspondente ao final do arquivo — nunca remover seções existentes sem justificativa
