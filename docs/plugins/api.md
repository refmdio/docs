# Plugin API Reference

This page summarises the host-facing APIs available to RefMD plugins: effect payloads, renderer contracts, and the browser host object. RefMD uses [Extism](https://extism.org) so that plugin backends can be written in many languages—Rust is showcased in examples, but any language that compiles to WASI is fair game. Use this reference together with the [Plugin Development Guide](development.md).

## Effect Catalogue

Return effects from the Extism backend (`ExecOutput.effects`). Recognised effects are applied server-side; unknown effects are forwarded to the frontend bundle, allowing custom handling.

### Document & Data Effects

| `type`          | Required fields                            | Description |
|-----------------|--------------------------------------------|-------------|
| `createDocument`| `title`, optional `docType`, optional `parentId` | Creates a document owned by the invoking user. The resulting document ID is made available to subsequent effects in the same response (for example `putKv`). |
| `putKv`         | `key`, `value`, optional `docId`, optional `scope` (`doc` default) | Stores arbitrary JSON in the plugin KV store. `docId` defaults to the document created earlier in the response. |
| `getKv`         | _Not supported as an effect_; use the REST API instead. |
| `createRecord`  | `docId`, `kind`, `data`                     | Inserts a structured record into `plugin_records`. |
| `updateRecord`  | `recordId`, `patch`                         | Partially updates a record (JSON merge). |
| `deleteRecord`  | `recordId`                                  | Deletes a record. |

### Host Interaction Effects

| `type`     | Fields                  | Description |
|------------|-------------------------|-------------|
| `navigate` | `to`                     | Triggers a client-side navigation. Use `:createdDocId` as a placeholder for the ID returned by `createDocument` in the same response. |
| `log`      | `message`, optional `level` (`info` default) | Emits a structured log entry on the server (useful for debugging). |
| `showToast`| `message`, optional `level` (`info`, `success`, `warning`, `error`) | Not handled server-side; forwarded to the frontend bundle for display, as demonstrated by `sample-plugin`. |

Unknown effect types are copied verbatim into the JSON response and delivered to the frontend (if present). Use this channel for custom client-side behaviours.

## Renderer Contract

Plugins can register Markdown renderers via the `renderers` array in `plugin.json`:

```json
"renderers": [
  {
    "kind": "mermaid",
    "function": "render",
    "hydrate": { "module": "assets/hydrate.js", "export": "default" }
  }
]
```

- `kind`: lower-case identifier matched against placeholders produced by RefMD’s Markdown pipeline.
- `function`: Extism export name (defaults to `render` when omitted).
- `hydrate.module`: optional browser module used to hydrate the rendered HTML. The path must stay inside the plugin archive (no absolute or `..` segments).
- `hydrate.export`: optional named export, defaulting to the module’s default export.
- `hydrate.etag`: optional string for cache busting.

The backend `render` function receives a JSON payload describing the placeholder, and must return:

```rust
#[derive(Serialize, Default)]
struct RenderOutput {
    ok: bool,
    html: Option<String>,
    warnings: Option<Vec<String>>,
    error: Option<String>,
}
```

- Set `ok` to `true` when rendering succeeds.
- Provide `html` containing the final markup to insert into the document.
- Use `warnings` or `error` to surface diagnostic messages (they appear in server/client logs).

See `mermaid-plugin/backend/src/lib.rs` for a complete example.

## Frontend Host Object

When a frontend bundle is loaded, RefMD calls `mount(container, host)`. The `host` object exposes:

### Core Methods

| Member | Description |
|--------|-------------|
| `host.exec(action, payload?)` | Invokes the backend `exec` function for `action`. Returns `{ ok, data, effects, error }`. Any effects included are already applied server-side; unrecognised ones should be handled client-side. |
| `host.toast(level, message)` | Displays a toast using the native UI (`level` one of `info`, `success`, `warning`, `error`). |
| `host.origin` | Base origin for the API endpoints (useful when building absolute URLs). |

### REST Helpers (`host.api`)

| Method | Purpose |
|--------|---------|
| `renderMarkdown(text, options?)` | Calls the server-side Markdown renderer and returns `{ html }`. |
| `renderMarkdownMany(items)` | Batch renders multiple snippets (if enabled on the host). |
| `listRecords(pluginId, docId, kind, token?)` | Fetches plugin records. |
| `createRecord(pluginId, docId, kind, data, token?)` | Creates a record. |
| `patchRecord(pluginId, id, patch)` | Partially updates a record. |
| `deleteRecord(pluginId, id)` | Deletes a record. |
| `getKv(pluginId, docId, key, token?)` | Retrieves a KV entry. |
| `putKv(pluginId, docId, key, value, token?)` | Stores a KV entry. |
| `uploadFile(docId, file)` | Reuses RefMD’s file upload pipeline. |

### UI Helpers (`host.ui`)

| Method | Description |
|--------|-------------|
| `hydrateAll(root)` | Upgrades RefMD-specific web components (attachments, wiki links, etc.) within `root`. |
| `hydrateAttachments(root)` / `hydrateWikiLinks(root)` | Narrower hydration helpers used by built-in features. |

### Context

`host.context` provides:

- `docId`: current document ID (if available).
- `route`: current route string.
- `token`: share token if the document is accessed via a share.
- `mode`: `'primary'` or `'secondary'` depending on whether the plugin is controlling the main view or a secondary pane.

Use this information to tailor the frontend experience or request additional data from the backend.

## REST Endpoints

All plugin REST endpoints live under `/api/plugins/...` or `/api/me/plugins/...`. Generated TypeScript clients can be produced with `npm run gen:api` in `refmd/app`.

Key endpoints for plugins:

- `POST /api/plugins/:plugin/exec/:action`
- `GET/POST /api/plugins/:plugin/docs/:doc_id/records/:kind`
- `PATCH/DELETE /api/plugins/:plugin/records/:id`
- `GET/PUT /api/plugins/:plugin/docs/:doc_id/kv/:key`
- `GET /api/me/plugins/manifest`

Refer to the backend’s OpenAPI specification (`refmd/api/openapi/openapi.json`) for full schemas.

---

Combining these primitives—effects for state changes, renderers for Markdown output, and the host object for optional UI—you can build plugins that range from simple render hooks to fully interactive extensions.
