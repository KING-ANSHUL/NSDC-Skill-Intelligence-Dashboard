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

## Hard-won lessons (don't redo these mistakes)

- **Don't rename KPIs for "formal" wording.** The spreadsheet names are the source of truth. A past pass renamed ~25 KPIs and it was a real regression — always diff against the sheet before touching labels.
- **Don't invent a mock-data recording of real people.** The camera "footage" is a hand-drawn synthetic scene (`makeScene`/`drawScene`), never a downloaded video — a monitoring demo built on someone else's classroom recording is a privacy/copyright liability.
- **Test slide-fit at the actual presentation viewport**, not a small default one — dead space and overflow only show up at the real screen size (this dashboard was audited at 1536×1030).
- **Verify with Playwright, not eyeballing.** After any layout change, sweep every tab × every scope (India / state / one centre) and check for `NaN`/`undefined`, clipped slides, and page-level scroll. A `localhost:8731` static server is already configured for this in `.claude/launch.json`.
- **Duplicate sections creep in easily** — when adding a KPI zone, grep for its section title first; "Committed vs. Actual" and "Active Hours & Instructor Presence" each existed twice in the code at one point before being caught.

## Working on this project

- No build step — edit the HTML directly, then reload in the browser (`preview_start` with the `dashboard` launch config, or open the file directly).
- When making layout/content changes, always re-run the Playwright verification sweep described above before considering the change done.
- Prefer surgical Python string-replacement scripts (read the file, `assert count==1`, replace, write) over rewriting large blocks by hand — the file is 400+ KB and manual edits are error-prone.

## Keeping this file in sync (read this every session)

This file is the shared memory between everyone working on this project through Claude — it is committed to the repo, so a teammate's `git pull` is the only way they get context on what you did and why. If this file is stale, the next person starts from zero.

**Before ending any session that changed something meaningful** (a decision, a new mechanism, a lesson learned, a data source, a fixed bug worth remembering), update the relevant section above — then `git add CLAUDE.md`, commit, and push it along with the code change. A code diff without the "why" behind it is only half the context.
