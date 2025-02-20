# `swc-plugin-cache-repro`

Standalone repro for an issue with SWC and concurrent acess to the plugin WASM bytecode cache.

## Repro

This fails reliably on my machine:

```bash
➜  rm -rf .swc && rm -rf build && useq 0 499 | xargs -P 25 -I{} ./bin/swc-darwin-arm64 compile --config-json '{"jsc":{"experimental":{ "cacheRoot": ".swc", "plugins":[["./node_modules/@swc/plugin-formatjs", { }]]}}}' --out-file ./build/Component{}.js src/Component{}.tsx
xargs: ./bin/swc-darwin-arm64: terminated with signal 10; aborting
```

And this succeeds:

```bash
# Pre-populate the plugin bytecode cache
➜  rm -rf .swc && ./bin/swc-darwin-arm64 compile --config-json '{"jsc":{"experimental":{ "cacheRoot": ".swc", "plugins":[["./node_modules/@swc/plugin-formatjs", { }]]}}}' --out-file /dev/null /dev/null

# Then we can run many actions in parallel successfully (and fast!)
➜  rm -rf build && seq 0 499 | xargs -P 25 -I{} ./bin/swc-darwin-arm64 compile --config-json '{"jsc":{"experimental":{ "cacheRoot": ".swc", "plugins":[["./node_modules/@swc/plugin-formatjs", { }]]}}}' --out-file ./build/Component{}.js src/Component{}.tsx
```

## Notes

Trying to watch the filesystem access to the `.swc` cache directory while the failing case is running: 

```
➜  fswatch -x .swc
.swc Created IsDir AttributeModified
.swc/plugins Created IsDir AttributeModified
.swc/plugins/v7_macos_aarch64_7.0.0 Created IsDir AttributeModified
.swc/plugins/v7_macos_aarch64_7.0.0/73d7dfbc154ced25d4c677fe2162e5aa0280fb947e891799719e0f23d786d2dc Created AttributeModified IsFile Updated Removed AttributeModified
.swc/plugins/v7_macos_aarch64_7.0.0/73d7dfbc154ced25d4c677fe2162e5aa0280fb947e891799719e0f23d786d2dc Created IsFile Updated Removed AttributeModified
.swc/plugins/v7_macos_aarch64_7.0.0/73d7dfbc154ced25d4c677fe2162e5aa0280fb947e891799719e0f23d786d2dc Created AttributeModified IsFile Updated AttributeModified
```
