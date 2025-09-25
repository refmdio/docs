# Feature Overview

A catalogue of capabilities available in RefMD. Use this as the master checklist when documenting or verifying functionality.

## Document Workspace
- **Dashboard** – Lists the ten most recently updated notes with one-click access and a quick “create document” action.
- **File tree** – Hierarchical document/folder navigation with drag-and-drop reordering, rename, delete, secondary-viewer opening, and Git ignore controls.
- **Secondary viewer** – Optional right-hand pane that displays another document or plugin-provided content alongside the main editor.
- **Backlinks/outgoing links** – Toggleable panel showing references to and from the current note.
- **Search** – Modal search across titles with tag filtering and direct navigation to results.
- **Wiki links** – `[[...]]` auto-completion for cross-linking within the workspace.

## Editing
- **Markdown editor** – Monaco-based editor supporting syntax highlighting, keyboard shortcuts, and auto-completion.
- **Display modes** – Editor-only, Split, and Preview layouts, remembered per user session.
- **Formatting palette** – Floating toolbar for bold, italics, headings, lists, tasks, quotes, code, tables, horizontal rules, and link insertion.
- **Scroll synchronisation** – Optional linkage between editor and preview scrolling in split mode.
- **Inline preview** – Server-rendered Markdown with hydration for wiki links, attachments, and custom elements.
- **Theme & Vim mode** – Light/dark theme switcher and Vim keybinding toggle stored per browser.
- **Attachments** – Upload via drag/drop, paste, or file chooser; embedded references use relative paths to the document’s attachments directory.

## Collaboration & Presence
- **Realtime cursors** – Colour-coded cursors and selections for each active user.
- **Presence list** – Header display of participants with click-to-follow navigation.
- **Connection status** – In-editor overlay indicating live/connecting/offline states.

## Plugins
- **Plugin catalogue** – View global and user-installed plugins with metadata (version, mounts, permissions, commands, author, repository).
- **Install from URL** – Upload plugin bundles via URL and optional token.
- **Removal** – Uninstall user-scoped plugins directly from the catalogue.
- **Command menu** – Sidebar plugin menu exposing declared `ui.toolbar` commands.
- **Secondary integrations** – Plugins can render secondary panes, inject commands, or control document routes via their manifest.

## Git Integration
- **Status indicator** – Shows repository health, pending changes, and sync errors.
- **Sync actions** – Trigger fetch/push cycles, initialise repositories, and configure remotes.
- **Diff & history dialogs** – Inspect per-file changes and commit history.
- **Ignore rules** – Add documents or folders to `.gitignore` from the file tree.

## Sharing & Publishing
- **Share links** – Generate document or folder URLs with view/edit/admin permissions and optional expirations (1 hour, 24 hours, 7 days, 30 days, or never).
- **Folder share tree** – Inspect child items included in folder shares and remove them individually without altering the source hierarchy.
- **Token management** – Copy or revoke share links from a single dialog.
- **Public publishing** – Assign permanent public URLs (`/u/<user>/<doc>`) and toggle publication status per document.
- **Share recipients** – Document shares open the full editor with scoped permissions; folder shares render a document picker; public links show a simplified, read-only layout.

## Visibility & Governance
- **Visibility dashboard** – Consolidated view of all active share links and published documents with copy, revoke, and unpublish controls.
- **Usage insights** – Counts of public documents and share links displayed at a glance.

## Profile & Accounts
- **Account profile** – Displays name, email, and direct links to visibility management and public pages.
- **Public profile** – Public landing page listing published documents for external viewers.
- **Authentication** – Email/password login and registration with validation guidance.
- **Session controls** – Sign out from the sidebar settings menu.

## Responsive Experience
- **Mobile layouts** – Adaptive sidebar, header, and editor/preview stacking for phones and tablets.
- **Touch interactions** – Access to search, share, plugin commands, and backups via mobile-friendly dialogs and buttons.

Keep this feature catalogue in sync with product releases so the documentation always reflects the functionality delivered to users.
