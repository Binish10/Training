# Training Tracker PWA — Project Plan

**Goal:** the illustrated manual plus a workout logger, running as an installable app on an iPhone 15, working offline, with data that's yours and exportable.

**Stack:** one `index.html`, one `sw.js`, one manifest. No framework, no build step, no dependencies. Hosted free on GitHub Pages.

**Total effort:** roughly 12–16 hours of build time, spread across stages you can use between.

---

## Guiding constraints

These decide most of the smaller choices later, so they're worth fixing up front.

**No build step.** Everything inline. The moment you add a bundler you inherit a toolchain that rots, and the whole appeal of this app is that it still works in three years without maintenance.

**Offline first, always.** You'll use this in a park with bad signal. Every feature must work with the network off.

**Data is yours and portable.** Export from day one, not as a later feature. Browser storage is not durable — iOS can evict it under storage pressure, and clearing site data wipes it.

**Ship each stage before starting the next.** Use it for a week. The features you think you need before training and the ones you actually need after three sessions are different sets.

---

## Repo structure

```
training/
├── index.html          # everything: styles, plan data, diagrams, UI, logic
├── sw.js               # offline cache
├── manifest.webmanifest
├── icon-192.png
├── icon-512.png
├── .nojekyll           # stops Pages mangling files
└── README.md
```

Five files. That's the whole project.

---

# Stage 0 — Repo and hosting skeleton

**Goal:** a URL on your phone showing "hello", installed to the home screen.

Do this before writing any real code. Getting deployment working on a trivial page takes 20 minutes; debugging it later while also debugging your app takes an evening.

### Tasks
1. Create a public repo `training` (public is required for free Pages).
2. Add `index.html` with a heading and nothing else.
3. Add `.nojekyll` (empty file) at the root.
4. Settings → Pages → Source: Deploy from a branch → `main` / root.
5. Wait for the green check, open the URL on your iPhone in **Safari** (not Chrome — only Safari can install to the home screen on iOS).
6. Share → Add to Home Screen. Confirm the icon appears and opens.

### Done when
The app opens from your home screen and shows your heading. Pushing a change to `main` and reloading shows the new text.

**Time:** 30 minutes.

---

# Stage 1 — The manual, ported

**Goal:** the full illustrated manual readable on the phone, offline.

You already have this file. This stage is mostly a port plus mobile navigation.

### Tasks
1. Move the existing manual HTML into `index.html`.
2. Add a bottom tab bar — the thumb-reachable zone on a 6.1" screen. Three tabs for now: **Today**, **Manual**, **History**. Today and History are stubs.
3. Make the manual collapsible by section, so you're not scrolling past 64 diagrams to reach the cool-downs.
4. Add safe-area padding (`env(safe-area-inset-bottom)`) so the tab bar clears the home indicator.
5. Set `<meta name="apple-mobile-web-app-capable" content="yes">` and a status bar style, so it launches full screen without Safari chrome.

### Done when
You can find any exercise diagram in under three taps, in landscape or portrait, with airplane mode on.

**Watch for:** the 64 inline SVGs make the file large. Check load time on the phone. If it drags, that's a signal to lazy-render diagrams rather than to split files.

**Time:** 2 hours.

---

# Stage 2 — Plan data and the Today screen

**Goal:** open the app, see today's session.

This is the architectural stage. Everything after it depends on getting the data shape right, so slow down here.

### The plan as data

Stop hand-writing the sessions in HTML. Define them once as a JS object and render from it. Otherwise every progression change means editing markup in five places.

```js
const PLAN = {
  monday: {
    name: "Upper body strength",
    warmup: "B",
    cooldown: "B",
    blocks: [
      { id: "pullup",   name: "Pull-ups",         type: "sets", sets: 4, target: "3–5 negatives" },
      { id: "pushup",   name: "Push-up board",    type: "sets", sets: 4, target: "two grips" },
      { id: "rod-row",  name: "Rod bent-over row", type: "sets", sets: 3, target: "10 reps", weight: true },
      { id: "rod-ohp",  name: "Rod overhead press", type: "sets", sets: 3, target: "8 reps", weight: true },
      { id: "facepull", name: "Band face-pulls",  type: "sets", sets: 2, target: "15 reps" }
    ]
  },
  // tuesday: run, wednesday: kettlebell, thursday: recovery,
  // friday: speed, saturday: run, sunday: rest
};
```

