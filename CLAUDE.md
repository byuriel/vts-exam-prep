# CLAUDE.md — VTS Anesthesia Exam Prep

This file orients a Claude Code session working on this project. Read it fully before making changes.

## What this project is

A standalone study app for the **AVTAA VTS (Veterinary Technician Specialist) Anesthesia & Analgesia** certification exam. It is a static, single-page web app that works two ways from the same code:
- **iPhone PWA** — installs to the home screen via Safari → Share → Add to Home Screen (runs full-screen with its own icon).
- **Browser app** — just visit the deployed URL on any computer.

Built for a veterinary technician (referred to in prior work as "Sunny Dee" / Dimari) preparing for the exam. It is a *sibling* project to a separate app called **VTS Compass** (application-prep tool) — do not confuse the two. This is the exam-practice app only.

## Tech stack (deliberately minimal — do not add a build system)

- **Plain static files.** No Vite, no webpack, no npm build step.
- **React 18 + ReactDOM + Babel-standalone**, all loaded from CDN (cdnjs) in `index.html`.
- `app.jsx` is transformed **in the browser** by Babel-standalone at runtime (`<script type="text/babel">`).
- **No bundler, no node_modules, no package.json.** Keep it this way. The whole point is that it deploys as static files with zero build.
- Styling is inline JS style objects + a small `<style>` block with CSS variables in `index.html`. No Tailwind, no CSS framework.

## Files

| File | Role |
|------|------|
| `index.html` | App shell. Loads React/ReactDOM/Babel from cdnjs, mounts `app.jsx` into `#root`. Has iPhone PWA meta tags. |
| `app.jsx` | **The entire app** — all logic + the full question bank inlined as JS arrays. ~800 lines + question data. This is the only file you normally edit. |
| `manifest.json` | PWA manifest (name, icons, theme) for home-screen install. |
| `icon-180.png`, `icon-192.png`, `icon-512.png` | App icons (violet compass). |
| `README.md` | User-facing description + deploy/install steps. |

There is also a **single-file build** delivered separately as `vts-exam-prep.html` (index.html + app.jsx inlined into one file). That is a convenience artifact; the canonical source is the multi-file folder. If you change `app.jsx`, you can regenerate the single file by inlining `app.jsx` into `index.html`'s babel script tag (and stripping the `manifest`/`apple-touch-icon` links).

## Architecture of app.jsx (read before editing)

Top-to-bottom structure:

