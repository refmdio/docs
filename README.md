# RefMD Documentation Site

This directory contains the Markdown sources consumed by [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/).

## Local Preview

Run the following from the repository root:

```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r requirements.txt
mkdocs serve
```

Open <http://localhost:8000> to view the site. MkDocs live-reloads whenever you edit files.

## Publishing on GitHub Pages

1. Commit the `mkdocs.yml`, `requirements.txt`, and `docs/` changes to your default branch.
2. Configure **Settings ▸ Pages** with `Source: GitHub Actions` (recommended) or run `mkdocs gh-deploy` from CI.
3. Use the [official MkDocs Material workflow example](https://squidfunk.github.io/mkdocs-material/publishing-your-site/#github-pages) to automate builds.

## Contributing Content

- Add new pages inside `docs/` and update the navigation in `mkdocs.yml`.
- Use standard Markdown plus MkDocs extensions (admonitions, definition lists, etc.).
- Keep examples synced with the source code in this repository.
