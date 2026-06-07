# BB Learning Platform — Session Handoff
## Start here for the next build session

---

## What this project is
Staff training platform for Bright Beginnings Preschool (Charlottesville, VA).
- **Live:** https://bbonboard.netlify.app
- **GitHub:** https://github.com/robhichens/bb-onboarding
- **Auto-deploys** from GitHub `main` via Netlify. No build step.

---

## Current state (as of this handoff)

### Files that exist and are deployed
| File | Purpose |
|---|---|
| `index.html` | Full onboarding app — auth, 5-module course, Firebase, ~250KB |
| `admin.html` | Admin dashboard — progress table, invite by email, CSV export |
| `logo.png` | BB logo (also base64-embedded in index.html) |
| `design-preview.html` | **LOCAL ONLY** — v2 design prototype, do not deploy |

### Assets folder
`assets/Brand and Elements/` — BB brand PNGs:
- `BB_Logo_fullColor-01.png` — full color logo (topbar)
- `BB_Logo_white.png` — white logo (dark backgrounds)
- `Tree_FullColor-01.png` — tree icon (coral/yellow/gray)
- `Copy of bird_coral-01.png` — single coral bird
- `Copy of bird_gold-01.png` — single gold bird
- `Copy of 3Birds_coral-01.png` — mama + 2 chicks (module hero decoration)
- + blue/gray variants of both bird styles

### What the current app does
- Staff log in with email/password (Firebase Auth)
- Create account: captures firstName, lastName, school, email
- 5-module onboarding course — sequential unlock, 80% quiz pass required
- Progress saved to Firestore `progress/{uid}`
- On completion: certificate modal ("Good job, {name}!")
- Admin panel: see all staff progress, invite by email (EmailJS), export CSV
- Admin access: set `role: "admin"` on a user's Firestore doc

---

## ✅ REBUILD PROGRESS (branch `rebuild-v2`, not yet pushed)

**Milestone 1 — DONE & verified in browser.** `index.html` was rebuilt to the approved
BB Learning Platform design + Option B multi-course shell. The live site (`main`) is untouched.

What's working:
- New design system CSS (Lora + Inter, coral/yellow/sky/charcoal tokens) — matches `design-preview.html`.
- Mobile app shell: white logo topbar, course-picker home, charcoal module heroes (blobs + birds),
  step-circle progress strip (no % bar), restyled accordion sections, shame-free quiz
  ("Great thinking." / "Let's look at that together.", no red, no score), achievement screen, bottom nav.
- `COURSES` array (Option B). Course 1 = existing 5 onboarding modules (content spliced verbatim).
  Course 2 shows as a locked "Virginia Licensing Standards" teaser card on Home.
- **Progress is a flat `sections{}`/`quizzes{}` namespace keyed by globally-unique module ids** →
  Firestore schema + `admin.html` stay compatible (no migration). StorageAdapter unchanged in shape.
- Firebase auth/signup/reset preserved. Logo now references `assets/.../BB_Logo_fullColor-01.png`
  (dropped the 100KB base64 blob → file 250KB → 148KB).
- Quiz now gated: unlocks only after all sections in a module are reviewed.
- Bonus panels wired: Progress, Quick Ref (ratios), Help.

Verification: drove the real engine + real StorageAdapter (localStorage stub) through
home → course → module → section-gating → quiz pass/fail → achievement → unlock → persistence. No console errors.

How to preview locally: `preview-test.html` is a gitignored harness (Firebase stubbed to localStorage,
auto-signs-in) — open it to click through without real auth. `index.html` is the real deploy file.
Backup of the old app: `index.html.bak` (gitignored).

**Milestone 2 — DONE & verified.** Full `LICENSING_COURSE` authored from `scope.md`:
4 modules (Roles & Qualifications / In the Classroom / Playground & Outdoors / Credentials & Training Clock),
accents orange/teal/purple/navy, 6/6/4/6 sections, **10 questions each (40 total) — all numbers verbatim**
(8VAC20-780), subtle citation pills. Added to `COURSES = [ONBOARDING_COURSE, LICENSING_COURSE]`;
Home now renders it as a real sky-accented card, locked until Onboarding is complete. Verified:
unlock gating, ratios table (1:4/1:5/1:8/1:10/1:18/1:20), "no waiver" callout, and the 8/10 pass threshold
(7/10 fails, 8/10 passes).

**Milestone 3 — DONE & verified.**
- **Situation hooks:** all 45 onboarding sections now have a tailored Lora-italic `hook:` (scenario-first opening, BB voice). Engine renders `s.hook` when present; licensing keeps its per-module-first-section hooks.
- **admin.html:** fully restyled to the new design system (charcoal topbar + white logo, Lora titles, coral/sky/yellow accents) and **extended for both courses** — grouped table header (Onboarding / Licensing), per-module badges, per-course % + overall %, CSV includes both courses. Counts driven by `COURSE_DEFS` (onboarding `[9,7,7,15,7]`, licensing `[6,6,4,6]`). Verified with seeded staff: math correct (e.g. 12/50 onboarding steps = 24%, overall 12/76 = 16%), no console errors.

**Still TODO:**
1. Push `rebuild-v2` → `main` to deploy (only when Rob says go). **Main is currently untouched.**
2. Optional: thin onboarding/licensing overlap later (separate task per scope.md §3).