**Four block types, because your training isn't all sets and reps.** This is the thing generic trackers get wrong for you:

| Type | Logs | Used by |
|---|---|---|
| `sets` | reps and optional weight per set | pull-ups, push-ups, rod work |
| `complex` | rounds completed per side, bell weight | kettlebell complexes |
| `interval` | total minutes, run/walk stage | Tuesday and Saturday runs |
| `contacts` | reps and total foot contacts | plyometrics, sprints, cone drills |

### Tasks
1. Write the `PLAN` object covering all seven days.
2. Today screen reads `new Date().getDay()` and renders that day's blocks.
3. Each block row: name, target, and an expand control that reveals the diagram inline.
4. Header shows the warm-up and cool-down letter, tapping it jumps to that section of the manual.
5. Add a manual day override — a small date/day picker — for when you shift a session.

### Done when
Opening the app on a Wednesday shows the kettlebell session with the right complex, and tapping "Clean" expands the diagram without leaving the screen.

**Don't yet:** save anything. This stage is display only.

**Time:** 3 hours.

---

# Stage 3 — Logging and storage

**Goal:** tick things off, enter numbers, and have them still be there tomorrow.

### Storage shape

One key per session date. Simple, greppable, and trivially exportable.

```js
// key: "session:2026-09-14"
{
  date: "2026-09-14",
  day: "monday",
  completed: true,
  blocks: {
    pullup:  { sets: [5, 4, 4, 3] },
    pushup:  { sets: [18, 15, 14, 12] },
    "rod-row": { sets: [10, 10, 10], weight: 12 }
  },
  notes: "elbows felt fine",
  steps: 8400
}
```

Separate keys for standalone data: `steps:2026-09-14`, `settings`, `meta`.

### Tasks
1. Checkbox per block, plus per-set number inputs.
2. Use `<input type="number" inputmode="numeric">` — brings up the number pad, not the full keyboard. Matters more than it sounds when your hands are sweaty.
3. Auto-save on every change. No save button — you'll forget, mid-workout.
4. Prefill each field with last session's value for that block. Most sessions you're matching or adding one rep, so this removes nearly all typing.
5. **Export to JSON** — a download button in settings. Build this now.
6. **Import from JSON** — validate before merging, and keep a raw copy of anything that fails to parse rather than discarding it.

### Done when
You log a full Monday session, force-quit the app, reopen it, and everything is there. Export produces a file you can open and read.

**The one non-negotiable:** export works before you log your first real session. Storage on iOS is not durable.

**Time:** 3 hours.

---

# Stage 4 — Rest timer

**Goal:** a 90-second timer that's still correct when you come back to the app.

Small feature, one real trap.

### The trap
iOS Safari suspends JavaScript timers when the screen locks or you switch apps. A naive `setInterval` countdown will read 12 seconds when you unlock after 90.

**Fix:** store the timer's *end timestamp*, not a remaining count. Recompute from `Date.now()` on every tick and on the `visibilitychange` event.

```js
const endsAt = Date.now() + seconds * 1000;
// on tick and on visibilitychange:
const remaining = Math.max(0, Math.round((endsAt - Date.now()) / 1000));
```

### Tasks
1. Preset buttons matching the plan: 60, 90, 120 seconds, plus the 45–60 second drill rest.
2. Timestamp-based countdown with a `visibilitychange` listener.
3. A short sound or vibration on completion — `navigator.vibrate` is unreliable on iOS, so use a brief audio cue. It needs a user gesture to unlock audio on first use; unlock it when the timer starts.
4. Timer persists across tab switches inside the app.

### Done when
Start a 90-second rest, lock the phone, unlock at 90 seconds, and it reads zero or is already done.

**Time:** 1.5 hours.

---

# Stage 5 — Steps and history

