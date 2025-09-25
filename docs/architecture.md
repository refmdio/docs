# System Architecture

RefMD is split into a Rust backend, a React frontend, and a realtime bridge powered by Yjs. Plugins extend both sides through Extism on the server and ESM modules in the browser. The sections below capture how these layers interact.

## High-level Components

- **API (`refmd/api`)** — Rust 2024 crate built with Axum, SQLx, and Tokio. It owns authentication, document CRUD, file storage, Git sync, Markdown rendering, and plugin execution.
- **App (`refmd/app`)** — React 19 + Vite single-page app. Routing is handled by TanStack Router, state by TanStack Query, and the editor by Monaco. Tailwind CSS provides styling primitives.
- **Realtime hub** — Implemented in `refmd/api/src/infrastructure/realtime`. Each document gets a Yjs room bridged over WebSockets; presence and text updates broadcast through this hub.
- **Plugins** — Server-side Extism WASM modules plus frontend ESM bundles. Files live under `plugins/` after installation and load dynamically at runtime.

```
Browser (React + Monaco) ──> Axum API (Rust)
          ▲                         │
          │                         ▼
       Plugins ◄──── WebSocket hub ─┘
```

## Backend Anatomy

- **Routing**: `main.rs` builds the Axum router, wires middleware (CORS, tracing, static assets), and exposes OpenAPI docs via `utoipa`.
- **Persistence**: PostgreSQL is the primary store. `sqlx` repositories live under `refmd/api/src/infrastructure/db/repositories`. Migrations are tracked in `refmd/api/migrations`.
- **Markdown**: The `/api/markdown/render` endpoint uses `comrak` + `ammonia` for rendering and sanitisation, then `syntect` for syntax highlighting.
- **Plugins**: `ExecutePluginAction` orchestrates Extism calls and applies server-side effects (`createDocument`, `putKv`, `createRecord`, `updateRecord`, `deleteRecord`, `navigate`, `log`). Plugin assets are managed by `FilesystemPluginStore`, while installations are tracked per user via the plugin installation repository.
- **Git sync**: `refmd/api/src/application/use_cases/git/*` wraps the `git2` crate with a vendored libgit2 build so users can init, sync, and inspect diffs from the UI.
- **Sharing**: `/api/shares` issues scoped tokens for documents or folders, tracks usage counts, and enforces permissions (`view`, `edit`, `admin`) via `access::require_view` / `resolve_document`.
- **Publishing**: `/api/public` toggles document publication status, exposes `/public/:owner/:id` for read-only pages, and feeds the public profile listings consumed by the frontend.
- **Files**: Uploads live on disk (configurable path). `GET /api/uploads/*` streams files through `serve_upload`, which re-validates JWT/share tokens before reading from storage; `/api/files/:id` falls back to authenticated downloads when you need the original metadata.
- **Security**: JWT-based auth with `argon2` password hashing. Share tokens allow read-only access to specific documents.

## Frontend Anatomy

- **Composition**: The app is built with Vite. `src/routes` defines TanStack routes (e.g. `/document/$id`, `/plugins`), and `src/features` encapsulates domain-specific UI.
- **Editor**: `refmd/app/src/features/edit-document` contains editor code. `MarkdownEditor.tsx` mounts Monaco lazily, `EditorLayout.tsx` handles responsive splits, and `Toolbar.tsx` wires formatting actions.
- **Preview pipeline**: `ui/Markdown.tsx` calls `MarkdownService.renderMarkdown`, then runs `upgradeAll` from `entities/document/wc` to hydrate attachments, wiki links, and other custom elements. Scroll synchronisation is handled via `useScrollSync`, while uploads and Vim mode live in `useEditorUploads` and dynamic `monaco-vim` imports.
- **API client**: OpenAPI definitions are generated from the backend via `npm run gen:api`. The resulting typed fetchers live in `src/shared/api/client`.
- **State**: TanStack Query caches API responses (documents, shares, plugins) while lightweight context providers manage editor-specific state.
- **Plugins**: `features/plugins/lib/runtime.ts` constructs the host object passed to plugin modules. It resolves frontend bundles from `/api/plugin-assets`, applies client-side effects (`navigate`, `showToast`), and exposes helpers for Markdown rendering, records, KV, and uploads.

## Realtime Flow

1. The browser opens a WebSocket to `/api/yjs/:id` and attaches JWT or share credentials.
2. The WebSocket handler resolves access via `access::resolve_document` (using the `AccessRepository` and share rules) before handing the socket to the realtime hub.
3. Document content travels as Yjs updates; file uploads and metadata changes continue through REST endpoints.
4. Awareness updates carry cursor colour and label data, allowing Monaco to render collaborator cursors.

## Deployment Notes

- `docker-compose.yml` launches the API, frontend, PostgreSQL, and supporting services. Use `docker compose -f docker-compose.dev.yml up --build` for local development builds.
- The API exposes `/health` for readiness probes and serves Swagger UI at `/api/docs` (backed by `/api/openapi.json`).
- Documentation is now published with [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/); see `README.md` in this directory for build instructions.

Next, explore the [Plugin Platform](plugins/index.md) for details on extending the system.
