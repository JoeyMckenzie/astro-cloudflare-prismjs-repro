# Repro for `prismjs` missing from dependencies

Minimal reproduction for `@astrojs/cloudflare` importing `prismjs` at runtime while only listing it in `devDependencies`.

`dist/vite-plugin-prism.js` does `import components from 'prismjs/components.js'`. With pnpm's [global virtual store](https://pnpm.io/settings#enableglobalvirtualstore) enabled, the adapter is linked from the shared store outside the project, so Node can only resolve what the adapter declares. `prismjs` isn't declared, so loading the Astro config fails.

## Reproduce

```sh
pnpm install
pnpm build
```

```
[astro] Unable to load your Astro config

Cannot find package 'prismjs' imported from ~/.local/share/pnpm/store/v11/links/@astrojs/cloudflare/14.3.4/<hash>/node_modules/@astrojs/cloudflare/dist/vite-plugin-prism.js
```

## Control

Set `enableGlobalVirtualStore: false` in `pnpm-workspace.yaml`, delete `node_modules`, then run `pnpm install && pnpm build` again. The build succeeds because `prismjs` gets hoisted into `node_modules/.pnpm/node_modules`, where the adapter can find it by accident.

## Workaround

Declare the dependency on the adapter's behalf in `pnpm-workspace.yaml`:

```yaml
packageExtensions:
  "@astrojs/cloudflare":
    dependencies:
      prismjs: ^1.30.0
```

## Versions

- `@astrojs/cloudflare` 14.3.4
- `astro` 7.3.6
- `wrangler` 4.148.0
