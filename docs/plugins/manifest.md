# Manifest Specification

Every plugin ships with a `plugin.json` file at the root of its distribution ZIP. The manifest controls how RefMD loads, routes, and authorises the plugin.

## Minimal Example

```json
{
  "id": "sample",
  "name": "Sample Plugin",
  "version": "0.1.2",
  "abi": "refmd-plugin/v1",
  "mounts": ["/sample/*"],
  "frontend": { "entry": "index.mjs", "mode": "esm" },
  "backend": { "wasm": "backend/plugin.wasm" },
  "permissions": ["doc.read", "doc.write"],
  "ui": {
    "toolbar": [{ "title": "New Sample Document", "action": "sample.create" }],
    "fileTree": {
      "icon": "FileText",
      "identify": { "type": "kvFlag", "key": "meta", "path": "isSample", "equals": true }
    }
  }
}
```

## Field Reference

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | ✓ | Stable, URL-friendly identifier (`^[A-Za-z0-9_-]+$`). Used for REST routes, storage directories, and UI grouping. |
| `name` | string | – | Human-readable label shown in the plugin gallery. |
| `version` | string | ✓ | Semantic or build version. Determines cache-busting and storage paths (`plugins/<scope>/<id>/<version>/`). |
| `abi` | string | – | Declares the expected ABI for the Extism module. Current backend uses `refmd-plugin/v1` for compatibility checks. |
| `mounts` | string[] | – | Route globs the frontend should own. Defaults to `/<id>/*` when omitted. |
| `frontend.entry` | string | ✓ (if frontend present) | Relative path (within the archive) to the ESM bundle. Resolved via `/api/plugin-assets/...`. |
| `frontend.mode` | string | – | Only `"esm"` is currently supported. Lowercase `esm` is assumed when omitted. |
| `backend.wasm` | string | ✓ (if backend present) | Relative path to the Extism WASM module. Defaults to `backend/plugin.wasm` if missing. |
| `permissions` | string[] | – | Requested host capabilities. The host surfaces them for display today; common values are `doc.read` and `doc.write`. |
| `commands` | string[] | – | Reserved for future use. The host currently ignores this field and instead derives actionable commands from `ui.toolbar`. |
| `config` | object | – | Arbitrary JSON blob stored alongside the manifest. Useful for static plugin configuration. |
| `ui.toolbar` | array | – | Adds buttons to the Plugins page or command palette. Each item accepts `title` and `action`. |
| `ui.fileTree` | object | – | Controls file-tree decorations: `icon` (Lucide icon name) and optional `identify` rules. `kvFlag` rules check plugin KV (scope `doc`) before attaching the icon. |
| `author` | string | – | Attribution shown in the UI. |
| `repository` | string | – | URL to the source repository or documentation. |
| `renderers` | array | – | Declares Markdown placeholder renderers. Each item needs a lower-cased `kind`, optional `function` (Extism export name, defaults to `render`), and optional `hydrate` spec. |

### Renderer Items

Renderer entries bridge server-side placeholder rendering and client-side hydration:

- `kind` — matches the placeholder `kind` emitted by the Markdown service.
- `function` — Extism export used for server-side rendering (`render` when omitted).
- `hydrate.module` — Relative module path (for example `frontend/hydrate.js`). Must stay within the archive—no absolute or parent-directory references.
- `hydrate.export` — Optional named export (defaults to the module default export).
- `hydrate.etag` — Optional string used to bust hydration caches.

### UI Identify Rules

`ui.fileTree.identify` currently supports the `kvFlag` strategy:

```json
{
  "type": "kvFlag",
  "key": "meta",
  "path": "isSample",
  "equals": true
}
```

- Uses the plugin ID, `doc` scope, and `docId` to resolve KV storage.
- Reads the JSON blob under `key` and optional dot-delimited `path`.
- Compares the value to `equals` (strict equality). Matching documents receive the plugin’s icon.

## Packaging Expectations

A plugin archive is a ZIP with this structure:

```
plugin.zip
├── plugin.json
├── backend/
│   └── plugin.wasm
└── frontend/
    └── dist/
        └── index.mjs
```

- Paths inside the archive must be relative; no absolute or parent-directory escapes.
- The host extracts archives into `plugins/global/<id>/<version>/` (for pre-installed plugins) or `plugins/<user-id>/<id>/<version>/` (user installs).
- Additional assets (CSS, hydration modules, etc.) are allowed as long as they live within the archive and are referenced using relative paths.

## Manifest Distribution

- Global manifests are read from disk when RefMD boots and merged into `/api/me/plugins/manifest`.
- User-specific manifests are loaded after installation (`InstallPluginFromUrl` stores the archive and manifest per user).
- The frontend sorts manifests by scope (user before global) and plugin ID.

Install your ZIP in a local RefMD stack and open the Plugins page to confirm parsing. The host logs manifest issues to the server console.
