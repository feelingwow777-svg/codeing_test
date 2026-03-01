# CLAUDE.md

This file provides guidance for AI assistants (Claude, Copilot, etc.) working on this codebase.

## Project Overview

**비행기 경로 시뮬레이터** (Aircraft Route Simulator) is a Korean-language Progressive Web App (PWA) that simulates flight routes between cities. It renders an interactive map, animates a flight path, and shows a "window view" video overlay while the flight progresses.

- **Language:** Vanilla HTML5 / CSS3 / JavaScript (ES6) — zero external dependencies
- **Type:** PWA (installable, offline-capable via Service Worker)
- **Localization:** Korean (`lang="ko"`)

## Repository Structure

```
codeing_test/
├── index.html             # Entry point — full app shell and UI markup
├── README.md              # Minimal project description
├── CLAUDE.md              # This file
│
│   ── (files referenced in index.html but not yet committed) ──
├── style.css              # All application styles
├── manifest.json          # PWA manifest (icons, display, theme)
├── sw.js                  # Service Worker (caching strategy, offline support)
├── flight video.mp4       # Looping in-flight window video
└── js/
    ├── airports.js        # Airport/city data and lookup utilities
    ├── flight-simulator.js# Core simulation engine (route calculation, animation)
    ├── offline-map.js     # Canvas-based map renderer + offline tile logic
    ├── audio.js           # Audio context, engine sound generation/playback
    └── main.js            # App bootstrap — wires all modules together
```

> **Note:** Only `index.html` and `README.md` are currently committed. All other files listed above are referenced by the HTML but have not yet been added to the repository.

## Architecture

### Module Responsibilities

| File | Responsibility |
|---|---|
| `airports.js` | Provides airport/city coordinates and metadata; populates `<select>` dropdowns |
| `flight-simulator.js` | Great-circle route math, step-by-step position interpolation, speed multiplier logic |
| `offline-map.js` | Draws the map (canvas or tile-based); handles offline tile caching |
| `audio.js` | Web Audio API engine sounds; exposes toggle and volume controls |
| `main.js` | Entry point; initialises modules, attaches all DOM event listeners |

### UI Components (defined in `index.html`)

- `#map` / `#videoContainer` — toggled map vs. window-view panels
- `#departure` / `#destination` — city selector dropdowns
- `#distance`, `#flightTime`, `#progress` — live flight telemetry
- `#startBtn`, `#pauseBtn`, `#stopBtn` — playback controls
- `.speed-btn[data-speed]` — speed multiplier (0.5x / 1x / 2x / 4x)
- `#soundToggle` / `#volumeSlider` — audio controls
- `#progressFill` — progress bar fill element

### PWA Setup

- `manifest.json` is linked from `<head>`; must include at minimum `name`, `short_name`, `start_url`, `display: standalone`, `theme_color`, and icon entries.
- `sw.js` is registered in an inline `<script>` at the bottom of `<body>`. The service worker should implement a cache-first or stale-while-revalidate strategy for all app assets.
- Cache-busting uses query strings: all `<script>` tags append `?v=<N>` (currently `?v=9`). Increment `N` whenever a JS file changes to force clients to re-fetch.

## Development Workflow

### Running the App

There is no build step. Serve the repository root with any static file server:

```bash
# Python 3
python3 -m http.server 8080

# Node.js (npx)
npx serve .

# VS Code Live Server extension also works
```

Then open `http://localhost:8080` in a browser.

### Making Changes

1. Edit the relevant file(s) directly (`index.html`, `style.css`, or a JS module).
2. Increment the `?v=N` query string on any changed `<script>` tags in `index.html` to bust browser and service worker caches.
3. Hard-refresh the browser (`Ctrl+Shift+R` / `Cmd+Shift+R`) or clear the service worker in DevTools > Application > Service Workers when testing SW changes.

### No Build, No Tests, No CI

This project currently has:
- **No bundler** (no webpack, Vite, Rollup, etc.)
- **No package manager** (no `package.json`)
- **No automated tests** (no Jest, Vitest, Playwright, etc.)
- **No CI/CD pipeline**

All verification is manual. When adding infrastructure (tests, linter, bundler), document it here.

## Key Conventions

### JavaScript

- **Vanilla ES6+** — use `const`/`let`, arrow functions, template literals, `async/await`. No TypeScript.
- **No frameworks** — no React, Vue, etc. Manipulate the DOM directly.
- **Module pattern** — each `js/*.js` file should expose a single object or set of functions on the global scope (or use ES modules if a bundler is added later).
- **No external CDN dependencies** — keep the app fully offline-capable.

### HTML / CSS

- Maintain the Korean locale (`lang="ko"`, Korean labels) for all UI text.
- Class naming follows a flat BEM-lite convention: `control-panel`, `control-section`, `btn`, `btn-primary`, `btn-secondary`, etc.
- Button states (disabled) are managed via the `disabled` HTML attribute and mirrored in JS.

### Cache Busting

When modifying any `js/*.js` file, update the corresponding `?v=N` in `index.html`:

```html
<!-- Before -->
<script src="js/airports.js?v=9"></script>

<!-- After (if airports.js changed) -->
<script src="js/airports.js?v=10"></script>
```

### Git

- Branch naming: `claude/<description>-<id>` for AI-generated branches.
- Commit messages should be concise and in English even though the UI is in Korean.
- There is no `.gitignore` yet — add one if editor/OS artefacts appear (`.DS_Store`, `*.swp`, `node_modules/`, etc.).

## Missing Files Checklist

If you are setting this project up from scratch or recovering missing files, ensure the following exist before the app will function:

- [ ] `style.css` — application stylesheet
- [ ] `manifest.json` — PWA manifest
- [ ] `sw.js` — service worker
- [ ] `flight video.mp4` — window-view video asset
- [ ] `js/airports.js`
- [ ] `js/flight-simulator.js`
- [ ] `js/offline-map.js`
- [ ] `js/audio.js`
- [ ] `js/main.js`

## Suggested Next Steps

For contributors looking to improve the project:

1. **Commit missing files** — add `style.css`, `manifest.json`, `sw.js`, `js/*.js`, and any other referenced assets.
2. **Add `.gitignore`** — at minimum exclude OS files and any future `node_modules/`.
3. **Add a linter** — ESLint with a minimal config keeps JS consistent without requiring a build step.
4. **Add basic tests** — Vitest or Jest with jsdom can test pure logic in `flight-simulator.js` and `airports.js` without a browser.
5. **Document the airport data format** — describe the shape of airport objects in `airports.js` so new entries can be added confidently.
