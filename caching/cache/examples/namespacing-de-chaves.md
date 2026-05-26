# Exemplo — Namespacing de chaves

## Contexto

As chaves seguem namespacing hierárquico. O prefixo global é aplicado automaticamente pela camada de cache — o caller nunca o inclui.

---

## Padrões de chave

| Recurso | Chave do caller | Chave no Redis (prefixo=`myapp`) |
|---------|----------------|----------------------------------|
| Usuário por ID | `users:42` | `myapp:users:42` |
| Permissões de usuário | `users:42:permissions` | `myapp:users:42:permissions` |
| Lista de produtos com filtros | `products:list:{hash}` | `myapp:products:list:{hash}` |
| Configurações globais | `settings:global` | `myapp:settings:global` |

---

## Hash de parâmetros variáveis

Quando uma listagem aceita filtros ou paginação variáveis, o hash determinístico dos parâmetros compõe a chave:

```
parâmetros: { status: 'active', page: 2, limit: 20 }
hash:        a3f9c2d1  (SHA256 truncado ou similar)
chave:       products:list:a3f9c2d1
```

Garante que cada combinação única de parâmetros tenha sua própria entrada no cache sem tornar a chave legível ilegível com query strings.

---

## Padrão para invalidação

```
deletePattern('users:42:*')   → remove todas as entradas do usuário 42
deletePattern('users:*')      → remove todas as entradas de usuários
deletePattern('products:*')   → remove todo o cache de produtos
```

O prefixo é aplicado automaticamente antes da varredura no Redis.
