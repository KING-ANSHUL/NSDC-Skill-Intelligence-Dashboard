# NSDC Skill Intelligence Dashboard

Camera-based analytics dashboard prototype for **PMKVY 5.0** — built as ConveGenius's capability pitch to NSDC (National Skill Development Corporation). Audience: NSDC's CTO, IAS officers, possibly a Minister, shown on a projector.

## The architectural rule (from Arnav, NSDC)

> "We will not depend on any kind of data that we need from the institute. We will depend only on two kinds of data — camera frames, and what they declared at accreditation time."

Everything in this dashboard is tagged by provenance: **Measured** (camera) / **Declared** (accreditation, locked once submitted) / **Modelled** (proposed detector, not yet built) / **Biometric** (AEBAS). Never invent a number without saying which of these four it is.

## Files

- `NSDC_Skill_Intelligence_Dashboard.html` — the only deliverable. Single self-contained file, vanilla JS, hand-rolled SVG charts, no build step, no external assets. Opens directly via `file://` or the local server.
- `.claude/launch.json` — serves this directory on `http://localhost:8731` (`python -m http.server`), needed because Playwright/browser tooling can't hit `file://` directly.

## Data sources baked into the dashboard

- **Real camera export**: `headcount_minute_202608081437.csv`, 35,152 rows, 8 cameras, 4 days (05–08 Aug 2026), 2-minute buckets, encoded as base64 in `RAW{}`. This is the *only* real dataset — one training centre (Greater Noida + Okhla), 6 courses.
- **KPI register**: `Copy of Mock Camera Analytics Dashboard - Prototype.xlsx` (in the user's Downloads) — the authoritative list of 37 live KPIs across 4 pages (Grounding & Master Data, Attendance & Punctuality, Training Quality, Assessments), plus 5 explicitly cut KPIs (marked red in the sheet: Trainer Coverage Ratio, Teacher Punctuality, Cohort Punctuality, Late Start, Early Finish). **KPI names and definitions must match this sheet exactly** — do not rename or reword them for "formality"; that has caused real regressions before.
- **National layer**: 1,148 mock centres generated at load from a seeded RNG (`hs()`/`rng()`), scattered inside real state outlines parsed from a simplemaps India SVG (`GEO{}`). This is the only synthetic dataset in the file, and it's explicitly documented as such in the code comments — never let it get confused with the one real centre's data.

## Structure of the dashboard code

Single `<script>` block, numbered sections (search for `· N ·` in comments):
1. Deterministic RNG, 2. Real feed decode, 3. Master data (centres/spaces/courses) + national geography, 4. Derivations, 5. Modelled layer, 6. DOM helpers, 7. Chart primitives, 8. Vision/camera engine, 9. Presentation layer (KPI cards, sections), 10. Shell (tabs, slide deck, render loop).

Key mechanisms:
- **`kcard()`** — the KPI card component. Every card carries a source dot (provenance), a value, optionally a camera button (`cam:{space,signal}`) that opens a live camera drawer under that exact card.
- **`CAMSIG{}`** — six camera "signals" (headcount, stations, instructor, trade, viva, feed) that answer a specific question when a KPI's camera button is clicked. The camera is evidence for a figure, not a standing section — don't reintroduce always-on camera bands.
- **Slide deck** (`buildDeck()`) — each tab's content is packed into slides sized to the viewport so nothing scrolls and nothing overflows the screen during a live presentation. A slide holding `.geo`/`.duo`/`.cg` claims the full frame; pure-card slides get `class="centered"` and space their sections evenly instead of leaving a dead band. This is presentation-mode behavior, not a normal scrolling page — be careful before "simplifying" it.
- **`duo()`** — composes two related blocks side by side (e.g. Training Quality + its selected-lab heatmap; Assessment Attendance + Proctoring Integrity) instead of stacking them as separate full-width sections. Preferred over adding a new top-level `sec()` when two blocks are conceptually related.
- **`heat({..., colorOf})`** — pass `colorOf(value, rowIndex)` and the heatmap scores each cell against a reference with the strict red→green ramp (`refCol`) instead of the magnitude-only blue ramp. Peak-occupancy cells score against 75% of that room's own enrolled roll; workstation utilisation against 70%. Omit `colorOf` and you get the old sequential ramp.
- **`hourBand(camId, date)`** — folds the real 2-minute series into 07:00–19:00 hour buckets (mean and peak per bucket) for the "Candidates by Hour Band" chart. Pure camera data, no timetable, so it stays inside the two-source rule.
- **`offStats(date)`** — splits missing feed inside a day's covered window into per-camera outages (server up, one camera dark = someone switched it off) and whole-centre outages (every camera dark = server or power down). Frequency across days is what separates intentional from technical, which is why the Server-Off card counts how many of the four days show it.
- **The `"ALL"` aggregate date** (section 4b, `ALLD`) — a synthetic date key that averages every camera's series slot-by-slot across the feed window, so every curve, hour band, scrubber and coverage figure works on it unchanged. Per-room derivations average **every numeric field generically** — do not reintroduce a hand-written field list, because any field it forgets crashes the page that reads it (`cov`, `anyMin` and `personMin` were each found that way). Peak on ALL is the average of the daily peaks, never a recomputed instantaneous max. `DAYLBL[ALLD]` must be set where `DAYLBL` is declared in section 9, not in 4b — 4b runs first and the const is not initialised yet.

## Hard-won lessons (don't redo these mistakes)

- **Don't rename KPIs for "formal" wording.** The spreadsheet names are the source of truth. A past pass renamed ~25 KPIs and it was a real regression — always diff against the sheet before touching labels.
- **Don't invent a mock-data recording of real people.** The camera "footage" is a hand-drawn synthetic scene (`makeScene`/`drawScene`), never a downloaded video — a monitoring demo built on someone else's classroom recording is a privacy/copyright liability.
- **Test slide-fit at the actual presentation viewport**, not a small default one — dead space and overflow only show up at the real screen size (this dashboard was audited at 1536×1030).
- **Verify with Playwright, not eyeballing.** After any layout change, sweep every tab × every scope (India / state / one centre) and check for `NaN`/`undefined`, clipped slides, and page-level scroll. A `localhost:8731` static server is already configured for this in `.claude/launch.json`.
- **Duplicate sections creep in easily** — when adding a KPI zone, grep for its section title first; "Committed vs. Actual" and "Active Hours & Instructor Presence" each existed twice in the code at one point before being caught.
- **Business-review overrides (2026-08-19) — do NOT "fix" these back to the KPI sheet.** The review meeting is newer than the sheet and explicitly changed: "Invigilator" → **Assessor** everywhere (trainers aren't allowed in the exam room; the external person is an assessor); the assessment denominator is **eligible** candidates (always ≤ enrolled, from SIDH later — mock for now), not enrolled; **Head-to-Machine Ratio removed outright** for practicals (group practicals are assessed together, so 1:1 is meaningless) with **no replacement ratio**; AEBAS is a **face-authentication** device, not fingerprint; asset checks are **Infra Readiness** (is the committed asset visible — yes/no), never a utilisation %, because bed-type assets are legitimately idle in some lessons. People-on-System benchmark is 1.2 — a reading near 1.1 is the assessor pausing behind a screen, not sharing.
- **The two-source rule is absolute.** KPIs may use only (a) what the centre declared at enrolment / assessment / accreditation time, which the camera then validates, and (b) camera video analytics. Never centre-supplied timetables or daily operational data — that can be managed or corrupted, which is the entire failure mode this product exists to catch. No KPI may depend on declared batch timings. Committed Training Hours is allowed: it is an accreditation-time commitment, not a daily timetable.
- **Batch sizes are illustrative.** 45 is the hard cap for every trade; sanctioned seats are drawn from {30, 35, 40, 45} and differ per batch. No SIDH lookup for now.
- **This dashboard is a historical (T−1) mockup and stays completely separate from the live dashboard** (the older, separately-built product). Its purpose is to demonstrate capability once SIDH and camera data arrive. Nothing here connects to a real feed, and no label may claim to be live.

## Provenance is the product (audit findings, 2026-08-19)

An adversarial audit of this file confirmed 30 defects, and the worst were all provenance lies rather than layout bugs. Treat these as standing rules:

- **A KPI's `src` must match where its number actually comes from.** Active Time %, Unsupervised Time %, Idle Time % and Instructor Presence Timeline were tagged `measured` while their values are computed from `MI`/`modelInstructor()` — the seeded model in section 5. No instructor detector exists. They are now `modelled`. Before tagging anything `measured`, trace the value back to `SER10`/the real export; if it passes through `MCH` or `MI`, it is modelled.
- **The camera drawer must not claim more than the space can deliver.** Its header now reads the KPI's own `src`, and says "no camera on this space" for `nc-1` (the Assessment Hall), which `SPACES` declares `role:"uncovered"` and the Coverage page reports as a coverage gap.
- **The "Measured only" toggle must actually do something.** Its CSS targets `.is-modelled`; `kcard()` now emits that class for modelled cards. If you add a new card kind, keep the class.
- **The provenance legend must be visible at presentation width.** It was hidden below 2080px — i.e. hidden at the 1536px this is actually shown at, leaving 57 dots with no key. It now lives in `.tabrow` (a flex wrapper around the tabs nav) rather than in the masthead, because the masthead has no room to spare: restoring the legend there pushed it 758px past its own width, and `overflow:hidden` had been silently clipping the stat strip for some time. **`.masthead` overflows invisibly — never trust it by eye.** Measure `scrollWidth - clientWidth` at 1536px after any masthead change. The strip now sheds States/Districts below 1700px and the whole `.mstat` below 1600px; `.crumb` is capped at 24ch with an ellipsis, `#fSpace` at 190px, `#fDay` at 150px.
- **Two-source enforcement caught a real breach:** "Active Class-Hours" compared against `committedCoveredMin`, a window derived from `COURSES[].start` — a declared daily batch start time. A centre that shifts its declared start would have repaired its own score. It now compares against `committedMin` (the accreditation-time hours-per-day commitment). `deriveDay()` still computes `lateStart`/`earlyFinish` from the same schedule fields — those are the register's red-cut KPIs and must never be surfaced.
- **Units must match the label.** `offStats` counted camera-minutes for Camera-Off and wall-clock minutes for Server-Off, rendering both as "min": 05 Aug read 714 min inside a 754-min window. Both are wall-clock now, with `camSlotMin` kept separately for the camera-minutes total.
- **Every aggregate must be honest about what it averaged.** `PARTIAL["ALL"]` unioned the daily windows and made Feed Availability read a flat 24.0 h against a real 18.8 h mean; `offStats("ALL")` read the averaged series (only −1 when all four days were dark) and reported zero outages beside a card saying 1,260 minutes without feed; `FEEDMETA["ALL"]` averaged slots but summed gaps. All three now average the real days. `MI["ALL"]` is still the latest day — averaging presence bits would invent a state that never occurred — so the strip's title says "latest day only" when the scope is ALL.

## Second audit batch (medium/low findings, 2026-08-19)

- **`bars()` calls `fmtV()` on null values.** It draws a null bar fine but still formats it, and `fmt(null)` throws — inside `SYNCS`' try/catch that swallows the error and leaves an empty chart with no console trace. Any `bars()` whose series can contain nulls needs `fmtV:x=>x==null?"…":fmt(x)` and a null-guarded `statusOf`. This is how the hour-band chart silently rendered nothing on the two partial days.
- **An unmeasured band is not a zero.** `hourBand()` keeps bands with no feed on the axis as nulls (labelled "no feed") instead of dropping them — dropping them made a partial export look like a full day that merely started late — and reports `pk:null` rather than a fabricated 0.0 peak.
- **A ceiling averaged is not a ceiling.** On `ALL`, `r.nightMax` is already a mean of daily maxima, so the overnight false-positive ceiling takes the true max across the four real days (2.0), not the max of the means (1.2).
- **`camLabel()` must not invent a camera.** `camLabel("nc-1")` used to print "CAM-NC-1" for a space with no camera; it now returns "no camera" for any id that is not `cam-*`.
- **The replay frame must not wear live grammar.** The camera drawer plays back a recorded export, so the badge says REPLAY with a static blue dot — never a pulsing red one.
- **`durNoLunch` was read but never produced**, so "continuous presence, lunch removed" printed the raw presence figure (4.8h). The lunch window is now actually subtracted (4.1h). If a caption claims a correction, grep that the field exists.

## Working on this project

- No build step — edit the HTML directly, then reload in the browser (`preview_start` with the `dashboard` launch config, or open the file directly).
- When making layout/content changes, always re-run the Playwright verification sweep described above before considering the change done.
- Prefer surgical Python string-replacement scripts (read the file, `assert count==1`, replace, write) over rewriting large blocks by hand — the file is 400+ KB and manual edits are error-prone.

## Keeping this file in sync (read this every session)

This file is the shared memory between everyone working on this project through Claude — it is committed to the repo, so a teammate's `git pull` is the only way they get context on what you did and why. If this file is stale, the next person starts from zero.

**Before ending any session that changed something meaningful** (a decision, a new mechanism, a lesson learned, a data source, a fixed bug worth remembering), update the relevant section above — then `git add CLAUDE.md`, commit, and push it along with the code change. A code diff without the "why" behind it is only half the context.

**This is enforced, not just a request:**
- A local git hook blocks any commit that touches `NSDC_Skill_Intelligence_Dashboard.html` without also staging `CLAUDE.md` **and** `SESSION_LOG.md`. One-time setup after cloning: `git config core.hooksPath .githooks` (only needs running once per clone).
- A GitHub Action (`.github/workflows/context-sync.yml`) re-checks the same rule on every push, server-side, regardless of whether the local hook was set up — so it catches it even on a fresh clone that skipped the one-time config.
- To intentionally bypass for a trivial change (typo, formatting), use `git commit --no-verify` — but default to updating both files, not to bypassing.

## SESSION_LOG.md — the substitute for syncing raw chat

Claude's actual conversation transcript lives only on the machine/account that ran it — it is not something git can or should sync (huge, includes internal tool-call noise, potentially sensitive). `SESSION_LOG.md` is the practical substitute: **every session that changes the dashboard appends a short entry** — what code changed, a 3-6 line summary of what was discussed/decided/rejected, and whether CLAUDE.md itself was touched. This is enforced by the same pre-commit hook and CI check described above.

A `post-merge` git hook (also enabled by the one-time `core.hooksPath` setup) fires automatically right after `git pull` and prints the new SESSION_LOG.md entries plus a code diff stat — so pulling doesn't just bring new lines of HTML, it tells you the story behind them without you having to go looking.
