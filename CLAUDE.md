# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) and other AI assistants when working in this repository.

## Project Overview

**本質チェッカー (Essence Checker)** is a mobile-first Progressive Web App (PWA) that takes a word as input and reveals its "essence" (本質) — a sharp, aphoristic interpretation of what the word really means. The UI is in Japanese.

Core behavior:
- User types a word and taps 診断 ("Diagnose").
- The app looks the word up in a built-in essence database (~70 entries), falls back to partial matching, and finally generates an essence from templates for unknown words.
- Results can be shared (Web Share API / clipboard) and are saved to a local history.

This is a **zero-dependency, no-build** static web app. There is no framework, no package manager, no bundler, and no backend. It runs by opening `index.html` in a browser.

## Repository Structure

| File | Purpose |
|------|---------|
| `index.html` | The entire application — HTML, CSS (in a `<style>` block), and JS (in a `<script>` block) all live here. This is the file you edit for almost any change. |
| `manifest.json` | PWA manifest (name, icons, colors, `display: standalone`). |
| `sw.js` | Service worker. Cache-first strategy; caches `index.html` and `manifest.json`. |
| `generate-icons.html` | One-off utility: open in a browser to generate/download the `icon-192.png` and `icon-512.png` referenced by the manifest. The PNG icons themselves are **not** committed. |
| `README.md` | User-facing description (Japanese). |

There is no `src/`, no tests directory, and no config files beyond the above.

## Architecture (all inside `index.html`)

The `<script>` block is organized into clearly commented sections:

1. **本質データベース (`essenceDB`)** — an object keyed by Japanese word → `{ essence, detail }`. Grouped by theme (emotions, society/work, relationships, technology, daily life, abstract concepts).
2. **カテゴリ別テンプレート (`essenceTemplates`)** — array of `{ pattern(w), detail(w) }` functions used to synthesize an essence for words not in `essenceDB`.
3. **サンプルワード (`exampleWords`)** — words shown as tappable example chips on the empty state.
4. **状態管理** — `history` array, persisted to `localStorage` under the key `essenceHistory`.
5. **Core flow:** `checkEssence()` → `getEssence(word)` → `showResult()` + `saveToHistory()`.
   - `getEssence()` resolution order: exact match → partial match (`includes`) → `generateEssence()`.
   - `generateEssence()` picks a template **deterministically** via a hash of the word, so the same unknown word always yields the same essence.
   - `checkEssence()` adds an artificial 0.8–2.0s delay to simulate "analysis".
6. **Rendering** — `showResult()`, `renderHistory()`, `renderHistoryHTML()` build HTML strings injected via `innerHTML`.
7. **Actions** — `shareResult()` (Web Share API with clipboard fallback), `showToast()`, history management (`saveToHistory`, `clearHistory`).
8. **Utilities** — `escapeHtml()` / `escapeAttr()` for sanitizing user input before it enters `innerHTML` / inline `onclick` attributes.
9. **PWA registration** — registers `sw.js` if supported, then calls `init()`.

## Development Workflow

### Running / previewing
There is no build step. Either:
- Open `index.html` directly in a browser, **or**
- Serve the folder (recommended, so the service worker and manifest load correctly):
  ```bash
  python3 -m http.server 8000
  # then open http://localhost:8000
  ```

### Testing
There is no automated test suite. Verify changes manually in a browser:
- Test a word that exists in `essenceDB` (e.g. 愛, お金).
- Test a partial match (e.g. a word containing a known key).
- Test an unknown word to confirm template generation and determinism.
- Confirm history persistence (reload the page), sharing, and the clear-history action.
- Check mobile layout (the app is constrained to `max-width: 480px`).

### Service worker caching — IMPORTANT
`sw.js` uses a cache name `essence-checker-v1` and serves cached assets first. **When you change `index.html` (or any cached asset), bump the `CACHE_NAME` version** (e.g. `essence-checker-v2`) so returning users actually receive the update. The `activate` handler deletes old caches automatically once the name changes.

## Conventions

- **Single-file app:** keep HTML, CSS, and JS together in `index.html` unless there's a strong reason to split. Match the existing section-comment style (`// ====` banners).
- **Language:** all user-facing strings are Japanese. Keep them Japanese. Code identifiers and comments are a mix of English and Japanese — follow the surrounding style.
- **Vanilla only:** no dependencies, no frameworks, no build tooling. Don't introduce npm, bundlers, or external scripts.
- **Styling:** uses CSS custom properties defined in `:root` (colors, radius). Reuse these variables; don't hardcode new color values. Dark theme, purple/cyan accent palette.
- **Security:** any user-controlled text rendered into the DOM must go through `escapeHtml()`, and anything placed into an inline `onclick` attribute must go through `escapeAttr()`. Preserve this when editing rendering code.
- **Adding essences:** add new entries to `essenceDB` under the appropriate thematic group, following the `'word': { essence, detail }` shape. Add words to `exampleWords` if you want them surfaced as chips.

## Git Workflow

- The default branch is `main`. Each feature/app has historically been developed on a `claude/...` branch and merged via PR.
- Commit messages in this repo use Conventional Commits with Japanese descriptions (e.g. `feat: 本質チェッカー - 言葉の本質を見抜くPWAアプリ`).
- Do not create a pull request unless explicitly asked.
</content>
</invoke>
