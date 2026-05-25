# Context — NestJS — Inicialização de Projeto

---

## Plataformas suportadas

| Plataforma | Suportada | Observações |
|------------|-----------|-------------|
| API REST | ✅ | Plataforma alvo desta spec |
| Web SPA | ❌ | |
| Mobile (iOS/Android) | ❌ | |
| Backend (serviço interno) | ✅ | Swagger pode ser omitido se a API não for exposta externamente |

---

## Pré-requisitos

- [ ] Node.js >= 20
- [ ] pnpm instalado globalmente (`npm install -g pnpm`)
- [ ] NestJS CLI instalado globalmente (`pnpm add -g @nestjs/cli`)
- [ ] Git inicializado no projeto

---

## Escopo

**Dentro do escopo:**
- Criação do projeto com o NestJS CLI
- Estrutura de diretórios obrigatória
- Instalação das dependências base via pnpm
- Configuração do `ConfigModule` com validação de variáveis de ambiente via Joi
- `ValidationPipe` global com `whitelist` e `forbidNonWhitelisted`
- Filtro de exceção global que normaliza todas as respostas de erro
- Swagger habilitado apenas em `NODE_ENV=development`
- Configuração de CORS via variável de ambiente

**Fora do escopo:**
- Configuração de banco de dados → ver `setup/nestjs-database-init`
- Autenticação e autorização → ver `auth/` (a criar)
- Logging estruturado → ver `observability/logging` (a criar)
- Testes automatizados → ver `setup/testing` (a criar)
- Containerização → ver `setup/docker` (a criar)

---

## Variações por plataforma

### Backend (serviço interno)
Swagger pode ser omitido. CORS restrito ou desabilitado dependendo do contexto de rede. O restante da configuração é idêntico.

---

## Considerações de ambiente

- **Desenvolvimento:** Swagger sempre habilitado em `/docs`. Variáveis carregadas de `.env`.
- **Produção:** Swagger não montado. Variáveis obrigatórias validadas na inicialização — a aplicação deve recusar subir se alguma estiver ausente ou com formato inválido.
- **Test:** `NODE_ENV=test` desabilita comportamentos dependentes de ambiente (ex: Swagger). Variáveis de teste carregadas de `.env.test`.
