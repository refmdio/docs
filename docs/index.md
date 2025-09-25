# RefMD Documentation

RefMD is a real-time, collaborative Markdown workspace that teams can extend through a plugin runtime. These docs describe how to use the editor, understand the architecture, and build extensions on top of RefMD.

## Guideposts

- [Quick Start](quick-start.md) shows how to deploy RefMD with docker-compose and expose it through a reverse proxy.
- [Feature Overview](feature-overview.md) catalogues the major capabilities available to workspace users and administrators.
- [System Architecture](architecture.md) shows how the Rust backend, Vite-powered frontend, realtime bridge, and plugin runtime fit together.
- [Plugin Platform](plugins/index.md) gathers the manifest reference and development workflow for building Extism-powered plugins.

## Conventions

- Examples assume the monorepo layout where the main application lives under `refmd/`.
- API payloads use JSON; code snippets are TypeScript or Rust depending on the layer they describe.
- When you see the term *host*, it refers to the RefMD application that loads and runs plugins.

!!! tip "Need to explore the code?"
    The [`sample-plugin`](https://github.com/refmdio/sample-plugin) repository contains a complete reference implementation. The [`@refmdio/plugin-sdk`](https://github.com/refmdio/plugin-sdk) package provides the client utilities reused in examples throughout these docs.

## Getting Help

- File bugs or feature requests in the [RefMD GitHub repository](https://github.com/refmdio/refmd).
- Join the discussion threads in the repository for roadmap updates and community plugin ideas.