1. **`PRACTICE_POOL`** — array of 246 practice questions.
2. **`EXAM_SETS`** — array of 6 arrays of question IDs. Each is a **fixed, dedicated 40-question exam with ZERO overlap** between exams (240 unique questions used). This is intentional — see "Critical invariants" below.
3. **`MOCK_POOL`** — array of 100 questions **reserved exclusively for the 2 mock exams**. These must NEVER appear in practice exams, topic practice, or adaptive mode. Held-out set = honest mock scores.
4. Question shape: `{ id, domain, q, options:[4 strings], answer:0-3, explain }`.
5. Helpers: `shuffle()`, `shuffleOptions()` (shuffles A/B/C/D and re-tracks the correct index), `buildSet()`.
6. Config constants: `EXAM_SIZE=40`, `MOCK_SIZE=50`, `ADAPTIVE_SIZE=25`, `PASS_THRESHOLD=80`, `NUM_PRACTICE=6`, `NUM_MOCKS=2`, `MOCK_SECONDS=3600` (60 min).
7. **Supabase cloud autosave** — `cloudSave()` / `cloudLoad()` / `cloudFetch()` / `mergeStores()`. Uses the `progress` table, row id `exam_main`. Falls back to `localStorage` (key `vts_exam_prep_v3`) then in-memory if offline. (Shares the Supabase project with VTS Compass but uses a **different row id**, so they don't collide.) Cross-device safety: load and every return-to-foreground (`visibilitychange`/`focus`) **merge** cloud + local via `mergeStores()` (best scores = max per key, missedCounts = max, in-progress mock = whichever attempt is further along) — never blind last-write-wins, so a stale suspended device can't clobber newer progress from another device. Mid-mock saves go through `cloudSaveThrottled()` (localStorage every tick, cloud POST at most every 5 s, flushed with `keepalive` on backgrounding). The ☁️ badge turns to ⚠️ Offline when Supabase is unreachable or rejects a write (`res.ok` is checked).
8. Components: `Compass`, `Ring`, `Quiz` (handles practice/topic/adaptive AND timed pausable mocks with resume), `Results`, `Review`, `App` (controller).
9. Mounts with `ReactDOM.createRoot(document.getElementById("root")).render(<App />)`.

## Features (behavior that must be preserved)

- **6 practice exams** — each a fixed dedicated set (`EXAM_SETS`), answer positions shuffled every attempt. Practice readiness = average of best score across all 6.
- **Topic practice** — 15 fresh questions from a chosen domain (pulls from `PRACTICE_POOL`, randomized).
- **Adaptive Review** — 🔒 locked until all 6 practice exams completed once. Weights previously-missed questions heavier (`missedCounts`), mixes in a few fresh. Focuses on misses.
- **2 Mock exams** — 🔒 locked until all 6 practice exams completed once. 50 questions from `MOCK_POOL` (held-out). **Timed (60 min)**, **pausable**, and **timer + progress persist** (saved to cloud/local as `mockProgress[id]`) so she can exit and resume exactly where she left off. Separate **mock readiness** score. Results show time used + time paused (honesty).
- **Two readiness rings** on home: Practice vs. Mock. The gap between them is the intended "true readiness" signal.
- **Feedback toggle** (practice only): explanation after each question, or all at the end. Mocks never show feedback until the end.
- **Cloud autosave** across devices; ☁️ sync badge on home.

## Critical invariants — do NOT break these

1. **`EXAM_SETS` must stay non-overlapping.** The 6 practice exams are dedicated, distinct sets. If you add/remove questions, re-partition so each exam is 40 unique with zero cross-exam overlap. There is a validation approach in "Common tasks" below.
2. **`MOCK_POOL` questions must never leak into practice.** Practice exams, topic practice, and adaptive mode pull ONLY from `PRACTICE_POOL`. Mocks pull ONLY from `MOCK_POOL`.
3. **No build system.** Do not convert to Vite/CRA/Next. Do not add `import` of npm packages into `app.jsx`. Babel-standalone transforms it in-browser; ES module `import` will break it.
4. **No `localStorage`/`sessionStorage` assumptions as the only store** — cloud (Supabase) is primary with local fallback; keep both paths.
5. **Every question must be well-formed**: unique `id`, non-empty `q`, exactly 4 `options`, `answer` in 0–3, non-empty `explain`.
6. **Content accuracy matters** — this is veterinary board-exam material. Do not invent clinical facts. If unsure about a clinical detail, say so rather than guessing.

## Deployment (Vercel, static)

The repo deploys as **static files, framework preset "Other"** (no build command, no output dir needed).

Standard push-to-deploy flow (Vercel auto-deploys on push to `main`):

```bash
git add .
git commit -m "describe change"
git push
```

If Vercel doesn't pick up the push, force a fresh deploy:

```bash
git commit --allow-empty -m "trigger deploy"
git push
```

If Vercel ever mis-detects the framework and tries to build: Vercel → project → Settings → Build & Development Settings → set **Framework Preset = Other**, clear build/output commands, Save, then redeploy.

### First-time Vercel import (if not yet connected)
1. Push this folder to a GitHub repo (files at the repo root, not in a subfolder).
2. vercel.com → Add New → Project → import the repo.
3. **Set Framework Preset to "Other"** on the config screen.
4. Deploy.

## Common tasks

### Verify exam-set integrity after editing questions
Run a quick node check (node is fine for validation even though the app itself has no build):

```bash
node -e '
const fs=require("fs");
const c=fs.readFileSync("app.jsx","utf8");
const P=eval(c.match(/const PRACTICE_POOL = (\[[\s\S]*?\]);\s*\nconst EXAM_SETS/)[1]);
const S=eval(c.match(/const EXAM_SETS = (\[[\s\S]*?\]);\s*\nconst MOCK_POOL/)[1]);
const M=eval(c.match(/const MOCK_POOL = (\[[\s\S]*?\]);\s*\n/)[1]);
const flat=S.flat();
const overlap=[...new Set(flat.filter((v,i)=>flat.indexOf(v)!==i))];
console.log("Practice:",P.length,"Mock:",M.length,"Exams:",S.map(e=>e.length).join(","));
console.log("Cross-exam overlap:",overlap.length?overlap.join(","):"NONE");
const pids=new Set(P.map(q=>q.id)), mids=new Set(M.map(q=>q.id));
console.log("Mock leaks into practice ids:",[...mids].filter(id=>pids.has(id)).join(",")||"NONE");
const bad=[...P,...M].filter(q=>!q.id||!q.q||!Array.isArray(q.options)||q.options.length!==4||typeof q.answer!=="number"||q.answer<0||q.answer>3||!q.explain);
console.log("Malformed:",bad.length);
'
```

### Add new practice questions and re-partition the 6 exams
1. Append well-formed questions to `PRACTICE_POOL`.
2. Re-generate `EXAM_SETS` by dealing questions round-robin by domain into 6 balanced buckets of 40, ensuring zero overlap (see the validation above). Need ≥240 practice questions for six non-overlapping 40s.
3. Run the integrity check.

### Regenerate the single-file `vts-exam-prep.html`
Inline `app.jsx` into the `<script type="text/babel">` tag of `index.html`, and remove the `manifest` + `apple-touch-icon` `<link>` tags.

## Testing before you push

There is no automated test suite. Sanity-check by:
1. Running the node integrity check above (compiles the arrays, checks invariants).
2. Opening `index.html` in a real browser (or the single-file HTML) and clicking through: a practice exam, the feedback toggle, and — after simulating 6 completed exams in storage — the mock timer/pause/resume.
3. Confirming the ☁️ sync badge appears (Supabase reachable) and scores persist on reload.

Note: the in-browser Babel transform means a syntax error shows only at runtime (blank screen + console error), not at "build" time — so always load it in a browser after editing.

## Style / tone for changes

- Keep the violet "space" theme (bg `#0c0a14`, violet `#7c3aed`→`#a855f7`, green for pass, amber/red for lower scores).
- Mobile-first: large tap targets, single-column, readable type.
- Prefer small, surgical edits to `app.jsx` over rewrites.
- This is real board-exam study material for someone's career — accuracy and clarity over cleverness.
