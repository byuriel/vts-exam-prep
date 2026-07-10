# VTS Anesthesia Exam Prep — iPhone & Browser App

A complete study system for the AVTAA VTS (Anesthesia & Analgesia) certification exam. Installs on the iPhone home screen and also runs in any browser. Scores sync across devices via the cloud.

## The study system (designed like real board prep)

**346 board-level questions total**, split into two walled-off pools:
- **246 practice questions** — the training ground. The **6 practice exams are now fully distinct** (40 unique questions each, zero overlap between exams). Answer positions still shuffle every attempt so she learns concepts, not positions.
- **100 reserved mock questions** — never shown in practice, so the mock exams are a true "cold" test

### 1. Practice Exams (6)
- 40 questions each, **a fixed dedicated set per exam** (no overlap between the 6 exams), with **answer positions shuffled every attempt** so she can't memorize positions
- Toggle feedback: explanations after each question, or all at the end
- **Practice readiness** = average of her best score across all 6

### 2. Adaptive Review  🔒→🎯
- **Unlocks after she completes all 6 practice exams once**
- Focuses on the questions she gets wrong most often (spaced-repetition weighting), mixing in a few fresh ones
- Mastered questions fade from the rotation automatically

### 3. Mock Exams (2)  🔒→🎓  — the honest gauge
- **Unlock after all 6 practice exams are done** (same gate as adaptive)
- 50 questions each, drawn from the reserved pool she's never seen
- **Timed** (60 minutes) with a visible countdown for real pacing pressure
- **Pausable** any time — and the timer + progress **save**, so she can close the app and resume exactly where she left off
- No feedback until the end; results show time used and time paused (honesty)
- **Mock readiness** is tracked **separately** from practice. The gap between her practice % and her cold mock % is the single most honest signal of test readiness.

### 4. Topic Practice
- Focused 15-question sets on any of the 12 domains, fresh each time

## Domains covered
Pharmacology · Physiology · Equipment · Monitoring · Pain Management · Local/Regional Anesthesia · Emergency/CPR · Calculations · Species-Specific · Complications · Ventilation · Fluid Therapy · Anesthetic Principles

## Two ways to use it

- **iPhone:** open the deployed URL in Safari → Share → **Add to Home Screen**. Runs full-screen with its own icon.
- **Browser:** just visit the URL on any computer. Same app, same cloud-synced scores.

## Cloud autosave
Best scores and in-progress mock state save automatically to Supabase, so progress follows her between phone and computer. If offline, it falls back to local storage and still works.

## Files / deployment

**Easiest:** deploy the single file `vts-exam-prep.html` anywhere (or just open it). One self-contained file.

**Best for home-screen install (custom icon):** deploy this whole folder. On Vercel, set Framework Preset to **"Other"** (static HTML, no build step).

| File | Purpose |
|------|---------|
| `index.html` | App shell |
| `app.jsx` | App logic + all 310 questions |
| `manifest.json` | Home-screen install metadata |
| `icon-180/192/512.png` | App icons |

## One honest note
This is rigorous, comprehensive, exam-realistic prep — but no tool can *guarantee* a pass. Paired with the official reading list and her clinical experience, it will get her walking in genuinely prepared. The practice-vs-mock gap is there precisely to tell the truth about readiness before test day does.
