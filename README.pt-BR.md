# `astro sync` sobrescreve as dependências otimizadas do servidor de desenvolvimento em execução

[English](README.md) · **Português**

Reprodução mínima: com o `astro dev` rodando, `astro sync`, `astro check` e `astro build` regravam `node_modules/.vite/deps` com dependências otimizadas para produção.

## Passos

```sh
npm install
npm run dev
```

Num segundo terminal, na raiz do projeto:

```sh
# 1. The dev cache holds React's development runtime: prints 0
grep -c "jsxDEV = void 0" node_modules/.vite/deps/react_jsx-dev-runtime.js

# 2. Run any of these while the dev server is still running
npx astro sync    # or: npx astro check / npx astro build

# 3. The dev cache now holds React's production runtime: prints 1
grep -c "jsxDEV = void 0" node_modules/.vite/deps/react_jsx-dev-runtime.js

# 4. The running dev server serves it for URLs it had not cached yet: prints 1
curl -s http://localhost:4321/node_modules/.vite/deps/react_jsx-dev-runtime.js | grep -c "jsxDEV = void 0"
```

O que cada passo mostra: (1) o cache de desenvolvimento tem a versão de desenvolvimento do React e o comando imprime 0; (2) qualquer um desses comandos roda com o servidor ainda aberto; (3) o cache passa a ter a versão de produção do React e o comando imprime 1; (4) o servidor aberto entrega essa versão para endereços que ainda não tinha em cache, e o comando imprime 1.

## Esperado

`astro sync`, `astro check` e `astro build` não mexem no cache de dependências do servidor de desenvolvimento em execução.

## Atual

O cache é substituído (o log mostra `Re-optimizing dependencies because vite config has changed`). Os módulos que o servidor carrega dele depois disso misturam o React de produção e o de desenvolvimento, o que aparece nas ilhas como `TypeError: _jsxDEV is not a function` ou `Invalid hook call`.
