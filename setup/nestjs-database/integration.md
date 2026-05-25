# Integration — NestJS — Configuração de Banco de Dados

---

## Dependências

| Módulo | Obrigatório | O que usa |
|--------|-------------|-----------|
| `setup/nestjs-init` | ✅ | `ConfigService` para leitura de todas as variáveis de ambiente de banco |

---

## Dependentes

| Módulo | Como usa este módulo |
|--------|----------------------|
| Qualquer módulo de feature | Usa `TypeOrmModule.forFeature([Entidade])` para registrar suas entidades; a conexão global configurada aqui é usada automaticamente |
| `setup/nestjs-health` | Registra o `TypeOrmHealthIndicator` usando a conexão configurada aqui — não requer alteração no `DatabaseModule` |
| `auth/` (a criar) | Seus módulos de domínio (usuários, sessões) registram entidades via `TypeOrmModule.forFeature` |

---

## Eventos emitidos

Este módulo não emite eventos de domínio.

---

## Eventos consumidos

Este módulo não consome eventos de domínio.

---

## Impacto em outros sistemas

- O glob de entidades `src/**/*.entity.ts` (no `data-source.ts`) e `**/*.entity{.ts,.js}` (no `DatabaseModule`) garantem que entidades adicionadas por qualquer módulo de feature sejam detectadas automaticamente — sem necessidade de alterar o `DatabaseModule`
- A tabela `migrations` criada pelo TypeORM no banco deve ser preservada em todos os ambientes — deletá-la causa perda do histórico de migrations aplicadas