**Goal:** see whether the plan is working.

### Tasks
1. Step entry: one number per day, on the Today screen. Manual entry — a web app can't read Apple Health.
2. Show the current week's step target from the six-week ramp, so you know whether 8,000 is on track or ahead.
3. History tab: a list of past sessions, tap to view what you logged.
4. Progression sparklines for the four metrics that matter: pull-up total reps, push-up total reps, kettlebell rounds completed, sprint rep count.
5. A simple streak or consistency view — sessions completed per week against the five scheduled.

Draw the sparklines yourself in SVG. A chart library is 200KB to render four twelve-point lines.

### Done when
After three weeks you can look at one screen and see whether pull-ups are moving.

**Time:** 3 hours.

---

# Stage 6 — Offline and install polish

**Goal:** it behaves like an app, not a bookmarked web page.

### Tasks
1. `manifest.webmanifest` — name, short name, `display: "standalone"`, theme colour matching the slate header, icons at 192px and 512px.
2. `sw.js` — cache the shell on install, serve cache-first.
3. **Cache versioning.** This is the trap that bites everyone: an installed PWA keeps serving the old shell after you push a fix. Put a version constant at the top of `sw.js` and bump it on every change to `index.html`.
4. Optional but worth it: a GitHub Action that fails the build if `index.html` changed and the `sw.js` version constant didn't. Fifteen lines of YAML, saves an hour of confusion later.
5. Test in airplane mode from a cold start.

### Done when
Airplane mode, cold launch from the home screen, full app works. Push a visible change, reload twice, and the change appears.

**Time:** 2 hours.

---

# Stage 7 — Optional, only if you still want it

Do none of these until you've used the app for a month. Most of them will turn out not to matter.

- **Photo journal** — meal or physique photos with no calorie analysis. Downscale to ~400px and use IndexedDB, not `localStorage` (which caps around 5MB).
- **Session notes** — a free-text field per session. Cheap, and often more useful than the numbers.
- **Deload prompt** — flag when three sessions in a row show declining reps.
- **Plan editor** — edit the `PLAN` object from the UI rather than in code. Only worth it if the plan is changing often.

---

## Build order rationale

Stages 0, 3 and 6 are the ones that fail silently if done late. Deployment problems compound with app bugs. Export built after you have real data means you risk losing it. Cache versioning added late means you spend an evening convinced your code is broken when it's just stale.

Stages 1, 2, 4, 5 are additive and can slip without consequence.

---

## Risks and mitigations

| Risk | Mitigation |
|---|---|
| iOS evicts local storage | Export weekly. Consider a monthly export committed to the repo. |
| Stale service worker serves old app | Version constant plus CI guard, from stage 6. |
| Single file grows unmanageable | Acceptable up to ~4,000 lines. Past that, split the `PLAN` object into a separate `plan.js`. |
| You stop using it | The real risk. The Today screen must open in one tap and need under five taps to log a full session. Optimise for that above everything. |
| Timer wrong after screen lock | Timestamp-based countdown, stage 4. |

---

## Testing checklist

Run before each deploy:

- Cold launch from home screen, airplane mode
- Log a full session, force-quit, reopen — data intact
- Export, clear site data, import — data restored
- Rest timer through a screen lock
- Portrait and landscape
- Every diagram renders

---

## What to reuse and what not to

From `lizardman472/tracker` (Rack-Free Tracker): read it for the single-file structure, the import validation and raw-recovery pattern, and the cache-version CI guard. **Don't copy code** — that repo shows no licence file, which under default copyright means you can read it but not reuse it. Fork it to your account as a reference copy, since small repos disappear.

Avoid the AGPL projects entirely unless you're happy publishing your source and changes.

---

## Suggested schedule

| Session | Stages | Hours |
|---|---|---|
| 1 | 0 and 1 | 2.5 |
| 2 | 2 | 3 |
| 3 | 3 | 3 |
| — | *Use it for a week* | — |
| 4 | 4 and 6 | 3.5 |
| 5 | 5 | 3 |

Stages 0–3 give you a usable app. Everything after is refinement.
