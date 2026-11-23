# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A deliberately minimal to-do app: plain HTML, CSS and JavaScript. No frameworks, no build step, no dependencies, no package.json, no tests, and not a git repo. Keep it that way unless asked.

- `index.html` is the whole app (markup, `<style>`, `<script>` in one file). Open it directly in a browser to run it.
- `mockups.html` is a presentation page showing phone-frame screenshots of the app (iPhone, Galaxy S25). The screenshots are embedded as base64 `data:` URIs so the page is self-contained (this is required for the side-panel preview, which renders local files as static `data:` snapshots where relative paths break). `mockups/*.jpg` are the source images and are not referenced by the page. If you re-embed images, use `node` (the system `python3` is broken on this machine, with an xcrun error).

## Architecture of index.html

- **State:** a single `todos` array of `{id, text, category, due, done}`, persisted to `localStorage` (keys `"todos"` and `"theme"`). All storage access is wrapped in try/catch so the app still works where storage throws.
- **Rendering:** `render()` rebuilds the whole `<ul id="list">` from `todos` on every change. User text is set via `textContent`, never `innerHTML` (injection safety); keep it that way.
- **Events:** delegated on the `<ul>` (click, dblclick, keydown, focusout), so rows need no individual listeners.
- **Editing:** driven by the `editingId` variable. `render()` shows an inline edit row for that id. `commitEdit` bails out unless `editingId` matches, which prevents a double commit when removing the input triggers a blur. The focusout handler ignores focus moving between fields of the same `.editrow`.
- **Categories:** fixed list in `CATEGORIES` (Class, Home, Work, Personal, Others). Items with an unknown category are shown and edited as "Others".
- **Due dates:** stored as `YYYY-MM-DD` strings and compared as strings against `today()` (local time) to flag overdue items that aren't done.
- **Theme:** CSS variables on `:root` and `:root[data-theme="dark"]`. Initial theme is the saved choice, else `prefers-color-scheme`.
- **Mobile:** viewport meta plus an `@media (max-width: 420px)` block that puts category and date side by side. Verified at 393 and 360 px widths.

## Checking changes

- Syntax check: extract the `<script>` body to a file and run `node --check <file>`.
- The in-app browser pane renders local files as static snapshots with `localStorage` disabled, and reload doesn't re-run the page. To re-test, `navigate` to the `file://` URL again (new tab id). Wrap `javascript_exec` snippets in IIFEs and avoid names like `add`/`render`, because function declarations there become globals and overwrite the page's own functions.
- Persistence and the full edit flow have only been syntax-checked, not exercised in a normal browser with storage enabled.
