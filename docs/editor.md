# Editor Guide

The RefMD editor combines Monaco, Yjs, and custom UI so teams can co-author Markdown documents in real time. This page highlights the key capabilities available by default.

## Layout and View Modes

- **Split view** (default) keeps the Markdown source and rendered preview side by side. Toggle to `editor` or `preview` only when you need extra focus; on phones the panes stack vertically.
- **Floating tool palette** sits in the lower-right corner so formatting actions, uploads, and view controls are always within reach without covering the writing area.
- **Secondary panels** (for example backlinks or plugins) appear on the right when available and automatically collapse on smaller screens.
- **Scroll sync** keeps the preview aligned with the caret in split view and can be disabled from the toolbar if you want each pane to move independently.

## Live Collaboration

- Changes appear in real time for every participant working on the same document.
- Colored cursors and selection highlights make it easy to see where teammates are editing.
- A Vim mode toggle is available for users who prefer modal editing shortcuts; RefMD remembers your preference per browser.
- Dark and light themes flow through to the editor automatically, so collaborators see the same palette as the rest of the interface.

## Markdown Tooling

- Apply formatting like bold, italics, headings, lists, task lists, quotes, code, tables, horizontal rules, and links with a single click or keyboard shortcut.
- Task lists stay interactive in the preview—check items off and everyone sees the update immediately.
- Typing `[[...]]` pops up document suggestions so you can create internal links without breaking your flow; the preview can open those links even when they point to another note.
- The toolbar and context menus expose upload, formatting, and navigation commands in one place, but nothing stops power users from relying on familiar Markdown syntax.

## Preview and Sync

- The preview pane renders your Markdown instantly and upgrades custom components such as link cards, attachment viewers, and wiki badges without extra setup.
- A table of contents appears on desktop, while mobile and split modes offer a floating TOC you can open from the preview toolbar.
- Scroll sync keeps the rendered pane aligned with the editor when enabled; turn it off when you want the panes to move independently.
- Dropping to a new line at the end of the document forces a refresh so late edits never get lost offscreen, and clicking a wiki link or hashtag will navigate or open search without leaving the preview flow.

## Attachments and Media

- Paste from the clipboard, drag files into the editor, or use the upload button—RefMD detects duplicates automatically and inserts the right Markdown so attachments stay in step with the document.
- Uploaded images open in a lightbox from the preview, letting you zoom without leaving the page. Other file types drop in as links that respect document-relative paths, keeping shared links portable.
- Read-only viewers keep the layout but disable uploads and editing controls, making it clear when you’re browsing a shared document.

## Sharing and Publishing

- Open **Share & Publish** from the toolbar or sidebar to generate invite links. Links can be scoped to a single document or an entire folder, carry view/edit/admin permissions, and optionally expire after 1 hour, 24 hours, 7 days, 30 days, or never.
- Shared links track usage counts so you can monitor how often a token has been used, revoke links at any time, or remove documents from folder shares without touching the originals.
- Published documents get a permanent public URL (for example `https://refmd.io/u/you/doc`) that surfaces on your public profile; toggle publishing on or off per document and copy or open the live page directly from the dialog.
- Recipients accessing a share token see the same collaborative editor—complete with real-time cursors, preview, and attachments—while public readers experience the simplified public layout.

## Context Awareness

- The editor keeps track of document context—open documents, share tokens, plugins—and refreshes tools automatically when navigation happens from the preview, file tree, or plugins.
- Adjacent panels expose backlinks, outgoing links, Git status, and sharing controls so you can manage metadata without leaving the page.
- When a plugin or link navigates to another note, the editor rehydrates with the new content, collaborators, and permissions in a single step.

Continue with the [System Architecture](architecture.md) overview to see how these pieces interact with the backend and realtime services.
