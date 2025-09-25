# User Guide

## 1. Access and Layout
- Sign up with name, email, and password; existing accounts sign in from the same entry point. Successful login opens the dashboard showing recent documents.
- The sidebar exposes the file tree, new document/folder actions, plugin commands, Git Sync, and the settings menu.
- The main workspace combines the Markdown editor with a live preview. Desktop layouts place them side by side; smaller screens stack them vertically.
- The header bar offers mode toggles (Editor/Split/Preview), search, backlinks, download, share, and theme controls, alongside collaborator avatars.

## 2. Document Management
- Create documents or folders via the buttons at the top of the file tree. Rename immediately to keep the tree organised.
- Drag and drop items to reorder. Context menus provide rename, open in secondary viewer, ignore in Git, and delete options.
- The dashboard lists up to ten recently updated documents with shortcuts to reopen or create new content.

## 3. Search and Navigation
- Use the header search button to find documents by title or filter by tag, then jump directly to the selected item.
- Activate the backlinks button to review incoming and outgoing references for the current note.

## 4. Writing
- Switch between Editor, Split, and Preview modes; the application remembers the last selection per browser.
- The floating toolbar supplies bold, italic, heading, list, table, quote, code, horizontal rule, and link commands. Standard keyboard shortcuts remain available.
- Typing `[[` opens wiki-link suggestions so internal references can be inserted without leaving the editor.
- Scroll synchronisation keeps editor and preview aligned in split mode and can be disabled from the toolbar.

## 5. Attachments
- Paste or drag files into the editor, or choose the upload button. Attachments are stored with the document and linked automatically.
- Images open in a lightbox within the preview; other files appear as regular links. Read-only visitors retain preview access but cannot upload.

## 6. Collaboration
- Collaborators appear in the header; selecting an avatar scrolls to that user’s cursor. Coloured selections update in real time.
- Connection status is indicated above the editor, clarifying whether you are live, reconnecting, or offline.
- Theme (light/dark) and Vim mode can be toggled from the header controls.

## 7. Plugins
- The Plugins page lists installed extensions, including scope (global or user), version, mounts, permissions, commands, author, and repository links.
- Install a plugin by supplying the bundle URL (and token if required); remove user-installed plugins with the Remove action.
- Installed commands appear in the sidebar plugin menu so they can be triggered with a single click.

## 8. Git Sync
- The Git Sync button shows repository status, pending changes, and errors.
- Use the dropdown to configure the repository, inspect diffs or history, and run a sync. Individual files can be ignored in Git from their context menu.

## 9. Sharing and Publishing
- The Share & Publish dialog creates document or folder links with view, edit, or admin permission levels and optional expiry windows (1 hour, 24 hours, 7 days, 30 days, or none).
- Folder shares display nested contents: copy the generated URLs or remove child entries without deleting the underlying folder.
- Publishing enables a permanent public URL (e.g. `/u/<name>/<document-id>`). Copy, open, or revoke the link within the same dialog.

## 10. Visibility Overview
- The Visibility page consolidates all active share links and published documents. Copy URLs, remove links, and unpublish documents in one place.

## 11. Profile and Public Pages
- The Profile page presents account details with shortcuts to visibility management and public pages. Profile links open the public-facing listing of published documents.
- Public visitors see a simplified layout dedicated to published content, with read-only access to each document.

## 12. Mobile Use
- On phones and tablets the sidebar collapses into menus and the editor/preview stack vertically. All actions remain available through dialogs and buttons adapted to smaller screens.

## 13. Signing Out
- Use **Sign out** from the sidebar settings menu to end the session.

Review this guide whenever RefMD adds or changes user-facing functionality so it always reflects the product in production.
