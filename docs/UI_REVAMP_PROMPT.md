# UI revamp prompt — NSDC Skill Intelligence Dashboard

Two prompts. Paste **Prompt 1** into Claude (Design canvas / a fresh session) with
`NSDC_Skill_Intelligence_Dashboard.html` attached or in the working directory. Use **Prompt 2** for the
implementation pass once the mockups are approved.

Everything between the `BEGIN PROMPT` / `END PROMPT` markers is the prompt — copy it whole.

---

# PROMPT 1 — visual redesign (mockups + spec, no code yet)

`--------------------------------- BEGIN PROMPT ---------------------------------`

You are redesigning the visual system of a single-file analytics dashboard that already works. This is
a **re-skin, hierarchy and chart-form pass — not a rebuild**. Every number, KPI name, tab and
interaction stays. I want artboards and a spec first; no code in this pass.

## 1. What the product is

A camera-based analytics dashboard prototype for **PMKVY 5.0**, built as ConveGenius's capability pitch
to **NSDC** (National Skill Development Corporation). It is shown on a **projector** to NSDC's CTO, IAS
officers, possibly a Minister. It is a **historical (T−1) mockup** — nothing is live, and no label may
ever claim to be live.

The single idea the design must sell is **provenance**. Every figure is tagged as one of four kinds, and
the audience has to read that tag from across a room:

| Tag | Meaning |
|---|---|
| **Measured** | computed from real camera frames |
| **Declared** | submitted at accreditation / enrolment time, locked once submitted |
| **Modelled** | a proposed detector that **does not exist yet** — a promise, not a measurement |
| **Biometric** | AEBAS face-authentication device |

The architectural rule behind the product: *"We will depend only on two kinds of data — camera frames,
and what they declared at accreditation time."* No centre-supplied timetables, no daily operational
data — those can be managed or corrupted, which is the exact failure mode this product exists to catch.
The design must make that discipline look like the feature it is, not a footnote.

## 2. The data actually behind the screen (this constrains chart form)

- **One real dataset**: a camera export from one training centre — 35,152 rows, **8 cameras, 4 days
  (05–08 Aug 2026)**, 2-minute buckets. Two of the four days are **partial** — the feed does not cover
  the full window, so some hour bands have **no data at all**.
- A synthetic **national layer** of 1,148 mock centres, used only for the map, funnel and rank rail.
- Everything else is either a declared constant or output of a seeded model.

Two consequences you must design around:

1. **A four-point trend is not a trend line.** Smoothed sparklines over 4 days imply a curve the data
   cannot support. Prefer **4 labelled dots on a shared axis** (a dot strip) wherever a "trend" is shown.
2. **An unmeasured band is not a zero.** Any hour band, day or room with no feed must render as a
   **hatched gap labelled "no feed"** — never a bar of height zero, never silently dropped.

## 3. Current visual system — read it before changing it

The CSS is one `<style>` block, lines 7–1018, with a stated direction: *"institutional instrument panel
— light paper plane, deep navy ink, hairline modular grid, dark inset CCTV panels as the visual anchor.
No card-soup, no shadows-as-hierarchy, no neon."*

Existing `:root` tokens:

- planes — `--paper #f1f4f7`, `--surface #fbfcfd`, `--surface-2 #f6f8fa`
- ink — `--ink #0e1720`, `--ink-2 #46586a`, `--muted #5f6d7a` (4.9:1 on surface; it carries real copy,
  not just chrome)
- lines — `--hair #e4e8ed` (gridline), `--rule #ccd4dd` (axis/baseline), `--ring rgba(14,23,32,.10)`
- chrome accent — `--accent #175a9e`, `--accent-soft #e8f0f8`
- categorical series — `--s1..--s8`, a validated order, **never cycled**
- sequential blue ramp — `--q1..--q6`, magnitude only
- status, reserved and never used as a series — `--good #0ca30c`, `--warning #fab219`,
  `--serious #ec835a`, `--critical #d03b3b`
