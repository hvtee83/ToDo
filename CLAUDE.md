# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A deliberately simple to-do tracker: plain HTML/CSS/vanilla JS, no frameworks, no build step, no tests, no package manager. Keep it that way. Not a git repository.

To run it, open `index.html` in a browser (double-click, or serve the folder with any static server). To verify UI changes, load it in a browser and exercise it by hand or via scripted DOM interaction.

## Files

- `index.html` is the whole app: markup, `<style>` and `<script>` in one file.
- `mockup.html` is a generated page that shows the app in iPhone 13 (390×844, light) and Vivo R14 (360×800, dark) frames. Each frame is an `<iframe srcdoc>` holding a **full copy** of `index.html` plus a seed script appended before `</body>` (`tasks=[...]; setTheme(...); render();`). The copy is a snapshot: it goes stale whenever `index.html` changes and must be regenerated or re-edited by hand. The headings and captions above/below the phones were added by hand, so a regeneration has to preserve them.

## Architecture of `index.html`

- **State**: a `tasks` array of `{id, text, category, due, done}`. `due` is `'YYYY-MM-DD'` or `''`. New ids are `max(id)+1`. `category` is one of `CATEGORIES = ['Work','Personal','Others']`, with Work as the default.
- **Rendering**: `render()` rebuilds the whole list from `tasks` on every change, through the `el()` helper (a wrapper over `createElement` + `Object.assign`). Text goes in via `textContent`, never `innerHTML`.
- **Ordering** (in `render()`): done tasks always sink to the bottom. The "Sort by due date" checkbox adds a secondary key by due date, with undated tasks last. The sort is stable, so it keeps insertion order otherwise.
- **Events**: delegation on `#list`. One click handler uses `e.target.closest('button')` (the buttons contain SVG icons, so don't compare `tagName`) and dispatches on the classes edit/save/cancel/del. Double-clicking `.text` also starts editing. `startEdit(li)` swaps the row for an inline editor (text, category select, date, Save/Cancel; Enter saves, Esc cancels). It keeps a legacy category as an extra option if one exists.
- **Persistence**: localStorage keys `todo.tasks`, `todo.sortDue`, `todo.theme`. Every access is wrapped in try/catch, and the app must still work with storage unavailable (as in private windows and the browser preview pane).
- **Theming**: CSS custom properties on `:root`, overridden under `:root[data-theme="dark"]`. `setTheme(mode, persist)` sets the attribute. The initial theme is the saved value, else `prefers-color-scheme`. The sun and moon toggle buttons carry `data-theme-btn` and `aria-pressed`.
- **Mobile**: layout was checked for no horizontal overflow at 390px and 360px widths. Preserve that when changing the styles.
