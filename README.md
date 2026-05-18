# Bright Beginnings — New Teacher Onboarding

An interactive, self-paced onboarding web app for new Bright Beginnings teachers. Replaces the 91-slide Canva teacher manual with a guided, browser-based experience built around the **5 C's**: Culture, Connection, Compensation, Compliance, and Curriculum.

Single HTML file. No build step. Drop it on Netlify (or open it locally) and it works.

---

## What it does

- Walks new staff through **5 sequential modules** with ~45 accordion sections total
- Tracks per-section completion ("Mark as reviewed")
- Ends each module with a **5–7 question quiz** (80% to pass, 28 questions total)
- **Unlocks the next module** only when the previous one is fully complete (sections + quiz)
- Saves progress automatically — staff can close the tab and come back where they left off
- Celebration modal on each module completion with auto-advance
- Mobile responsive

---

## The 5 modules

| # | Module | What's in it |
|---|---|---|
| 1 | **Culture** | Mission, philosophy, 6 core values, locations, leadership & chain of communication, 4 stages of team, 8 tips for success, Disney-style customer service |
| 2 | **Connection** | Required paperwork, online licensing trainings (Better Kid Care, VA Preservice, etc.), GroupMe/ProCare/Tadpoles, PD days, weekly meetings, 5 Languages of Appreciation, Enneagram |
| 3 | **Compensation** | Biweekly payroll, benefits (medical, 401K, PTO, childcare tuition), deductions, hours, RTO process, paid holidays, $200 referral bonus |
| 4 | **Compliance** | Gossip policy, attendance, social media, dress code, cell phone policy, rules of conduct, ratios (incl. nap-time doubling), infant safe sleep, allergies/MAT, health policy, handwashing & cleaning, restorative behavior management, daily routine, resignation |
| 5 | **Curriculum** | Lesson planning, Tadpoles & iPad use, tour takeovers ($25 enrollment bonus), outdoor weather policy, Outdoor Classroom Project, classroom branding standards, parent communication |

---

## Tech

Plain HTML, CSS, and vanilla JavaScript. No frameworks, no build, no dependencies — just open the file in a browser.

- **Fonts**: Playfair Display + Inter (loaded from Google Fonts CDN)
- **Storage**: `localStorage` via an abstracted `StorageAdapter` class (see below)
- **Design tokens**: navy `#1a1a2e`, orange `#FF6B35`, teal `#4ECDC4`, cream `#F5F0E8`, purple `#9B59B6` — matching the BB design system

---

## Architecture

### Single source of truth: `MODULES` array

All content lives in one JavaScript array near the top of the `<script>` block. Each module has:

```js
{
  id: 'culture',
  number: 1,
  title: 'Culture',
  tagline: 'Who we are.',
  subtitle: '...',
  accent: 'orange',
  icon: '<svg>...</svg>',
  sections: [
    {
      id: 'mission',
      title: 'Our Mission',
      sub: 'Why we exist',
      accent: 'orange',
      icon: '<svg>...</svg>',
      body: `<p>HTML content...</p>`
    },
    // ...more sections
  ],
  quiz: [
    {
      q: 'Question text?',
      options: ['A', 'B', 'C', 'D'],
      correct: 2,  // 0-indexed
      feedback: 'Explanation shown after submission.'
    },
    // ...more questions
  ]
}
```

To edit content: change the data in `MODULES`. The UI re-renders from it.

### Storage adapter pattern

Progress is persisted through a `StorageAdapter` object — the UI only calls `load()`, `save(state)`, and `clear()`. Currently backed by `localStorage`. To swap in Firebase later, replace the three methods. There's a commented example in the file showing the swap.

```js
const StorageAdapter = {
  async load() { /* localStorage now → Firestore later */ },
  async save(state) { /* ... */ },
  async clear() { /* ... */ }
};
```

### App state shape

```js
appState = {
  currentModule: 0,            // or null when on home view
  sections: {                  // key: `${moduleId}::${sectionId}`
    'culture::mission': true,
    'culture::philosophy': true,
    // ...
  },
  quizzes: {                   // key: moduleId
    'culture': {
      passed: true,
      score: 5,
      total: 5,
      pct: 100,
      at: '2026-05-18T...'
    }
  },
  startedAt: null
}
```

### Progress logic

- A module is **complete** when all its sections are marked reviewed **and** its quiz is passed (≥80%)
- Module N is **unlocked** when module N-1 is complete (module 1 is always unlocked)
- Overall progress = total completed units / total units across all modules (where each module contributes `sections.length + 1` units)

---

## Running it

### Locally
Just open `teacher_onboarding.html` in any modern browser.

### Hosted
Drop it on Netlify, GitHub Pages, or any static host. No build step.

Suggested URL: `training.brightbeginnings.com`

---

## Editing content

**To change a section's text:** edit the `body` string in the `MODULES` array. Use the existing HTML helpers — `<ul class="check-list">`, `<ul class="x-list">`, `<div class="callout">`, `<div class="two-col">`, `<div class="stat-grid">`, `<div class="people-grid">`, `<div class="locations-grid">`.

**To add or change a quiz question:** edit the module's `quiz` array. Each question is `{ q, options, correct (0-indexed), feedback }`.

**To add a new section:** push a new object into the relevant module's `sections` array. The accordion will render automatically.

**To change the pass threshold:** edit `PASS_THRESHOLD` (default `0.8`).

---

## Roadmap / known gaps

- [ ] **Video embeds** — CSS includes a `.video-placeholder` component, but section bodies don't yet render any video tiles. Vimeo/YouTube URLs need to be plugged in (Better Kid Care, Handwashing 3:05, Infant Safety 5:46, Diapering 21:15, etc.). Cleanest path: add a `videos: [{title, duration, url, eyebrow}]` array to each section and render inside the accordion body.
- [ ] **Firebase backend** — swap `StorageAdapter` to Firestore for persistent, cross-device progress
- [ ] **Admin panel** — per-staff completion view (uses the same `appState` data already being saved)
- [ ] **Auth** — staff log in with their BB email so progress follows them across devices
- [ ] **Custom domain** — point `training.brightbeginnings.com` at the Netlify deploy
- [ ] **Print / certificate** — generate a completion certificate PDF when all 5 modules pass
- [ ] **Mentor sign-off** — separate "mentor confirms this section was covered in person" checkbox alongside the staff one

---

## Design system

Follows the Bright Beginnings personal design style guide (the same one driving `kppipeline.netlify.app/guide.html`).

- **Display font**: Playfair Display (700/800)
- **Body font**: Inter (300–900)
- **Cards**: white background, 22px border-radius, 5px left-border accent (rotates through orange/teal/navy/purple), subtle shadow, lift on hover
- **Section eyebrows**: 11px uppercase pills, accent color on tinted background
- **Heroes**: navy with decorative orange + teal pseudo-element circles, left-aligned 66px Playfair title
- **Alternating section backgrounds**: cream / white / navy

---

## Files

```
teacher_onboarding.html    # the whole app — HTML, CSS, JS in one file
README.md                  # this file
```

---

## License / use

Internal Bright Beginnings tool. Built for staff onboarding across our three sites (Crozet, Forest Lakes, Mill Creek).
