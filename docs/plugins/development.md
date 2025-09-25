# Plugin Development Guide

RefMD plugins are **backend-first**. Start by compiling an Extism WASI module that can answer host invocations; add a browser bundle only when you need UI beyond Markdown renderers and built-in affordances. This guide focuses on the recommended workflow. For concrete code see the full-stack [`sample-plugin/`](../../sample-plugin/) and the backend-only Markdown integration in [`mermaid-plugin/`](../../mermaid-plugin/).

> 💡 **Why Extism?** Extism lets plugin authors choose their favourite language (Rust, Go, JavaScript/TypeScript, TinyGo, Python, and more) as long as it can compile to WASI. Rust examples are used here because the RefMD core team maintains them, but you can start from any language runtime listed on [extism.org](https://extism.org).

## Prerequisites

- Rust stable with the `wasm32-wasip1` target (`rustup target add wasm32-wasip1`).
- Node.js 20+ (only if you plan to ship a frontend bundle).
- Extism CLI (optional) for local WASM smoke tests.
- A running RefMD stack (`docker compose up -d`).

## Recommended Workflow

1. Create a Rust crate under `backend/` and expose `exec` (and optionally `render`) via `#[plugin_fn]` exports.
2. Design the actions your plugin responds to and the **effects** each action returns. Effects tell the host what to do; most plugins never touch the database or HTTP APIs directly.
3. Install the ZIP into a local RefMD instance, exercise the actions, and watch the API logs for errors.
4. Add a frontend only if you need custom UI (dashboards, tool panels, etc.). Markdown-only plugins can stop here.

## Backend Essentials

`backend/Cargo.toml` template:

```toml
[dependencies]
extism-pdk = "1"
serde = { version = "1", features = ["derive"] }
serde_json = "1"
htmlescape = "0.3" # Handy when escaping Markdown renderer output
```

Minimal `exec` entry point:

```rust
use extism_pdk::*;
use serde::{Deserialize, Serialize};

#[derive(Deserialize)]
struct ExecInput {
    action: String,
    payload: serde_json::Value,
    ctx: serde_json::Value,
}

#[derive(Serialize, Default)]
struct ExecOutput {
    ok: bool,
    data: Option<serde_json::Value>,
    effects: Vec<serde_json::Value>,
    error: Option<serde_json::Value>,
}

#[plugin_fn]
pub fn exec(input: Json<ExecInput>) -> FnResult<Json<ExecOutput>> {
    let req = input.0;
    let mut out = ExecOutput::default();

    match req.action.as_str() {
        "my-plugin.create" => {
            out.ok = true;
            out.effects.push(serde_json::json!({
                "type": "createDocument",
                "title": "New doc from plugin",
            }));
            out.effects.push(serde_json::json!({
                "type": "putKv",
                "key": "meta",
                "value": { "createdBy": "my-plugin" }
            }));
        }
        _ => {
            out.error = Some(serde_json::json!({
                "code": "UNKNOWN_ACTION",
                "action": req.action,
            }));
        }
    }

    Ok(Json(out))
}
```

Build with:

```bash
cd backend
cargo build --release --target wasm32-wasip1
cp target/wasm32-wasip1/release/<crate>.wasm backend/plugin.wasm
```

## Working with Markdown Renderers

Expose a `render` function when you want to influence Markdown output. The Mermaid plugin renders Mermaid code blocks entirely from the backend:

```rust
#[derive(Serialize, Default)]
struct RenderOutput {
    ok: bool,
    html: Option<String>,
    warnings: Option<Vec<String>>,
    error: Option<String>,
}

#[plugin_fn]
pub fn render(input: Json<serde_json::Value>) -> FnResult<Json<RenderOutput>> {
    let code = input.0.get("code").and_then(|v| v.as_str()).unwrap_or("");
    let mut out = RenderOutput::default();
    out.ok = true;
    out.html = Some(format!(
        "<pre class=\"refmd-mermaid-code\"><code class=\"language-mermaid\">{}</code></pre>",
        htmlescape::encode_minimal(code)
    ));
    Ok(Json(out))
}
```

Declare the renderer in `plugin.json`:

```json
"renderers": [
  {
    "kind": "mermaid",
    "function": "render",
    "hydrate": { "module": "assets/hydrate.js" }
  }
]
```

`hydrate.module` lets you ship a lightweight browser module (for example to load Mermaid JS and draw the final SVG). If Markdown enhancements are sufficient, you can skip a full frontend entirely.

## Optional Frontend

Create a frontend only when you need richer UI:

1. Initialise the project:
   ```bash
   cd frontend
   npm install
   npm install @refmdio/plugin-sdk
   ```
2. Export `mount(container, host)` and use SDK helpers (`createKit`, `markdownPreview`, etc.) to integrate with the host UI.
3. Implement lifecycle hooks when needed:
   - `exec(action, { host, payload })`
   - `canOpen(docId, ctx)` / `getRoute(docId, ctx)`
4. Build the bundle (the sample plugin uses esbuild) and point `plugin.json.frontend.entry` at the output file (for example `dist/index.mjs`).

## Packaging Checklist

Your ZIP **must** mirror the paths referenced in `plugin.json`. A minimal layout looks like:

```
plugin.zip
├── plugin.json
├── backend/
│   └── plugin.wasm
├── index.mjs            # if frontend.entry = "index.mjs"
└── assets/
    └── hydrate.js      # optional hydration module
```

1. Copy the compiled WASM module into `backend/plugin.wasm`.
2. Copy or rename your frontend bundle so that it matches `frontend.entry` (the sample plugin renames `frontend/dist/index.mjs` to `index.mjs` at the archive root).
3. Add any additional assets (for example `assets/hydrate.js`) and ensure all paths are relative—no leading `/` or `..`.
4. Create the archive:
   ```bash
   (cd dist && zip -r ../my-plugin.zip .)
   ```
   Here `dist` represents the directory containing the final layout shown above.

See `sample-plugin/.github/workflows/build-plugin.yml` for a CI script that performs these steps automatically.

## Installing & Validating

1. Open **Plugins** (`/plugins`) in RefMD and install from the ZIP URL.
2. Ensure commands defined under `ui.toolbar` appear, and that any renderers influence Markdown as expected.
3. Call `/api/me/plugins/manifest` to confirm the manifest was parsed correctly.
4. Watch `docker compose logs -f api` for Extism runtime output during testing.

## Debugging Tips

- Emit `log` effects to inspect intermediate values during backend execution.
- Renderer errors leave placeholders untouched; return `warnings` or `error` strings to bubble details into the logs.
- When a frontend is present, monitor the browser console and `/api/plugins/.../exec/...` network calls for failures.

Focus on the backend first—most RefMD plugins ship successfully without any custom JavaScript. Layer UI enhancements only when the use case demands it.
