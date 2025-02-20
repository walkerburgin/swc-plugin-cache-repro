# `swc-plugin-cache-repro`

Standalone repro for an issue with SWC and concurrent acess to the plugin WASM bytecode cache.

## Repro

This fails reliably on my machine:

```bash
➜  rm -rf .swc && rm -rf build && seq 0 499 | xargs -P 25 -I{} ./bin/swc-darwin-arm64 compile --config-json '{"jsc":{"experimental":{ "cacheRoot": ".swc", "plugins":[["./node_modules/@swc/plugin-formatjs", { }]]}}}' --out-file ./build/Component{}.js src/Component{}.tsx
xargs: ./bin/swc-darwin-arm64: terminated with signal 10; aborting
```

And this succeeds:

```bash
# Pre-populate the plugin bytecode cache
➜  rm -rf .swc && ./bin/swc-darwin-arm64 compile --config-json '{"jsc":{"experimental":{ "cacheRoot": ".swc", "plugins":[["./node_modules/@swc/plugin-formatjs", { }]]}}}' --out-file /dev/null /dev/null

# Then we can run many actions in parallel successfully (and fast!)
➜  rm -rf build && seq 0 499 | xargs -P 25 -I{} ./bin/swc-darwin-arm64 compile --config-json '{"jsc":{"experimental":{ "cacheRoot": ".swc", "plugins":[["./node_modules/@swc/plugin-formatjs", { }]]}}}' --out-file ./build/Component{}.js src/Component{}.tsx
```
