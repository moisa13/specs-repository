# Integration — NestJS — Inicialização de Projeto

---

## Dependências

Este é o módulo fundacional do boilerplate. Não depende de outros módulos desta spec.

| Módulo | Obrigatório | O que usa |
|--------|-------------|-----------|
| — | — | — |

---

## Dependentes

Todos os módulos de feature adicionados ao projeto dependem da estrutura e dos módulos globais definidos aqui.

| Módulo | Como usa este módulo |
|--------|----------------------|
| `setup/nestjs-database-init` | Parte da estrutura de diretórios e usa `ConfigService` para ler credenciais do banco |
| `auth/` (a criar) | Usa `ConfigService` para segredos JWT; depende do `ValidationPipe` e do filtro global |
| Qualquer módulo de feature | Herda `ConfigService` global, `ValidationPipe` e formato de erro |

---

## Eventos emitidos

Este módulo não emite eventos de domínio.

---

## Eventos consumidos

Este módulo não consome eventos de domínio.

---

## Pacotes a instalar

> Versões mínimas por major. Verificar compatibilidade com a versão do NestJS em uso antes de instalar.

| Pacote | Versão mínima | Finalidade |
|--------|---------------|------------|
| `@nestjs/config` | ^3.0.0 | `ConfigModule` e `ConfigService` |
| `joi` | ^17.0.0 | Validação do schema de variáveis de ambiente |
| `class-validator` | ^0.14.0 | Validação de DTOs via `ValidationPipe` |
| `class-transformer` | ^0.5.0 | Transformação e serialização de tipos nos DTOs |
| `@nestjs/swagger` | ^7.0.0 | Geração da documentação da API |
| `swagger-ui-express` | ^5.0.0 | Interface web do Swagger |

---

## Impacto em outros sistemas

- A estrutura de diretórios definida em `contracts.md` deve ser seguida por todos os módulos adicionados ao projeto
- O formato de resposta de erro definido em `contracts.md` é o contrato com todos os clientes da API — mudá-lo é uma alteração breaking