- dark video plane — `--vid #0a0f16`, `--vid-2 #111b26`, `--vid-ink #dfe8f2`, `--vid-muted #7e94aa`
- RYG reference ramp — `--ryg0..--ryg5`, used by `heat()` when a cell is scored against a reference
- type — Segoe UI Variable Display stack, body 13px/1.45, `.micro` = 10px uppercase 0.1em tracking

Keep this **direction**; sharpen its execution. If you change the palette, keep the token names and
re-validate: body copy ≥ 4.5:1, series colours ≥ 3:1 against `--surface`.

## 4. Structure you are designing into

Five tabs (`PAGES`): **Overview · Attendance · Quality · Assessments · Coverage**.
18 sections, **57 KPI cards**, 17 chart panels. Chrome from the top: `.masthead` (dual NSDC/MSDE
lockup, scope crumb, space + day selects, clock, mock badge) → `.tabrow` (tabs + the four-dot provenance
legend) → `.filterbar` → `.deck`.

Scope switches between **India → a state → one centre**; the same tabs re-render at each level.

Components, by real class name — redesign these, do not replace them:

- **`.kc` (`kcard()`)** — the KPI card. Slots: provenance dot, label, big value `.kc-v`, optional delta
  `.kc-d`, comparison bar `.kc-cmp` (observed bar + declared marker), 3-part stacked bar `.kc-tri`,
  sparkline `.kc-spark`, meter `.km`, status chip `.kc-chip`, footnotes, and a **Camera Signals** button
  — the last four all inside a wrapping footer `.kc-f`. Modelled cards also carry `.is-modelled`, which a
  "Measured only" toggle dims.
- **`.kg n{2..6}`** — the card grid. Rows run 2, 3, 4, 5 and 6 cards across. At `n6` a card is ~245px
  while a status chip plus the Camera Signals button need ~300px — **the widest rows (`n6` Per Room,
  `n5` Training Quality) are the worst crowding in the UI and you must solve them.**
- **`.camdrawer`** — camera evidence, opening **under the exact card** whose button was clicked. Dark
  `--vid` plane, hand-drawn synthetic scene (never real footage), a **REPLAY** badge with a *static*
  blue dot, a scrub timeline `.tl-*`, signal rows `.sigrow`. Header states the calling KPI's own
  provenance and says "no camera on this space" for the uncovered Assessment Hall.
- **`.duo()`** — two conceptually related blocks side by side.
- **`.hm` / `heat()`** — two different ramps: **RYG scored against a reference** and **blue magnitude**.
  They must not be confusable at a glance.
- **`.geo` / `.mapstage` / `.mrail`** — national map + rank rail.
- **`.cg` / `.cx()`** — chart grid and chart panel; each panel has a Table toggle. Charts are hand-rolled
  inline SVG (`svg.viz`), no library.
- also: `.strip`, `.tl`, `.fun` (funnel), `.vk`, `.cov`, `.mstat`, `.verdict`, `.chip`, `.legend`.

## 5. Non-negotiable constraints

1. **One self-contained HTML file.** Vanilla JS, hand-rolled SVG, no build step, no external asset, no
   CDN, no icon font, no webfont file. Anything you design must be reachable with plain CSS + inline SVG.
2. **A presentation deck, not a scrolling page.** `body{overflow:hidden}`. `buildDeck()` packs each tab
   into slides sized to the viewport so **nothing scrolls and nothing overflows** during a live demo.
   Design at **1536 × 1030** and state per artboard that it fits. Slides holding `.geo`/`.duo`/`.cg`
   claim the full frame; pure-card slides are `.centered` and space their sections evenly. **Denser is
   fine; taller is not.**
3. **KPI names are frozen.** They come from an external KPI register (37 live KPIs over 4 pages, plus 5
   explicitly cut ones). A past pass renamed ~25 labels "for formality" and it was a real regression. Do
   not reword a single label, chip or section title unless I ask. Type, weight, size, position: yours.
