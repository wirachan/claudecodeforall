# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A static, dependency-free Thai food recommender web app ("แนะนำอาหารไทย"). No build step, no package manager, no test runner — plain HTML/CSS/JS served directly from disk or any static file server. Content and UI copy are entirely in Thai.

`CONTEXT.md` is the domain glossary (เมนู, แคตตาล็อก, หมวด, ภาค, ระดับความเผ็ด, สุ่มเมนู, เมนูวันนี้, รายการโปรด) — read it before changing terminology or behavior tied to these concepts, since the code follows this vocabulary closely (variable names, data fields, UI labels). `task.md` tracks the acceptance checklist for the original build; treat unchecked items as open work, not as unspecified behavior.

## Running / testing

There is no build or test tooling. To work on the app, open `index.html` directly in a browser (or serve the folder with any static server, e.g. `npx serve .`) and click through manually. There is no linter or automated test suite — verify changes by exercising the UI (drawing a menu, toggling favorites, filtering/searching, checking mobile width ~390px, and checking the browser console for errors), per the manual checks listed in `task.md` section 7.

## Architecture

Three pages share state through one script:

- **`common.js`** — the shared module (`window.ThaiFood`). Owns the `CATALOG` array (10 dishes, each with `id`, `th`/`en` names, `desc`, `cat`, `spice` 0-3, `region`, `emoji`/`tint`/optional `img`) and the `INGREDIENTS` map keyed by dish `id`, merged onto each dish as `d.ing`. Also owns favorites persistence (`STORAGE_KEY = "thai-food-favorites"`, `loadFavorites`/`saveFavorites`, with `storageOk()` reporting whether localStorage read/write failed) and small render helpers (`heartBtn`, `spiceHtml`, `menuUrl`) reused by both pages. **Any new dish or ingredient list is added here**, not in the page files.
- **`index.html`** — catalog page: search box, category/region/spice chip filters, the "สุ่มเมนู" (draw) feature with a short spin animation (skipped under `prefers-reduced-motion`) that never repeats the immediately-previous pick, and the favorites section. All rendering is manual `innerHTML` string-building (no framework/virtual DOM) driven by module-scoped filter state (`activeCat`, `activeRegion`, `activeSpice`, `query`).
- **`menu.html`** — per-dish detail page, reached via `?id=<dish-id>` (built by `F.menuUrl`), showing full ingredient groups. Handles a legacy `#/menu/<id>` hash route from `index.html` by redirecting here with `&from=legacy`, which changes back-link behavior (falls back to a plain href instead of `history.back()` since there's no real previous page in this session).
- **`style.css`** — shared styles for both pages, built on CSS custom properties defined in `:root` (`--cream`, `--chili`, `--leaf`, etc.) for the cream/chili-orange/banana-leaf-green theme.

Favorites are synced across the two pages and across tabs: both listen for the `storage` event on `STORAGE_KEY`, and `index.html` also resyncs on `pageshow` when the page is restored from bfcache (navigating back from `menu.html`). Favorite/localStorage failures are surfaced via a dismiss-free `#warn` banner rather than blocking the UI — the catalog and detail views must keep working in-memory even when storage is unavailable.

All catalog/detail markup is built from data the app itself defines (`CATALOG`/`INGREDIENTS` in `common.js`), so the `hl()` search-highlighting and card-building string concatenation in `index.html`/`menu.html` do not HTML-escape those values — do not introduce user-controlled input into that path without escaping it.
