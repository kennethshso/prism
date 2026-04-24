# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Prism Tools** is a zero-dependency, no-build utility dashboard — a static SPA served as plain HTML/CSS/JS files. There is no package manager, no bundler, and no framework. Everything lives in `index.html`.

## Running the Project

Open `index.html` directly in a browser, or serve with any static file server:

```bash
python3 -m http.server 8000
# or
npx serve .
```

PWA features (service worker, install prompt) require HTTPS or `localhost` — they won't work on `file://`.

## Architecture

### Single-file SPA (`index.html`)

All CSS, HTML structure, and JavaScript are embedded in one file. The structure inside `<script>` follows this pattern:

1. **Module-level state** — shared state that survives route changes (e.g., `pomo` object for the Pomodoro timer)
2. **Page renderers** — pure functions (`renderHome`, `renderURLShortener`, `renderPomodoro`, `renderQRGenerator`) that return HTML strings
3. **Tool initialisers** — functions called after render to attach event listeners (`initURLShortener`, `initPomodoro`, `initQRGenerator`)
4. **Router** — hash-based (`#home`, `#url-shortener`, `#pomodoro`, `#qr-generator`); calls renderer then initialiser on each navigation
5. **PWA registration** — registers `sw.js`

### Adding a New Tool

1. Add a render function: `function renderMyTool() { return \`...\`; }`
2. Add an init function: `function initMyTool() { /* attach listeners */ }`
3. Register the route: `'#my-tool': { render: renderMyTool, init: initMyTool }`
4. Add a card in `renderHome()` with the same hash
5. Add a nav link in `<header>`

### External Dependencies (CDN)

| Library | URL | Used for |
|---|---|---|
| Inter font | Google Fonts | Typography |
| qrcodejs 1.0.0 | cdnjs.cloudflare.com | QR code rendering |
| is.gd API | `https://is.gd/create.php?format=json&url=` | URL shortening |

qrcodejs renders both a `<canvas>` (hidden) and an `<img>` element. The download button reads from `img.src` (with canvas fallback).

### PWA Files

- `manifest.json` — app metadata, icons, home screen shortcuts per tool
- `sw.js` — cache-first strategy; skips is.gd API requests (always needs network); cache key is `prism-v1` — bump this string to bust the cache on updates
- `icon.svg` — single SVG icon used for all sizes via `"sizes": "any"` in the manifest

### Design Tokens

All colours and radii are CSS custom properties on `:root` in `index.html`. Key values:

```
--bg: #F4EFE4      --primary: #8B6535   --green: #5C7A5C
--surface: #FDFBF7 --accent:  #B07A4A   --text:  #2D2417
--card: #FFFFFF    --muted:   #8B7D6B   --border: #DDD4C4
```

### Pomodoro State Persistence

The `pomo` object is declared at module scope so the timer keeps running when the user navigates away and back. The router re-renders the Pomodoro page from the current `pomo` state, so the ring and time display always reflect live values. `document.title` is updated with the countdown while running.