4. **Business-review overrides that look like bugs but are not:** "**Assessor**", never "Invigilator";
   the assessment denominator is **eligible** candidates, not enrolled; **there is no Head-to-Machine
   ratio** and no replacement for it; AEBAS is **face-authentication**, not fingerprint; asset checks are
   **Infra Readiness** (is the committed asset visible — yes/no), **never a utilisation %**.
5. **Nothing may read as live.** No pulsing dots, no LIVE pill, no ticking-red grammar, no fake-realtime
   motion. The clock and the REPLAY badge are the only time affordances.
6. **The provenance legend must be visible at 1536px**, and it lives in `.tabrow`, not the masthead — the
   masthead has zero spare width and its `overflow:hidden` has silently clipped content before. If your
   design adds anything to the masthead, say what it displaces.
7. **Accessibility:** keyboard tab nav, visible focus rings, `aria-selected` on tabs,
   `prefers-reduced-motion` honoured, and **no colour-only encoding** — the RYG heatmaps and the four
   provenance tags each need a second channel (shape, hatch, glyph or label).

## 6. Global form rules I want you to adopt

**(a) Let form carry provenance, not just colour.** Propose a system where the *shape* of the mark says
where the number came from — e.g. Measured = solid fill, Declared = an outline or reference rule,
Modelled = a hatched/screened fill, Biometric = its own glyph. Then colour is a second channel and the
a11y requirement is satisfied by construction. A Modelled card must look **different in kind**, not just
a different hue — and it must never wear a precise-looking delta arrow or a tight meter, because
precision implies measurement.

**(b) Never a pie, donut, 3D or needle gauge.** One exception, argued below: People-on-System, where
labelled bands *are* the message.

**(c) A card whose only content is the denominator of another card should be merged into it.** I count
roughly **12 card slots** recoverable this way (marked **MERGE** below). That recovered space is the
budget for real hierarchy — a primary number per slide that reads in four seconds.

**(d) Comparison beats repetition.** Where three cards hold three numbers on the same axis (round
durations, the committed trio, the triangulation), one panel with a shared scale says more in less space.

**(e) Reference marks must be labelled with their source.** A declared/committed reference rule reads
"committed at accreditation" — **never** "scheduled" or "as per timetable". There is no timetable in this
product and a caption implying one is a breach of the two-source rule.

## 7. Per-KPI visual brief

For each KPI below: what the number actually is, and the form I want you to design for it. Where I mark
**MERGE**, fold the card into its neighbour. Where I mark **HERO**, that graphic should dominate its
slide. Push back with a better form if you have one — but say why.

### Overview → Headcount Overview  (`kg4`, measured)

- **Avg Attendance** — mean of per-space averages, learners. → Big number + **4-day dot strip**, not a
  smoothed spark. Print the denominator ("mean of per-space averages") on the card face; a mean-of-means
  is a subtle claim and should not hide in a tooltip.
- **Peak Occupancy** — busiest 2-minute mean. → Big number + dot strip, **plus a faint reference tick at
  the enrolled roll**. The question in the room is always "peak against what?".
- **Active Classrooms** — *n* of *m* rooms. → **Unit chart**: one glyph per room, filled = active. A meter
  over a countable set of ~12 is strictly worse than showing the 12.
- **Active Class-Hours** — observed hours vs hours committed at accreditation. → **Bullet chart**;
  committed drawn as a **full-height rule**, not a hairline tick. Note: this KPI had a real two-source bug
  where its colour was driven by a schedule-derived ratio. Its colour must come from the committed
  comparison and nothing else.

### Overview → Center Identity & Permitted Sectors  (declared)

- **Accreditation Validity** — days left. → **Countdown bar with labelled bands** (expired / <90d /
  valid). A bare meter cannot tell an officer whether 240 days is good.
