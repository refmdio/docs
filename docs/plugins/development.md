# Plugin Development Guide

Follow this guide to build, package, and install a RefMD plugin. The [`sample-plugin/`](../../sample-plugin/) repository is the canonical reference; adapt it as you iterate.

## Prerequisites

- **Rust** stable with the `wasm32-wasip1` target (`rustup target add wasm32-wasip1`).
- **Node.js** 20 or later with npm.
- **Extism CLI (optional)** if you want to invoke the WASM module locally.
- A running RefMD stack (`docker compose up -d` from the repo root).

## Project Layout

```
my-plugin/
├── backend/              # Rust crate compiled to wasm32-wasip1
│   └── src/lib.rs        # Extism entry point (`exec`)
├── frontend/             # ESM bundle built with @refmdio/plugin-sdk
│   ├── src/index.ts
│   └── dist/index.mjs    # Output consumed by RefMD
├── plugin.json           # Manifest (see Manifest Specification)
└── README.md
```

Keep build scripts and the manifest alongside these directories so CI can package the archive easily.

## Backend (Extism)

1. Create a Rust crate under `backend/` and add `extism-pdk` plus `serde` dependencies.
2. Implement an `exec` function annotated with `#[plugin_fn]`. It receives JSON input `{ action, payload, ctx }` and must return `{ ok, data, effects, error }`.
3. Emit **effects** to ask the host to perform work. The backend currently understands:
   - `createDocument`
   - `putKv`
   - `createRecord`
   - `updateRecord`
   - `deleteRecord`
   - `navigate`
   - `log`

   Any other effects are forwarded to the frontend module for client-side handling (for example `showToast`).
4. Validate required fields (document IDs, record IDs) and return structured errors (`{ "code": "BAD_REQUEST" }`) so the host can prompt the user.
5. Compile the crate:

```bash
cd backend
cargo build --release --target wasm32-wasip1
```

Copy `target/wasm32-wasip1/release/<crate>.wasm` to `backend/plugin.wasm` before packaging.

## Frontend (ESM)

1. Scaffold a TypeScript project under `frontend/` and install the SDK:

```bash
cd frontend
npm install
npm install @refmdio/plugin-sdk
```

2. Use `@refmdio/plugin-sdk` helpers to render UI and talk to the host:

```ts
import { createKit, resolveDocId } from '@refmdio/plugin-sdk'

export async function mount(container: Element, host: any) {
  const kit = createKit(host)
  const state = { docId: host?.context?.docId || resolveDocId(), message: '' }

  kit.store({
    container,
    initialState: state,
    render: (current, set) => kit.card({
      title: 'My Plugin',
      body: kit.fragment(
        kit.input({ value: current.message, onInput: (v) => set({ message: v }) }),
        kit.button({
          label: 'Say hello',
          onClick: () => host.exec('my-plugin.hello', { docId: current.docId }),
        }),
      ),
    }),
  })
}
```

3. Implement optional lifecycle exports:
   - `exec(action, { host, payload })` — run purely in the browser when the host invokes a command.
   - `canOpen(docId, ctx)` / `getRoute(docId, ctx)` — claim ownership of certain documents and reroute `/document/:id` to your custom route.

4. Build the bundle (the sample project uses esbuild):

```bash
npm run build
# Produces frontend/dist/index.mjs
```

Match the output path to the manifest entry (for example `"frontend": { "entry": "dist/index.mjs" }`).

## Host APIs

The host object injected into frontend modules exposes:

- `host.exec(action, payload?)` — call the backend Extism action.
- `host.api.renderMarkdown(text, options?)` — reuse the server renderer.
- `host.api.listRecords(pluginId, docId, kind)` — manage structured plugin records.
- `host.api.createRecord/patchRecord/deleteRecord` — CRUD helpers for records.
- `host.api.getKv/putKv` — scoped key/value storage.
- `host.api.uploadFile(docId, file)` — reuse the standard upload pipeline.
- `host.ui.hydrateAll(element)` — upgrade markdown placeholders (attachments, wiki links).
- `host.toast(level, message)` — surface feedback through the native toast component.
- `host.context` — contains `docId`, `route`, `token`, and `mode` (`primary` or `secondary`).

## Working with Records and KV

- Records live in the `plugin_records` table and map to `/api/plugins/:plugin/docs/:docId/records/:kind`.
- KV entries live in `plugin_kv` and are keyed by `(plugin, scope, scope_id, key)`. Use scope `doc` for document-specific metadata.
- The backend automatically persists `createRecord`, `updateRecord`, `deleteRecord`, and `putKv` effects.

## Packaging

1. Ensure the build artefacts exist:
   - `backend/plugin.wasm`
   - `frontend/dist/index.mjs`
   - `plugin.json`
2. Confirm `plugin.json` uses paths that exist inside the archive (for example `"frontend": { "entry": "dist/index.mjs" }`).
3. Create a ZIP with relative paths only:

```bash
zip -r my-plugin.zip plugin.json backend frontend/dist/index.mjs
```

Upload `my-plugin.zip` to a location RefMD can reach (GitHub Release asset, S3 bucket, etc.).

## Installing in RefMD

1. Sign in to RefMD and open **Plugins** (`/plugins`).
2. Choose **Install from URL** and paste the direct link to your ZIP.
3. After installation the manifest appears in the list. Toolbar commands (from `ui.toolbar`) become available immediately.
4. Use `ui.fileTree.identify` or custom routes to surface documents created by the plugin.

## Debugging Tips

- Watch the browser console for module import or hydration errors. The host logs issues with resolving `frontend.entry`.
- Server-side errors from Extism are logged by the API process. Run `docker compose logs -f api` during development.
- Inspect `/api/me/plugins/manifest` to view the merged manifest the client receives.
- If a command returns `error.code = "BAD_REQUEST"` mentioning `docId`, the host prompts the user to pick a document and retries.

Once your plugin is stable, publish a release so others can install it directly from your repository.
