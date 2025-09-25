# Plugin Platform Overview

RefMD plugins combine an Extism-powered backend with a browser-based frontend. The host loads plugin manifests at runtime, exposes APIs for storage and rendering, and routes matching URLs into plugin-controlled views.

## Runtime at a Glance

1. **Manifest discovery** — The API aggregates manifests from global (`plugins/global/*`) and per-user (`plugins/<user-id>/*`) directories via `/api/me/plugins/manifest`.
2. **Frontend loading** — The React app resolves `frontend.entry` to an ESM module and imports it dynamically (see `features/plugins/lib/runtime.ts`).
3. **Backend execution** — Plugin actions call into Extism WASM modules. Results can return `effects` that the host applies on the server (document CRUD, records, KV) or in the client (`navigate`, `showToast`).
4. **Routing** — `mounts` and optional `canOpen` / `getRoute` exports let plugins take over specific routes and documents.

## Key Concepts

- **Host object**: When a plugin module runs, it receives a host with `exec`, `api`, `ui`, and `context` helpers. This is the primary bridge back into RefMD.
- **Scopes**: Records and KV storage currently use the `doc` scope. Future revisions may add user- or workspace-level scopes.
- **Effects**: Extism backends return JSON effects. Recognised server-side effects include `createDocument`, `putKv`, `createRecord`, `updateRecord`, `deleteRecord`, `navigate`, and `log`. Unknown effects are forwarded to the frontend module for client-side handling.
- **UI affordances**: Plugins can inject commands into the toolbar/command palette and attach icons to documents in the file tree via the `ui` manifest section.

## Reference Material

- [Manifest Specification](manifest.md) details every field in `plugin.json` and how the host interprets it.
- [Development Guide](development.md) walks through building, packaging, and installing a plugin using the sample repository.

The [`sample-plugin`](https://github.com/refmdio/sample-plugin) repository doubles as a living example and is referenced throughout the guides below.