- **Sector Authorization Count** — count of permitted sectors. → Number + the **sector names as small
  chips** beneath. The declared list is short and far more informative than its cardinality.
- **Declared Space Count** — rooms. → Number + **unit glyph row split covered / uncovered**, which makes
  it read directly against the next card.
- **Camera Coverage Ratio** — covered spaces vs declared spaces. → **Unit chart of spaces** with the ratio
  as caption. One space (the Assessment Hall) is deliberately uncovered and reported as a coverage gap —
  a percentage hides that story, nine glyphs with one hollow tells it.

### Overview → Enrollment & Capacity  (declared, `kg4`)

All four cards are one fact stated four ways.

- **Total Enrolled Headcount** → keep, as a **stacked capacity bar**: enrolled fill + vacant remainder,
  with all three readouts labelled on the bar.
- **Total Sanctioned Capacity** → **MERGE** (it is the bar's total).
- **Center-Wide Vacant Seats** → **MERGE** (it is the remainder segment).
- **Center-Wide Enrollment Utilization Rate** → **MERGE** (it is fill ÷ total; show as the bar's caption).
  Net: 4 cards → 1 card, **3 slots freed.**

### Overview → Trade Concordance  (mostly modelled)

- **Trade Concordance Rate** — do the trades seen match what was allocated. → **Split bar of matching vs
  non-matching rooms** over the countable 12, in modelled/hatched treatment. Not a %.
- **Trades Not Matching Allocation** — count. → Show **which rooms**, as named chips. A count of 2 begs
  the question the chips answer.
- **Spaces Without a Session** (measured) → **Unit chart**, dark glyph = no session detected.
- **Mean Classifier Confidence** (modelled) → **This is the most misleading number on the page as a bare
  mean.** Confidence is a distribution: show a **mean with a min–max whisker**, or a small histogram.

### Attendance → Command Summary  (measured, `kg4`)

- **Cohort Detection Count** — detected vs expected cohorts. → Bullet.
- **Avg Cohort Duration** — continuous presence with the lunch window removed. → Number + dot strip, and
  **show the correction**: two stacked segments, raw presence and the removed lunch band. A caption that
  claims a correction should be visible as one (this exact caption once printed the uncorrected figure).
- **Avg Cohort Headcount** — while the cohort is in the room. → Dot strip.
- **Peak Headcount Trend** — day on day. → **Trend-primary card variant**: the dot strip is the main
  element and the number is secondary. This card *is* a trend; invert the normal card anatomy for it and
  spec that variant.

### Attendance → Active Hours & Instructor Presence  (all modelled, `kg4`)

- **Active Time %** / **Unsupervised Time %** / **Idle Time %** → three slices of one whole shown as three
  cards. → **One stacked 100% bar card + legend**, hatched (modelled). **2 slots freed**, and the
  reader finally sees that they sum.
- **Instructor Presence Timeline** → **HERO of this section.** Intrinsically temporal, currently reduced
  to a count. Design a **gantt strip across the instructional day**: present / absent bands, with the
  merged brief step-outs drawn as hairline notches so the merging rule is visible. Give it the width the
  two merged cards just freed.

### Attendance → Committed vs. Actual  (declared, `kg3`, all comparison)

- **Committed Training Hours** (h/day) · **Committed Headcount** (learners) · **Committed Trainer Count**
  (trainers) → all three are the same form: **bullet chart, committed as the reference rule, observed as
  the bar.** Use identical bar geometry across the three so the eye compares *shortfall depth* directly.
  Reference label reads "committed at accreditation".

### Attendance → Triangulated Attendance  (`kg3`, three different provenances)

- **Enrolled** (declared) · **Biometric Attendance Count** (AEBAS) · **Camera Headcount** (measured, peak
  seen) → **HERO — the single best visual opportunity in the dashboard.** Three cards each holding one leg
  of a triangulation should be **one slope / dumbbell chart**: three dots on a shared count axis, each in
  its provenance treatment, with both gaps labelled (declared→biometric, biometric→camera). That one
  graphic *is* the product thesis. Make it the anchor of the Attendance tab. **2 slots freed.**

### Attendance → Per Room · Active, Unsupervised, Idle  (`kg6` — the ~245px crowding)

- **Per-room Active/Unsupervised/Idle** (modelled, one card per room) → **small multiples**: 12 identical
  mini stacked bars with room short-codes in a single panel. Removing the card frame removes the 245px
  problem entirely rather than negotiating with it.
- **Peak, Selected Day** · **First Cohort Detection** · **Avg Cohort Duration** · **Active Time %** →
  these are four facts about *one selected room*, not four peer KPIs. → **A "room dossier" definition
  list**: labelled rows, tabular numerals, right-aligned values. Cards imply parallel peers and mislead
  here. **3 slots freed.**

### Quality → Training Quality  (`kg5`, all modelled)

- **Instructor Interactivity Ratio** — moving or at the instruction zone. → Ratio with no natural 100%:
  give it a **drawn benchmark rule** or drop the meter. A meter without a real denominator is a lie about
  one.
- **Equipment Utilization Rate** → meter is acceptable, but the per-trade spread matters more than the
  mean — pair it visually with the existing by-trade chart so they read as one statement.
- **Zone Utilization Rate** (footnote "Per-station occupancy") → **dot plot per station**. Note: the cut
  KPI "Head-to-Machine" must not reappear in any label or caption here.
- **Workstation Utilization Rate** — computer labs, scored against a **70%** reference. → Meter **with the
  70% reference drawn**, not implied.
- **Infra / Asset Readiness** — are the committed assets visible in frame, **yes/no**. → A **checklist of
  per-asset glyphs**. Never a %, never a meter: bed-type assets are legitimately idle in some lessons.
  Visually it must be **categorically distinct** from its four neighbours, or a viewer will read it as a
  rate — which is exactly the misreading the business review removed.
- **Classroom Interactivity Ratio** — where there is no equipment. → Same benchmark treatment as its
  equipment twin, so the pair reads as a pair.

### Assessments → Assessment Attendance  (`kg4`)

- **Assessment Attendance Rate** → Bullet against **eligible** candidates, with the word "eligible"
  **on the card face**. That override is otherwise invisible and reads as a bug to anyone who knows the
  old definition.
- **Vivas Conducted** — *n* of *m*. → Unit chart if the denominator is small, bullet if not.
- **Viva Completion Rate** — ≥ 5 minutes counts as complete. → **Split bar complete / incomplete** with
  the 5-minute threshold in the caption. The threshold *is* the definition.

### Assessments → Proctoring Integrity Signals  (`kg2`)

- **People-on-System Ratio** — benchmark **1.2**; a reading near **1.1** is the assessor pausing behind a
  screen, not candidates sharing. → **The one licensed band scale**: a horizontal axis with 1.0 / 1.2 and
  the breach zone marked, the reading as a marker. Interpretation is the entire content of this KPI, so
  the bands must be on screen — a bare "1.1" invites the wrong conclusion.
- **Assessor Presence Rate** — % of the exam window. → Meter, with the **window length stated** beside it.

### Assessments → Round Durations  (`kg3`, modelled)

- **Theory Duration** (pen-and-paper) · **Average Viva Duration** (per candidate–assessor pair) ·
  **Practical Duration** (machine-based demonstration) → three bare numbers on the same unit. → **One
  panel: three horizontal bars on a shared minute axis**, each with its declared round length as a
  reference rule. **2 slots freed**, and the comparison becomes free.

### Coverage → Feed Integrity  (`kg4` + 2, measured / declared)

- **Camera Coverage Ratio** → the **same unit chart** as on Overview, pixel-identical, so the repeat is
  recognised rather than re-read.
- **Feed Availability** (h of 24) → **a 24-hour band strip, not a meter.** The *shape* of the window is
  the information, and there is already a "Camera Availability Window" chart on this page — make the card
  a true miniature of it.
- **Minutes Without Feed** (07:00–19:00, all cameras) · **Camera-Off Minutes** (some cameras dark while
  others run — someone switched one off) · **Server-Off Minutes** (every camera dark at once — power or
  server) → one decomposition split across three cards. → **One stacked bar of the covered window**: feed
  present / camera-off / server-off, all in wall-clock minutes. Then add the thing that actually
  distinguishes intentional from technical: **a 4-dot glyph showing how many of the four days show
  server-off.** Today that lives in prose; it should be a mark. **2 slots freed.**
- **Overnight Baseline Reading** (21:00–06:00, persons) — the false-positive floor. → Number annotated
  with the **true max across the four days**, not the max of the daily means; on the `ALL` aggregate a
  mean-of-maxima understates the ceiling (1.2 vs the real 2.0).

### Coverage → Circulation and Open Areas  (`kg3`, measured, one card per area)

- **Per-area busiest time** → "busiest" is a *time*, not a magnitude. → **A mini time-of-day strip per
  area** with the peak marked, instead of a number.

### The 17 chart panels

Give each a short spec, and apply these rules across all of them:

- **Nulls render as hatched "no feed" gaps** on the axis, never as zero-height bars and never dropped —
  dropping them made a partial export look like a full day that merely started late.
- **Sort categorical comparisons by value, not alphabetically**, and draw the reference rule.
- **The two heatmap ramps must be instantly distinguishable.** Suggested: the blue magnitude ramp gets a
  continuous legend; the RYG reference-scored ramp gets **discrete labelled steps** ("under reference /
  at / above"), so nobody reads a red cell as merely "a low number".
- The Table toggle on every panel is an honesty feature — design its affordance to be findable, not
  hidden.
- Panels to spec: *Distribution across scope · Enrolment Against Sanctioned Capacity · Observed Peak
  Against Enrolled Roll · Enrolment, Biometric and Camera Compared · Biometric Claim Less Camera
  Observation · Peak Occupancy by Space and Day · Peak Occupancy Trend · Headcount Across the
  Instructional Day · Candidates by Hour Band · Instructor Presence by Trade · Interactivity Index by
  Trade · Equipment Utilisation by Trade · Workstation Utilisation by Hour · Observed Present Against
  Eligible Candidates · Assessor Presence by Trade · Camera Availability Window · Movement Index Across
  the Day.*

## 8. Beyond the cards

- **Hierarchy.** With ~12 slots freed, propose a **primary / secondary / tertiary** card treatment and say
  which KPI is primary on each of the five tabs. An officer glancing for four seconds should land on it.
- **The dark camera plane** is meant to be the anchor — the moment the room understands the product is
  when a drawer opens *under a number*. Design that moment deliberately: the transition, the alignment to
  the calling card, the REPLAY badge, the signal rows.
- **Slide rhythm.** Per tab, say where a slide wastes a vertical band and where it is cramped; and how
  `.centered` even-spacing plays against full-frame slides.
- **Masthead + filterbar** are two chrome rows before any content. Can they carry the same information in
  less height without losing the dual lockup, the scope crumb or the mock badge?
- **Projector legibility.** 1536px wide, viewed from 6+ metres, washed colour, imperfect focus. State your
  minimum type size, stroke weight and minimum mark size for that condition, and apply it.

## 9. Explicitly do not

- No card-soup, no shadow-as-hierarchy, no neon, no gradient hero band, no glassmorphism.
- No dark-mode flip of the whole dashboard — the light paper plane is the institutional register; the dark
  plane is reserved for video.
- No emoji, no decorative icons. Institutional, not consumer-SaaS.
- No new dependency, framework, chart library or font file.
- No new KPI, no new data, no invented number. If a layout wants a figure that does not exist, say so
  instead of filling the hole.

## 10. Deliverables for this pass

1. **Artboards at 1536 × 1030** — one per tab (5), plus: a **card-anatomy board** showing all four
   provenance treatments × the primary/secondary/tertiary hierarchy, a **camera-drawer-open** board, and
   boards for the three HERO graphics (Triangulated Attendance slope chart, Instructor Presence gantt,
   Feed Integrity decomposition).
2. **A token table** — every `:root` value kept, changed or added, with the measured contrast ratio of
   each text and series colour against its actual background.
3. **Component specs** for `.kc` (all slots + the trend-primary variant), `.kc-f`, `.kg n2..n6`,
   `.camdrawer`, `.hm` (both ramps), `.cx`, `.tabrow`, `.masthead` — sizes, spacing, and states
   (default / hover / focus / selected / modelled).
4. **A merge ledger** — the card merges you accept, slots freed, and what you spent them on.
5. **Before/after on the three worst spots** you find, each with the reason it was wrong.
6. **An implementation note** mapping every change to the existing class names, flagging anything that
   touches deck-fit behaviour — those need re-measuring, not eyeballing.

Ask before assuming anything about the KPI register, the two-source rule, or the deck mechanism. Start by
reading the file and telling me the three things you would change first, and why.

`---------------------------------- END PROMPT ----------------------------------`

---

# PROMPT 2 — implementation pass (after the mockups are approved)

`--------------------------------- BEGIN PROMPT ---------------------------------`

Implement the approved redesign in `NSDC_Skill_Intelligence_Dashboard.html`. Every constraint from the
design brief still applies: one file, no build, no external assets, frozen KPI names, nothing reads as
live, the deck must fit 1536 × 1030 with no page-level scroll.

**Method**

- Edit with surgical Python string-replacement scripts (read file → `assert count == 1` → replace →
  write). The file is 440 KB; hand-editing large blocks has caused real breakage.
- Prefer changing `:root` tokens and existing rules over adding classes. If a mechanism exists
  (`kcard()`, `duo()`, `heat({colorOf})`, `hourBand()`, `bars()`, `CAMSIG`), reuse it — do not add a
  parallel one.
- Any new card kind must still emit `.is-modelled` for modelled data, or the "Measured only" toggle
  silently stops working.
- `cx()` nests the built node inside an unclassed wrapper div, so a fill-the-frame block needs
  `height:100%`, not just `flex:1`.
- `bars()` formats null values even when it draws them fine, and `fmt(null)` throws inside a try/catch
  that swallows it — any series that can contain nulls needs `fmtV:x=>x==null?"…":fmt(x)` and a
  null-guarded `statusOf`.
- Before tagging anything `measured`, trace the value to the real export. If it passes through `MCH` or
  `MI`, it is **modelled**.

**Verification before calling it done** — Playwright against `http://localhost:8731`
(`.claude/launch.json`, config `dashboard`):

- Sweep **every tab × every scope** (India / a state / one centre) × the `ALL` aggregate day.
- Check for `NaN` / `undefined` in rendered text, clipped slides, and any page-level scroll.
- Wait **~500 ms after every slide change** before measuring — the 440 ms entrance animation inflates
  `scrollHeight` by 4–9 px and `DECK_GO(i)` restarts it. Nineteen phantom "overflowing slides" across
  three scopes were once entirely this artifact.
- When measuring overflow, **exclude SVG descendants and scrollable subtrees**: SVG child rects are not
  clipped by their `svg` (a map measured 671 px past its panel) and a 1,148-row table inside
  `overflow:auto` measures 45,000 px tall while scrolling harmlessly.
- Measure `.masthead` `scrollWidth - clientWidth` at 1536 px. It overflows invisibly.
- Confirm the provenance legend is visible at 1536 px, and that every Camera Signals button in the widest
  card rows is fully inside its card.

Then update `CLAUDE.md` and append to `SESSION_LOG.md` in the same commit — a pre-commit hook and a
GitHub Action both block a commit that touches the HTML without staging both.

`---------------------------------- END PROMPT ----------------------------------`
