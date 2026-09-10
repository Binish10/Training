# Training

Illustrated training manual and workout log. Installable web app, works offline, data stays on the device.

## Files

| File | What it is |
|---|---|
| `index.html` | Everything — styles, the plan as data, 64 SVG diagrams, UI, logic |
| `sw.js` | Offline cache |
| `manifest.webmanifest` | Install metadata |
| `icon-192.png`, `icon-512.png` | Home screen icons |
| `.nojekyll` | Stops GitHub Pages processing the files |

No build step, no dependencies.

## Deploy

1. Push these files to the root of a public repo.
2. Settings → Pages → Source: **Deploy from a branch** → `main` / `/ (root)`.
3. Wait for the green check, then open the Pages URL **in Safari** on the iPhone.
4. Share → **Add to Home Screen**.

Only Safari can install to the home screen on iOS. Chrome on iOS cannot.

## Release rule

**Bump `const C` in `sw.js` on every change to `index.html`.**

Without it an installed copy keeps serving the cached old shell and your change never appears. This is the single most common way to lose an hour on a PWA.

## Local development

Service workers need http, not `file://`:

```
python3 -m http.server 8000
```

Then open `http://localhost:8000`.

## Data

Stored in `localStorage` under the `tt:` prefix. One key per session date, plus `tt:settings`.

iOS can evict browser storage under pressure, and clearing site data wipes it. **Export from the Data tab weekly.** That file is the only copy.

Import validates the payload shape and skips malformed session records rather than accepting them. Unparseable files are kept as a `recovery:` key instead of being discarded.

## Editing the plan

The `PLAN` object near the top of the script is the single source of truth. Four block types:

| Type | Logs |
|---|---|
| `sets` | reps per set, optional weight |
| `complex` | rounds per side, bell weight |
| `interval` | total minutes, with the run stage shown for the current week |
| `contacts` | reps and foot contacts |

Each block's `ex` field points at a diagram by slug — the same slug on `figure[data-ex]` in the manual, so diagrams are never duplicated.

Week number, run stage and step target all derive from the start date set in the Data tab.
