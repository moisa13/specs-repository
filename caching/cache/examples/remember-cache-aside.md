# Exemplo — Padrão remember (cache-aside)

## Contexto

O `remember` é a forma canônica de consumir cache. Encapsula o padrão cache-aside com double-checked locking distribuído: lê do cache, executa a função produtora apenas em caso de miss, e grava o resultado.

---

## Fluxo

```
remember('users:42', 300, () => db.findUser(42))

  1. Lê 'prefixo:users:42' do Redis
  2. Hit → retorna valor desserializado (fim)
  3. Miss → tenta SET NX PX no lock 'prefixo:lock:users:42'
  4. Lock adquirido → executa db.findUser(42), grava resultado, libera lock
  5. Lock não adquirido → aguarda e relê 'prefixo:users:42'
     5a. Hit → retorna valor (produtor gravou com sucesso)
     5b. Null → executa db.findUser(42) diretamente (produtor caiu antes de gravar)
  6. Retorna valor
```

---

## Com CACHE_ENABLED=false

```
remember('users:42', 300, () => db.findUser(42))

  → executa db.findUser(42) diretamente
  → retorna resultado sem interagir com o Redis
```

---

## Com Redis indisponível

```
remember('users:42', 300, () => db.findUser(42))

  → falha ao ler do Redis
  → executa db.findUser(42) como fallback
  → retorna resultado sem propagar o erro de conexão
```
