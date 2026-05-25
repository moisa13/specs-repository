# Context — NestJS — Configuração de Banco de Dados

---

## Plataformas suportadas

| Plataforma | Suportada | Observações |
|------------|-----------|-------------|
| API REST | ✅ | Plataforma alvo desta spec |
| Web SPA | ❌ | |
| Mobile (iOS/Android) | ❌ | |
| Backend (serviço interno) | ✅ | Mesma configuração; ajustar `DB_SSL` conforme a rede interna |

---

## Pré-requisitos

- [ ] `setup/nestjs-init` aplicado — `ConfigService` global disponível
- [ ] PostgreSQL acessível (em desenvolvimento: container Docker via `docker run` ou `docker compose`)
- [ ] Node.js >= 20
- [ ] pnpm instalado

---

## Escopo

**Dentro do escopo:**
- Instalação dos pacotes TypeORM e driver PostgreSQL
- Variáveis de ambiente necessárias para a conexão
- `DatabaseModule` com `TypeOrmModule.forRootAsync` lendo configurações do `ConfigService`
- Arquivo `data-source.ts` para uso exclusivo do TypeORM CLI
- Estrutura de diretórios para migrations
- Scripts npm para operações de migration
- Comportamento de execução automática de migrations em desenvolvimento
- Comportamento de execução manual de migrations em produção

**Fora do escopo:**
- Entidades e repositórios → responsabilidade de cada spec de feature
- Health check de banco → ver `setup/nestjs-health`
- Seeding de dados → não coberto por esta spec
- Multi-tenancy ou múltiplas conexões → não coberto por esta spec
- Configuração de testes com banco → ver `setup/testing` (a criar)
- Containerização do banco → ver `setup/docker` (a criar)

---

## Considerações de ambiente

- **Desenvolvimento:** migrations executadas automaticamente no bootstrap. PostgreSQL rodando via Docker.
- **Produção:** migrations executadas manualmente antes do deploy (`pnpm migration:run`). `migrationsRun: false`. SSL habilitável via `DB_SSL=true`.
- **Test:** migrations não executadas automaticamente. Gerenciamento de estado de banco definido pela spec `setup/testing` (a criar).
