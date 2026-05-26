# Context — Caching — Cache de Dados

---

## Plataformas suportadas

| Plataforma | Suportada | Observações |
|------------|-----------|-------------|
| API REST | ✅ | Comportamento principal |
| Web SPA | ❌ | |
| Mobile (iOS/Android) | ❌ | |
| Backend (serviço interno) | ✅ | Inclui workers assíncronos |

## Pré-requisitos

- [ ] Redis disponível como backend de cache (banco dedicado, separado de filas e sessões)
- [ ] Estratégia de prefixo de chaves definida para o projeto

## Escopo

**Dentro do escopo:**
- Padrão cache-aside com double-checked locking distribuído
- Namespacing hierárquico de chaves
- TTLs configuráveis com multiplicador global
- Invalidação por chave exata, por padrão de chave e por recurso
- Aquecimento de cache no boot da aplicação
- Desabilitação global de cache via flag de configuração

**Fora do escopo:**
- Notificação em tempo real de eventos de invalidação para clientes → escopo de WebSocket/real-time
- Cache de sessão de usuário → escopo de autenticação
- Cache de respostas HTTP no nível de proxy/CDN → escopo de infraestrutura

## Tecnologia de backend

Esta spec assume Redis como backend de cache. Os conceitos de padrão, namespacing e invalidação são descritos em termos de comportamento; os comandos Redis específicos (SET NX PX, SCAN, Pub/Sub) são detalhados na implementação técnica (`setup/nestjs/nestjs-cache`).