---

## Original plan — THE BIG REBUILD

### Three things happening simultaneously:
1. **Full redesign** — new BB Learning Platform design system (approved)
2. **Option B architecture** — multi-course shell, course picker home screen
3. **Course 2** — Virginia Licensing Standards (4 modules, full content spec in `scope.md`)

### Design system (APPROVED — match `design-preview.html` exactly)

```css
:root {
  --coral:       #F08782;
  --yellow:      #FFD437;   /* large fills: use #FFEEA0 */
  --sky:         #AEDFE5;
  --mid-gray:    #A7A9AC;
  --lt-gray:     #D1D2D4;
  --dk-gray:     #6D6E71;
  --charcoal:    #545454;
  --cream:       #FAFAF5;
  --coral-soft:  #FAE8E7;
  --yellow-soft: #FFF8D6;
  --sky-soft:    #E8F7F9;
}
```

**Fonts:** `Lora` (Google) — ONLY for module titles, situation hooks, achievement headings (italic 400 / 600 weight). `Inter` (300–700) — everything else.

**Key component decisions:**
- Topbar: white bg, `BB_Logo_fullColor-01.png` at ~34px height, avatar right
- Module hero: charcoal bg, coral blob top-right (op 0.11), yellow blob bottom-left (op 0.08), `3Birds_coral-01.png` bottom-right (op 0.16)
- Situation hook: yellow-soft bg, 4px yellow left border, Lora italic 400 19px, gold bird accent
- Key fact callout: sky-soft bg, 4px sky left border
- Progress: step circles (never a % bar) — coral=done, coral-outline=current, lt-gray=upcoming
- Quiz correct: yellow-soft bg + "Great thinking." (#6d4c00)
- Quiz wrong: sky-soft bg + "Let's look at that together." (#2a7a82) — **NEVER red**
- Achievement: #FFEEA0 bg, tree bottom-right (mix-blend-mode: multiply), coral bird icon
- Bottom nav (mobile): 5 tabs — Home / Modules / Progress / Quick Ref / Help

**Shame-free rules (enforced throughout):**
- Never "Incorrect" / "Wrong" / "Try again" / red color in quiz
- Never show a score or percentage anywhere
- Progress label always: "You've completed Step X of Y" or "Module X of 5"

### Option B architecture
```js
const COURSES = [
  {
    id: 'onboarding',
    title: 'New Staff Onboarding',
    subtitle: '...',
    modules: [ /* existing 5 modules */ ]
  },
  {
    id: 'licensing',
    title: 'Virginia Licensing Standards',
    subtitle: 'What the Commonwealth requires of every classroom and playground.',
    modules: [ /* 4 modules from scope.md */ ]
  }
];
```

Home screen shows both course cards with progress pips. Separate Firestore keys per course so progress doesn't collide.

### Course 2 — Licensing Standards
Full content spec: `C:\Users\rober\Downloads\scope.md`
- 4 modules: Roles & Qualifications / In the Classroom / Playground & Outdoors / Credentials & Training Clock
- 10 quiz questions each (40 total) — all verbatim in scope.md
- Regulatory source: 8VAC20-780 — **never alter any number**
- Pass threshold: 0.8 (8/10)
- Module accents: orange / teal / purple / navy

### Onboarding content retrofit (Option B, full)
Every section body needs a **Situation Hook** opening — Lora italic card, yellow-soft bg. The existing section bodies are plain HTML paragraphs; each needs a scenario-first hook added at the top before the existing content.

---

## Suggested build order

```
Step 1: Redesign shell (CSS tokens, new components, topbar, bottom nav)
Step 2: Refactor to COURSES array + course picker home screen
Step 3: Retrofit onboarding section bodies with situation hooks
Step 4: Build licensing course content (4 modules, 40 questions from scope.md)
Step 5: Update admin.html to new design
Step 6: Push, test, deploy
```

---

## Technical details to preserve

### Firebase
- SDK: compat v9.6.0 CDN — `firebase.auth()` / `firebase.firestore()` globals
- Config in the `<script>` block at top of index.html — don't move to module
- `onAuthStateChanged` gates everything — app doesn't initialize until user is signed in
- `StorageAdapter.save()` includes email + serverTimestamp on every write

### Quiz engine
- `PASS_THRESHOLD = 0.8`
- `quizAnswers = {}` resets on module re-entry
- `submitQuiz()` grades, saves to `appState.quizzes[moduleId]`
- `showCelebrate(modIdx)` — if final module: shows certificate modal; else: shows standard celebration

### Admin
- Module section counts array: `[8, 6, 7, 15, 7]` — update if section counts change in redesign
- EmailJS: service `service_vzv701i`, template `template_wvq9wp9`, key `sxTxSw0jkVLUIWPpr`

### Git
- Repo connected: `origin = https://github.com/robhichens/bb-onboarding.git`
- Branch: `main` — Netlify auto-deploys on push
- `design-preview.html` should be gitignored or just never pushed (it's local reference only)

---

## Rob's working style notes
- Show a preview before full builds — he'll give quick clear feedback
- "Push it" / "love it" / "go ahead" = green light, no need to ask again
- Commit messages: descriptive, not ceremonial
- He iterates fast — small targeted edits beat large rewrites where possible
