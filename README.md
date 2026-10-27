# `astro sync` overwrites the running dev server's optimized deps

**English** · [Português](README.pt-BR.md)

Minimal reproduction: while `astro dev` is running, `astro sync`, `astro check` and `astro build` rewrite `node_modules/.vite/deps` with dependencies optimized for production.

## Steps

```sh
npm install
npm run dev
```

In a second terminal, from the project root:

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

## Expected

`astro sync`, `astro check` and `astro build` leave the running dev server's dependency cache untouched.

## Actual

The cache is replaced (the log shows `Re-optimizing dependencies because vite config has changed`). Modules the dev server loads from it afterwards mix production and development React, which surfaces in islands as `TypeError: _jsxDEV is not a function` or `Invalid hook call`.
