# Editor Guide

The RefMD editor combines Monaco, Yjs, and custom UI so teams can co-author Markdown documents in real time. This page highlights the key capabilities available by default.

## Layout and View Modes

- **Split view** (default) shows Monaco beside the rendered preview. Switch between `editor`, `preview`, and `split` modes from the toolbar; layouts stack vertically on small screens.
- **Preview scroll sync** keeps the rendered pane aligned with the caret. Toggle it from the toolbar when you want the panes to move independently.
- **Floating tool palette** anchors to the lower-right corner, exposing formatting, uploads, and utilities without obstructing the writing surface.

## Live Collaboration

- Every document is backed by a shared `Y.Doc` and an `awareness` channel (see `refmd/app/src/features/edit-document/hooks/useMonacoBinding.ts` and `useAwarenessStyles.ts`). Cursor colours and selections update as collaborators move through the file.
- Monaco is bound to Yjs through `useMonacoBinding`, so edits from all participants merge into a single CRDT state without conflicts.
- Optional Vim keybindings ship via the `monaco-vim` integration. The toggle lives in the toolbar and stores the preference in `localStorage`.

## Markdown Tooling

- Formatting actions are wired through `useMarkdownCommands` and operate on the current Monaco selection. Commands cover bold, italic, strikethrough, headings, ordered/unordered lists, task lists, quotes, code, tables, horizontal rules, and links.
- Task list checkboxes in the preview stay interactive. Toggling a checkbox rewrites the corresponding `- [ ]` line and syncs the change across clients (`handleTaskToggle` in `Editor.tsx`).
- `monaco-markdown` loads on demand to provide GFM-aware behaviours for lists, tables, and fenced code blocks.
- Wiki link auto-completion (`[[Document]]`) is registered through `registerWikiLinkCompletion`, making internal linking fast without breaking the typing flow.

## Preview and Hydration

- The preview pane renders via the backend Markdown pipeline (`MarkdownService.renderMarkdown`). Once the HTML arrives, `host.ui.hydrateAll` upgrades custom elements such as attachment viewers and wiki-link badges.
- Scroll synchronisation is handled by `useScrollSync`; each editor scroll event sends an anchor line and percentage to the preview component.
- Pressing **Enter** at the end of the document triggers a preview update so late-stage edits stay visible.

## Attachments and Media

- Clipboard pastes, drag-and-drop uploads, and manual file selection all flow through `useEditorUploads`. Files are posted to `/api/files` and embedded with document-relative URLs so shared links continue to work.
- The toolbar exposes an explicit “Upload file” control. In read-only contexts the control is hidden and drop targets are disabled.

## Context Awareness

- The editor receives the active document ID, collaborator info, and access mode from the surrounding route. When plugins emit a `navigate` effect, the host updates the route and rehydrates the editor with the new context.
- Document-wide utilities—backlinks, outgoing links, Git status, and sharing controls—are provided by adjacent features (`refmd/app/src/features/document-backlinks`, `features/git-sync`, `features/sharing`).

Continue with the [System Architecture](architecture.md) overview to see how these pieces interact with the backend and realtime services.
