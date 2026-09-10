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

No build step, no dependencies. Create `.nojekyll` in GitHub with **Add file → Create new file** — browsers hide dotfiles, so dragging it in usually fails.

## Deploy

1. Push these files to the root of a public repo.
2. Settings → Pages → Source: **Deploy from a branch** → `main` / `/ (root)`.
3. Open the Pages URL **in Safari** on the iPhone. Only Safari can install to the home screen on iOS.
4. Share → **Add to Home Screen**.

## Release rule

**Bump `const C` in `sw.js` on every change to `index.html`.**

Without it an installed copy keeps serving the cached old shell and your change never appears. Currently at `training-v8`.

After a design or icon change, delete the home screen icon and re-add it — the version bump refreshes the page but not always the icon.

## Local development

Service workers need http, not `file://`:

```
python3 -m http.server 8000
```

## Data

`localStorage`, `tt:` prefix. One key per session date, plus `tt:settings`.

iOS can evict browser storage under pressure, and clearing site data wipes it. **Export weekly from the Data tab.** That file is the only copy.

Import validates the payload and skips malformed session records. Unparseable files are kept under a `recovery:` key rather than discarded.

## What's in it

- **Today** — reads the day, loads that session, fields prefill from last time. Five block types: sets, kettlebell complexes, run intervals, foot contacts, and tick-only for recovery days. Friday is hidden until week 3 and alternates plyometrics and cones.
- **Manual** — 67 diagrams in collapsible sections, each with a Watch link, plus a curated "Learn these first" list of 12.
- **History** — sparklines for pull-ups, push-ups, kettlebell rounds, sprint reps and steps.
- **Data** — start date, export, import, storage usage, erase.
- **Rest timer** — 10s to 3m. Timestamp-based, so locking the phone doesn't break it. Shows on Today, and follows you across tabs while running.
- **Dark mode** — follows the system setting.

## Editing the plan

`PLAN` near the top of the script is the single source of truth. Each block's `ex` field points at a diagram by slug, matching `figure[data-ex]` in the manual, so diagrams exist once.

`VIDEO` maps every diagram slug to a YouTube **search query** — never a video ID, because videos get deleted and privatised while search queries don't rot. Add a slug there and a Watch link appears automatically on that diagram and on any block using it. Queries are written to land on instructional results ("pigeon pose hip stretch how to", not "pigeon") because bare names return workout compilations.

`LEARN` is the curated subset shown in the Manual's "Learn these first" section — the movements where the timing matters and a diagram can't convey it. Each needs a `why` line describing what to watch for.

Week number, run stage and step target all derive from the start date in the Data tab.

## Plan revision 2

The programme was audited and corrected. Squat and single-leg patterns added, the RDL moved off Wednesday, the Friday speed day held back to week 3 and split into alternating plyometric and cone weeks, and a short upper block added after Tuesday's run. Full rationale is in `master-training-plan.md`.

## Known gaps

- Export is manual — no reminder.
- No automatic back-off signal when performance declines across sessions.
