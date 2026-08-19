# Business-Review Dashboard Fixes Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Apply every correction and new-KPI request from the 2026-08-19 business-team review to `NSDC_Skill_Intelligence_Dashboard.html` — terminology fixes, eligible-denominator, assessor rename, illustrative 25–45 batch sizes, infra readiness, time-bucketed occupancy, actionable instructor pattern, RYG heatmaps, camera-off vs server-off split, an All-days date view, and historical T-1 framing — all on mock data, nothing live. The mockup stays completely separate from the old live dashboard; its purpose is capability demonstration.

**Architecture:** The dashboard is one self-contained 428 KB HTML file with a single `<script>`. All edits are made through surgical Python string-replacement scripts (`assert count==1` on every anchor, then replace, then write) run from the scratchpad — never hand-edit the file. Verification is a Playwright sweep against `http://localhost:8731` (static server config already in `.claude/launch.json`), not eyeballing. The repo's pre-commit hook requires `CLAUDE.md` **and** `SESSION_LOG.md` staged in any commit that touches the dashboard HTML, so each commit step below stages all three.

**Tech Stack:** Vanilla JS + hand-rolled SVG (no build step) · Python 3 patch scripts · Playwright MCP for verification · git with `.githooks` enforcement.

**Meeting decisions this implements (and their authority):** The KPI sheet is normally the naming source of truth, but this business review is newer and explicitly overrides it in five places: (1) "Invigilator" → "Assessor" (trainer is not allowed in the exam room; the external person is called an assessor), (2) assessment denominator is **eligible** candidates, not enrolled, (3) Head-to-Machine Ratio for practicals is **removed outright** (group practicals are assessed together, so 1:1 is meaningless — and per the follow-up, no Candidate-to-Assessor ratio either), (4) AEBAS is a **face-authentication** device, not fingerprint, (5) equipment presence is an **Infra Readiness** check (asset visible or not), never a utilisation % for bed-type assets, (6) the **two-source rule is absolute**: KPIs may use only accreditation/enrolment/assessment-time declared data and camera analytics — never centre-supplied timetables or daily data (it can be managed/corrupted), so no KPI may depend on declared batch timings, (7) the dashboard is **historical (T-1)**: everything renders for data up to the previous day, never live, (8) three exam types exist — practical (machine), computer-based (desktop), theory (pen-and-paper). These deviations must be recorded in CLAUDE.md so nobody "fixes" them back to the sheet later.

---

## Clarifications applied (2026-08-19 follow-up from Anshul)

- Batch sizes: illustrative, sanctioned drawn from {30, 35, 40, 45}, may differ per batch; 45 is the hard cap for every trade. No SIDH lookup for now.
- Eligible candidates: always ≤ enrolled; comes from SIDH later, mock for now.
- Reviewer's "slide" for the Overview: **ignore it**; only the spoken "ongoing trainings" count goes in.
- Three exam types: practical (machine), computer-based (desktop), theory (pen-and-paper). Do **not** split assessments into sections yet.
- **Candidate-to-Assessor Ratio: dropped.** Head-to-Machine (practical): removed outright, replaced by nothing.
- Two-source rule is absolute: no KPI may depend on centre-supplied timetables/batch timings. Committed Training Hours stays (it is accreditation-time declared commitment, not a daily timetable) — flagged as a clarifying question below in case that reading is wrong.
- Historical T-1 mode only: the dashboard renders data up to the previous day; nothing claims to be live. This mockup stays completely separate from the old live dashboard.
- Multi-date: build an "All days" aggregate view; every KPI/visualization must also work per single date.
- RYG thresholds: not defined by the business yet — chosen here for meaningfulness (documented per chart).

---

## File Structure

- Modify: `NSDC_Skill_Intelligence_Dashboard.html` — the only deliverable; all functional changes land here
- Modify: `CLAUDE.md` — document the five sheet-overrides (Task 10)
- Modify: `SESSION_LOG.md` — one entry per commit (Tasks 5, 9, 11)
- Create: patch scripts `q1.py` … `q9.py` in the session scratchpad (`C:\Users\ANSHUL~1\AppData\Local\Temp\claude\C--KSK\<session>\scratchpad\`) — throwaway, never committed
- This plan: `docs/superpowers/plans/2026-08-19-business-review-fixes.md`

**Before starting:** confirm the static server is up (`preview_start` with the `dashboard` config or `python -m http.server 8731` in `C:\KSK`) and `git status` is clean.

---

### Task 1: Terminology — face-auth device and Assessor

**Files:**
- Modify: `NSDC_Skill_Intelligence_Dashboard.html` (~lines 4114, 4132, 4362, 4380, 4386)

- [ ] **Step 1: Verify the anchors still exist (each must appear exactly once)**

```bash
cd /c/KSK
grep -c 'foot:"fingerprint device"' NSDC_Skill_Intelligence_Dashboard.html          # expect 1
grep -c 'the fingerprint device and the camera' NSDC_Skill_Intelligence_Dashboard.html  # expect 1
grep -c 'lbl:"Invigilator Presence Rate"' NSDC_Skill_Intelligence_Dashboard.html    # expect 1
grep -c 'cx("Invigilator Presence by Trade"' NSDC_Skill_Intelligence_Dashboard.html # expect 1
grep -c 'an invigilator was in frame' NSDC_Skill_Intelligence_Dashboard.html        # expect 1
```

- [ ] **Step 2: Write and run the patch script**

Write to scratchpad as `q1.py`, then run `python q1.py`:

```python
# -*- coding: utf-8 -*-
# Meeting corrections: AEBAS is a face-authentication device, not fingerprint
# ("don't keep things those are outdated"), and the exam-room person is an
# assessor, never an invigilator (trainers are not allowed in the room).
F = r'C:\KSK\NSDC_Skill_Intelligence_Dashboard.html'
s = open(F, encoding='utf-8').read()
def rep(a, b, n=1):
    global s
    assert s.count(a) == n, ('anchor', a[:70], s.count(a))
    s = s.replace(a, b)

rep('foot:"fingerprint device"', 'foot:"face-authentication device"')
rep('the fingerprint device and the camera',
    'the face-authentication device and the camera')
rep('lbl:"Invigilator Presence Rate"', 'lbl:"Assessor Presence Rate"')
rep('cx("Invigilator Presence by Trade"', 'cx("Assessor Presence by Trade"')
rep('an invigilator was in frame', 'an assessor was in frame')

open(F, 'w', encoding='utf-8').write(s)
print("q1 ok — bytes", len(s))
```

Expected output: `q1 ok — bytes <~428400>`

- [ ] **Step 3: Verify in the browser**

Navigate Playwright to `http://localhost:8731/NSDC_Skill_Intelligence_Dashboard.html`, then evaluate:

```js
() => { setScope({lvl:"ctr",st:"BR",ctr:"BR-01"});
  S.tab="assessment"; S.slide=0; render();
  const t = document.getElementById("view").textContent;
  const bad = [];
  if (/invigilator/i.test(t)) bad.push("invigilator still present");
  if (!t.includes("Assessor Presence Rate")) bad.push("assessor card missing");
  S.tab="attendance"; render();
  const t2 = document.getElementById("view").textContent;
  if (/fingerprint/i.test(t2)) bad.push("fingerprint still present");
  return bad; }
```

Expected: `[]`

*(No commit yet — Tasks 1–4 commit together in Task 5, because the pre-commit hook requires CLAUDE.md + SESSION_LOG.md alongside any HTML change and we batch those.)*

---

### Task 2: Eligible denominator + centre-wide batch clarity

**Files:**
- Modify: `NSDC_Skill_Intelligence_Dashboard.html` (assessment page ~4326–4392; Committed Headcount note ~4090)

- [ ] **Step 1: Verify anchors**

```bash
cd /c/KSK
grep -c 'decTxt:totC+" enrolled"' NSDC_Skill_Intelligence_Dashboard.html   # expect 1
grep -c 'cx("Observed Present Against Enrolled Roll"' NSDC_Skill_Intelligence_Dashboard.html  # expect 1
grep -c 'candidates enrolled for it' NSDC_Skill_Intelligence_Dashboard.html # expect 1
grep -c 'Enrolled headcount for each course against the headcount the camera detected' NSDC_Skill_Intelligence_Dashboard.html # expect 1
```

- [ ] **Step 2: Write and run `q2.py`**

```python
# -*- coding: utf-8 -*-
# Only ELIGIBLE candidates go forward to assessment (that list comes from the
# assessment sheet / app, not the enrolment register) — the denominator and
# every label around it changes. Also: enrolled totals are centre-wide; no
# single batch may read above the 45-seat cap, and the card says so.
F = r'C:\KSK\NSDC_Skill_Intelligence_Dashboard.html'
s = open(F, encoding='utf-8').read()
def rep(a, b, n=1):
    global s
    assert s.count(a) == n, ('anchor', a[:70], s.count(a))
    s = s.replace(a, b)

# --- Assessment Attendance Rate card ---
rep('decTxt:totC+" enrolled"', 'decTxt:totC+" eligible"')
rep('''    kcard({lbl:"Assessment Attendance Rate",src:"modelled",val:pc(totP/totC*100),
      tone:rygTone(totP/totC*100,85),''',
    '''    kcard({lbl:"Assessment Attendance Rate",src:"modelled",val:pc(totP/totC*100),
      tone:rygTone(totP/totC*100,85),
      note:"Candidates detected on assessment day against the ELIGIBLE list — only eligible candidates proceed to assessment, and that list comes from the assessment data sheet, not the enrolment register.",''')

# --- the eligible-vs-present chart ---
rep('''    cx("Observed Present Against Enrolled Roll",
      [{k:"Enrolled",c:CSSV("--q2")},{k:"Present",c:SER[0]}],
      W=>columns({W,cats:A.map(a=>a.code),series:[
        {k:"Enrolled",c:CSSV("--q2"),v:A.map(a=>a.candidates)},''',
    '''    cx("Observed Present Against Eligible Candidates",
      [{k:"Eligible",c:CSSV("--q2")},{k:"Present",c:SER[0]}],
      W=>columns({W,cats:A.map(a=>a.code),series:[
        {k:"Eligible",c:CSSV("--q2"),v:A.map(a=>a.candidates)},''')
rep('()=>mkTable(["Course","Enrolled","Present","Rate"],',
    '()=>mkTable(["Course","Eligible","Present","Rate"],')
rep('"Candidates detected present on assessment day against candidates enrolled for it."',
    '"Candidates detected present on assessment day against the eligible list for that assessment."')

# --- Committed Headcount: centre-wide, batch cap ---
rep('note:"Enrolled headcount for each course against the headcount the camera detected in that same session."',
    'note:"Enrolled headcount for each course against the headcount the camera detected in that same session. Centre-wide total across all six batches — no single batch exceeds the 45-seat cap."')

# --- Illustrative batch sizes: sanctioned drawn from {30,35,40,45}, varying
#     per batch; enrolled <= sanctioned and always above each camera's busiest
#     2-minute mean (CPA~24, ELE~15, FIT~17, WLD~7, RSA/SOL dark) ---
rep('{code:"CPA",name:"COPA",                   sector:"IT-ITeS",      enrolled:28,sanctioned:30,',
    '{code:"CPA",name:"COPA",                   sector:"IT-ITeS",      enrolled:34,sanctioned:40,')
rep('{code:"ELE",name:"Electrician",            sector:"Power",        enrolled:27,sanctioned:30,',
    '{code:"ELE",name:"Electrician",            sector:"Power",        enrolled:31,sanctioned:35,')
rep('{code:"FIT",name:"Fitter",                 sector:"Capital Goods",enrolled:26,sanctioned:30,',
    '{code:"FIT",name:"Fitter",                 sector:"Capital Goods",enrolled:26,sanctioned:30,')
rep('{code:"WLD",name:"Welder",                 sector:"Capital Goods",enrolled:19,sanctioned:25,',
    '{code:"WLD",name:"Welder",                 sector:"Capital Goods",enrolled:19,sanctioned:30,')
rep('{code:"RSA",name:"Retail Sales Associate", sector:"Retail",       enrolled:21,sanctioned:25,',
    '{code:"RSA",name:"Retail Sales Associate", sector:"Retail",       enrolled:28,sanctioned:35,')
rep('{code:"SOL",name:"Solar Technician",       sector:"Power",        enrolled:14,sanctioned:25,',
    '{code:"SOL",name:"Solar Technician",       sector:"Power",        enrolled:27,sanctioned:45,')
# eligible must sit strictly at-or-below enrolled
rep('const cand=Math.round(c.enrolled*(0.9+g()*0.1));',
    'const cand=Math.min(c.enrolled,Math.round(c.enrolled*(0.82+g()*0.15)));')

open(F, 'w', encoding='utf-8').write(s)
print("q2 ok — bytes", len(s))
```

- [ ] **Step 3: Verify — no per-batch number above 45, "eligible" wording live**

Playwright evaluate:

```js
() => { setScope({lvl:"ctr",st:"BR",ctr:"BR-01"});
  S.tab="assessment"; S.slide=0; render();
  const t = document.getElementById("view").textContent;
  const bad = [];
  if (!t.includes("eligible")) bad.push("eligible missing");
  if (t.includes("Enrolled Roll")) bad.push("old chart title");
  // batch-size cap: every per-course figure must be <= 45
  const over = COURSES.filter(c => c.enrolled > 45 || c.sanctioned > 45);
  if (over.length) bad.push("batch over 45: " + over.map(c=>c.code));
  return bad; }
```

Expected: `[]`

---

### Task 3: Remove Head-to-Machine outright · People-on-System benchmark language

**Files:**
- Modify: `NSDC_Skill_Intelligence_Dashboard.html` (~4326–4362)

- [ ] **Step 1: Verify anchors**

```bash
cd /c/KSK
grep -c 'lbl:"Head-to-Machine Ratio"' NSDC_Skill_Intelligence_Dashboard.html  # expect 1
grep -c 'const wm=A.filter' NSDC_Skill_Intelligence_Dashboard.html            # expect 1
grep -c 'tone:maxPos>1.05' NSDC_Skill_Intelligence_Dashboard.html             # expect 1
```

- [ ] **Step 2: Write and run `q3.py`**

```python
# -*- coding: utf-8 -*-
# Reviewer: group practicals are assessed 3-at-a-time, so a 1:1 head-to-machine
# KPI for practical assessments is wrong — remove it outright (follow-up:
# no Candidate-to-Assessor ratio either). A People-on-System reading near 1.1
# is the assessor pausing behind a screen, not cheating — say so in the KPI's
# own language ("keep it in that language as I am giving").
F = r'C:\KSK\NSDC_Skill_Intelligence_Dashboard.html'
s = open(F, encoding='utf-8').read()
def rep(a, b, n=1):
    global s
    assert s.count(a) == n, ('anchor', a[:70], s.count(a))
    s = s.replace(a, b)

# --- People-on-System: benchmark 1.2, assessor-at-screen explanation ---
rep('''      tone:maxPos>1.05?"crit":"good",
      cmp:{seen:maxPos,declared:1,seenTxt:fmt(maxPos,2),decTxt:"1.00 expected"},
      chip:maxPos>1.05?{t:"would flag",tone:"crit"}:{t:"within expectation",tone:"ok"},
      note:"More than one person at a single computer during a scored exam."}),''',
    '''      tone:maxPos>1.2?"crit":"good",
      cmp:{seen:maxPos,declared:1,seenTxt:fmt(maxPos,2),decTxt:"1.00 expected"},
      chip:maxPos>1.2?{t:"would flag — possible sharing",tone:"crit"}
           :maxPos>1.02?{t:"\≈1 \· assessor at a screen",tone:"ok"}
           :{t:"within expectation",tone:"ok"},
      note:"More than one person at a single computer during a scored exam. Benchmark is 1.2 — a reading around 1.1 is usually the assessor pausing behind a screen while walking the room, not sharing."}),''')

# --- Head-to-Machine card deleted outright: group practicals are assessed
#     together, and per the follow-up no Candidate-to-Assessor ratio either ---
rep('''    kcard({lbl:"Head-to-Machine Ratio",cam:{space:"nc-1",signal:"viva",cap:22},src:"modelled",
      val:maxH2M==null?"—":fmt(maxH2M,2),unit:"max",
      tone:maxH2M>1.2?"crit":maxH2M>1.05?"warn":"good",
      cmp:maxH2M==null?null:{seen:maxH2M,declared:1,seenTxt:fmt(maxH2M,2),decTxt:"1.00 expected"},
      chip:maxH2M>1.2?{t:"above 1:1",tone:"crit"}:{t:"at ratio",tone:"ok"},
      foot:wm.length?SP[wm.reduce((a,b)=>a.headToMachine>b.headToMachine?a:b).space].short:""}),
''', '')

# the integrity grid drops to its two remaining cards
rep('  AR.push(kg(3,[\n    kcard({lbl:"People-on-System Ratio"',
    '  AR.push(kg(2,[\n    kcard({lbl:"People-on-System Ratio"')
# the two consts that only fed the removed card
rep('''  const wm=A.filter(a=>a.headToMachine!=null);
  const maxPos=Math.max(...A.map(a=>a.peopleOnSystem));
  const maxH2M=wm.length?Math.max(...wm.map(a=>a.headToMachine)):null;''',
    '''  const maxPos=Math.max(...A.map(a=>a.peopleOnSystem));''')

open(F, 'w', encoding='utf-8').write(s)
print("q3 ok — bytes", len(s))
```

Note: the `\—`/`\≈`/`\·` escapes above are for the *Python source*; if the anchor for Head-to-Machine fails on the em-dash, print the live block (`sed -n '4350,4362p'`) and paste the exact text — the em-dash in `val:maxH2M==null?"—"` is a literal — character in the file.

- [ ] **Step 3: Verify**

Playwright evaluate:

```js
() => { setScope({lvl:"ctr",st:"BR",ctr:"BR-01"});
  S.tab="assessment"; S.slide=0; render();
  const t = document.getElementById("view").textContent;
  const bad = [];
  if (t.includes("Head-to-Machine")) bad.push("H2M still present");
  if (t.includes("Candidate-to-Assessor")) bad.push("C2A must not exist");
  if (!/assessor at a screen|within expectation|possible sharing/.test(t)) bad.push("PoS chip missing");
  if (/undefined|NaN/.test(t)) bad.push("NaN/undefined");
  return bad; }
```

Expected: `[]`

---

### Task 4: Round Durations (theory · viva · practical) + Infra Readiness card

**Files:**
- Modify: `NSDC_Skill_Intelligence_Dashboard.html` (ASSESS mock ~4308; assessment AR column ~4348; quality kg ~4246)

- [ ] **Step 1: Verify anchors**

```bash
cd /c/KSK
grep -c 'vivaAvg:6.5+g()\*6,' NSDC_Skill_Intelligence_Dashboard.html   # expect 1
grep -c 'AR.push(cgrid(1,\[' NSDC_Skill_Intelligence_Dashboard.html    # expect 1
grep -c 'lbl:"Classroom Interactivity Ratio"' NSDC_Skill_Intelligence_Dashboard.html # expect 1
```

- [ ] **Step 2: Write and run `q4.py`**

```python
# -*- coding: utf-8 -*-
# Reviewer: "viva duration, theory duration, practical duration तीनों डाल दो —
# तीनों round होते हैं". And equipment presence becomes an Infra Readiness
# check (is the committed asset visible in frame), never a utilisation % —
# beds are legitimately unused in some lessons, so a low ratio proves nothing.
F = r'C:\KSK\NSDC_Skill_Intelligence_Dashboard.html'
s = open(F, encoding='utf-8').read()
def rep(a, b, n=1):
    global s
    assert s.count(a) == n, ('anchor', a[:70], s.count(a))
    s = s.replace(a, b)

# --- mock fields for the two new rounds ---
rep("    vivaAvg:6.5+g()*6,",
    "    vivaAvg:6.5+g()*6, theoryDur:70+g()*40, practicalDur:22+g()*18,")

# --- Round Durations pair in the assessment right column, above its chart ---
rep('''  AR.push(cgrid(1,[
    cx("Assessor Presence by Trade",null,''',
    '''  AR.push(sec("Round Durations","three assessment rounds"));
  AR.push(kg(3,[
    kcard({lbl:"Theory Duration",src:"modelled",
      val:fmt(avg(A.map(a=>a.theoryDur)),0),unit:"min",
      foot:"pen-and-paper round"}),
    kcard({lbl:"Viva Duration",src:"modelled",
      val:fmt(avg(A.map(a=>a.vivaAvg)),1),unit:"min",
      foot:"per detected candidate-assessor pair"}),
    kcard({lbl:"Practical Duration",src:"modelled",
      val:fmt(avg(A.map(a=>a.practicalDur)),0),unit:"min",
      foot:"machine-based demonstration, individual or group"}),
  ]));
  AR.push(cgrid(1,[
    cx("Assessor Presence by Trade",null,''')

# --- Infra Readiness card on the Quality page ---
rep('''    kcard({lbl:"Classroom Interactivity Ratio",src:"modelled",''',
    '''    kcard({lbl:"Infra / Asset Readiness",src:"modelled",
      cam:{space:equipCam(),signal:"stations"},
      val:(()=>{const rooms=roomsIn().filter(r=>SP[r].stations>0);
        const committed=rooms.reduce((a,r)=>a+SP[r].stations,0);
        const present=committed-1;   /* one welding booth not sighted in the mock */
        return pc(committed?present/committed*100:0);})(),
      tone:"warn",
      chip:{t:"1 welding booth not sighted",tone:"warn"},
      foot:"committed assets visible in frame",
      note:"Is the committed asset physically present and visible \— a readiness check, not a utilisation rate. Some assets are legitimately unused in a given lesson (a GDA bed during dummy practice), so utilisation would mislead; presence cannot."}),
    kcard({lbl:"Classroom Interactivity Ratio",src:"modelled",''')

open(F, 'w', encoding='utf-8').write(s)
print("q4 ok — bytes", len(s))
```

- [ ] **Step 3: Verify**

Playwright evaluate:

```js
() => { const bad = [];
  setScope({lvl:"ctr",st:"BR",ctr:"BR-01"});
  S.tab="assessment"; S.slide=0; render();
  let t = document.getElementById("view").textContent;
  ["Theory Duration","Viva Duration","Practical Duration"].forEach(k => {
    if (!t.includes(k)) bad.push(k + " missing"); });
  S.tab="quality"; S.slide=0; render();
  t = document.getElementById("view").textContent;
  if (!t.includes("Infra / Asset Readiness")) bad.push("infra card missing");
  if (!t.includes("Equipment Utilization Rate")) bad.push("sheet KPI lost");  // sheet KPI stays
  if (/NaN|undefined/.test(t)) bad.push("NaN/undefined");
  return bad; }
```

Expected: `[]`

---

### Task 5: Commit A — corrections batch

**Files:**
- Modify: `CLAUDE.md`, `SESSION_LOG.md`

- [ ] **Step 1: Append the sheet-override note to CLAUDE.md**

Append this block to the end of the `## Hard-won lessons` section (after the "Duplicate sections" bullet):

```markdown
- **Business-review overrides (2026-08-19) — do NOT "fix" these back to the KPI sheet.** The review meeting is newer than the sheet and explicitly changed: "Invigilator" → **Assessor** everywhere (trainers aren't allowed in the exam room); assessment denominator is **eligible** candidates (from the assessment data sheet), not enrolled; **Head-to-Machine Ratio removed outright** for practicals (group practicals are assessed together — 1:1 is meaningless; no replacement ratio); AEBAS is a **face-authentication** device, not fingerprint; asset checks are **Infra Readiness** (asset visible yes/no), never utilisation %, for bed-type assets. People-on-System benchmark is 1.2 — ~1.1 is the assessor behind a screen.
```

- [ ] **Step 2: Append the session entry to SESSION_LOG.md**

```markdown
## 2026-08-19 (Anshul's session) — Business-review corrections, part 1
**Code changed:** NSDC_Skill_Intelligence_Dashboard.html — Invigilator→Assessor, fingerprint→face-auth, eligible denominator on assessments, Head-to-Machine removed outright, People-on-System benchmark language (1.2 / assessor-at-screen), Round Durations trio (theory/viva/practical), Infra Readiness card, centre-wide batch-cap note
**Chat summary:** Business team reviewed the demo with the NSDC-side stakeholder. Terminology corrections were direct quotes ("this is a face auth device", "it's called assessor, not invigilator"). Bigger conceptual fixes: assessment counts only eligible candidates (always ≤ enrolled, SIDH later); per-machine ratios are wrong for group practicals so that KPI is gone with no replacement; utilisation is the wrong lens for bed-type assets (presence check instead). All still mock data — no live centre yet.
**Context updated:** CLAUDE.md hard-won lessons now list the five sheet-overrides so nobody reverts them to match the spreadsheet.
```

- [ ] **Step 3: Commit and push**

```bash
cd /c/KSK
git add NSDC_Skill_Intelligence_Dashboard.html CLAUDE.md SESSION_LOG.md
git commit -m "Apply business-review corrections: assessor, face-auth, eligible, batch sizes, infra readiness"
git push
```

Expected: hook passes (all three files staged), push succeeds.

---

### Task 6: Time-bucketed occupancy + peak-deviation chip (selected room)

**Files:**
- Modify: `NSDC_Skill_Intelligence_Dashboard.html` (selected-room block in `pageAttendance`, ~4180–4200)

- [ ] **Step 1: Verify anchors**

```bash
cd /c/KSK
grep -c '"selected space"' NSDC_Skill_Intelligence_Dashboard.html          # expect 1
grep -c 'cx("Headcount Across the Instructional Day"' NSDC_Skill_Intelligence_Dashboard.html  # expect 1
grep -c 'lbl:"Peak, Selected Day"' NSDC_Skill_Intelligence_Dashboard.html  # expect 1
```

- [ ] **Step 2: Write and run `q5.py`**

```python
# -*- coding: utf-8 -*-
# Reviewer ("this is a very good insight"): for a batch, show how many
# candidates were in the room in each hour band against the day's peak —
# a morning spike that empties out after lunch must be visible at a glance.
F = r'C:\KSK\NSDC_Skill_Intelligence_Dashboard.html'
s = open(F, encoding='utf-8').read()
def rep(a, b, n=1):
    global s
    assert s.count(a) == n, ('anchor', a[:70], s.count(a))
    s = s.replace(a, b)

# --- hour-band aggregation straight from the real per-2-minute series ---
rep('''    v.appendChild(sec(SP[cur].name+(cc?" \· "+cc.name:""),"selected space"));''',
    '''    const hourBand=(camId,date)=>{
      const ser=SER10[camId]&&SER10[camId][date]; if(!ser) return [];
      const out=[];
      for(let h=7;h<19;h++){
        let sum=0,n=0,pk=0;
        for(let i=sl(h*60);i<sl((h+1)*60);i++){
          const x=ser[i]; if(x<0) continue;
          sum+=x/10; n++; if(x/10>pk) pk=x/10;
        }
        out.push({h:String(h).padStart(2,"0")+"\–"+String(h+1).padStart(2,"0"),
                  avg:n?sum/n:null, pk});
      }
      return out;
    };
    v.appendChild(sec(SP[cur].name+(cc?" \· "+cc.name:""),"selected space"));''')

# --- the headcount line gets the hour-band bars beside it ---
rep('''    v.appendChild(cgrid(1,[
      cx("Headcount Across the Instructional Day",null,''',
    '''    v.appendChild(cgrid(2,[
      cx("Headcount Across the Instructional Day",null,''')
rep('''        "Six-minute means straight from the export. Gaps are hatched, never drawn as zero."),
    ]));''',
    '''        "Six-minute means straight from the export. Gaps are hatched, never drawn as zero."),
      cx("Candidates by Hour Band",null,
        W=>{const hb=hourBand(cur,d).filter(b=>b.avg!=null);
          if(!hb.length) return emptyViz("No feed on this day");
          const pkAll=Math.max(...hb.map(b=>b.pk),1);
          return bars({W,cats:hb.map(b=>b.h),v:hb.map(b=>b.avg),
            fmtV:x=>fmt(x,1),max:niceMax(pkAll),vlabel:"Mean in band",minH:208,
            statusOf:x=>refCol(x,pkAll*0.6),
            threshold:pkAll,thresholdLabel:"day peak"});},
        ()=>mkTable(["Hour","Mean","Peak in band"],
          hourBand(cur,d).map(b=>[b.h,b.avg==null?"\—":fmt(b.avg,1),fmt(b.pk,1)])),
        "Candidates in the room per hour band against the day's peak \— a morning spike that empties out after lunch shows up here immediately."),
    ]));''')

# --- deviation chip: peak far above the mean is itself a finding ---
rep('''      kcard({lbl:"Peak, Selected Day",src:"measured",val:fmt(rr.peak,1),
        foot:rr.peakMin!=null?"at "+hm(rr.peakMin):""}),''',
    '''      kcard({lbl:"Peak, Selected Day",src:"measured",val:fmt(rr.peak,1),
        chip:(rr.avgIn&&rr.peak/rr.avgIn>1.8)
          ?{t:"peak "+fmt(rr.peak/rr.avgIn,1)+"\× the mean",tone:"warn"}:null,
        foot:rr.peakMin!=null?"at "+hm(rr.peakMin):"",
        note:"A peak far above the mean usually means a morning spike that emptied out \— the hour bands beside this show exactly where."}),''')

open(F, 'w', encoding='utf-8').write(s)
print("q5 ok — bytes", len(s))
```

Anchor caution: the `\·` in the first anchor is the literal `·` character in the file (`" · "+cc.name`). If the assert fails, print the live line (`grep -n '"selected space"'` then `sed -n`) and match it exactly.

- [ ] **Step 3: Verify**

Playwright evaluate:

```js
async () => { setScope({lvl:"ctr",st:"BR",ctr:"BR-01"});
  S.tab="attendance"; S.slide=0; render();
  await new Promise(r=>setTimeout(r,400));
  const slides=[...document.querySelectorAll(".slide")];
  const i=slides.findIndex(sl=>sl.textContent.includes("Candidates by Hour Band"));
  if(i<0) return ["hour-band chart missing"];
  DECK_GO(i); await new Promise(r=>setTimeout(r,400));
  const cur=slides[i];
  const bad=[];
  if(cur.scrollHeight-cur.clientHeight>10) bad.push("slide clipped +"+(cur.scrollHeight-cur.clientHeight));
  if(/NaN|undefined|Infinity/.test(cur.textContent)) bad.push("NaN");
  return bad; }
```

Expected: `[]`

---

### Task 7: Instructor day-on-day pattern (actionable chip)

**Files:**
- Modify: `NSDC_Skill_Intelligence_Dashboard.html` (centre-summary Active Time % card, ~4061)

- [ ] **Step 1: Verify the anchor — print the live card first**

```bash
cd /c/KSK
sed -n '4058,4072p' NSDC_Skill_Intelligence_Dashboard.html
```

The card body (from the current file) is:

```js
    kcard({lbl:"Active Time %",src:"measured",cam:{space:busiestRoom(),signal:"instructor"},
      val:pc(pAct),tone:rygTone(pAct,75),
      tri:[{k:"Active",v:zAct,c:ST.good},{k:"Unsupervised",v:zUns,c:ST.warning},
           {k:"Idle",v:zIdl,c:CSSV("--rule")}],
      chip:{t:hrs(zAct)+"h of "+hrs(zDay)+"h",tone:rygTone(pAct,75)},
      note:"Share of the day a room had a cohort AND an instructor present together."}),
```

- [ ] **Step 2: Write and run `q6.py`**

```python
# -*- coding: utf-8 -*-
# Reviewer: one low day is a sick day, not a finding — "capture day on day and
# see the pattern... KPIs should be actionable". The chip now distinguishes a
# repeated pattern (act) from a single-day dip (watch).
F = r'C:\KSK\NSDC_Skill_Intelligence_Dashboard.html'
s = open(F, encoding='utf-8').read()
def rep(a, b, n=1):
    global s
    assert s.count(a) == n, ('anchor', a[:70], s.count(a))
    s = s.replace(a, b)

# per-day active share across the whole feed window, computed once above the card
rep('''  v.appendChild(sec("Active Hours & Instructor Presence","camera \· share of the observed day"));''',
    '''  const actByDay=FEED_DATES.map(dd=>{
    const rs=roomsIn().map(r=>D[dd][r]).filter(Boolean);
    const tot=Math.max(1,rs.length*(DAY_TO-DAY_FROM));
    return rs.reduce((a,r)=>a+(r.activeMin||0),0)/tot*100;
  });
  const lowDays=actByDay.filter(x=>x<40).length;
  v.appendChild(sec("Active Hours & Instructor Presence","camera \· share of the observed day"));''')

rep('''      chip:{t:hrs(zAct)+"h of "+hrs(zDay)+"h",tone:rygTone(pAct,75)},
      note:"Share of the day a room had a cohort AND an instructor present together."}),''',
    '''      spark:{vals:actByDay,cur:FEED_DATES.indexOf(d),ref:75},
      chip:lowDays>=3?{t:"low on "+lowDays+" of "+FEED_DATES.length+" days \— pattern, act",tone:"crit"}
          :(pAct<40?{t:"single-day dip \— watch, don't act",tone:"warn"}
                   :{t:hrs(zAct)+"h of "+hrs(zDay)+"h",tone:rygTone(pAct,75)}),
      note:"Share of the day a room had a cohort AND an instructor present together. One low day can be a sick day \— not actionable. A repeated pattern across days is a red flag, and the sparkline shows which it is."}),''')

open(F, 'w', encoding='utf-8').write(s)
print("q6 ok — bytes", len(s))
```

Note: `tri` and `spark` are mutually exclusive in `kcard()` (`if/else if` chain renders only one) — the spark **replaces** the tri here intentionally; the tri composition survives on the per-room cards two slides later. Also: the `\·` in the sec() anchor is the literal `·` in the file.

- [ ] **Step 3: Verify**

Playwright evaluate:

```js
() => { setScope({lvl:"ctr",st:"BR",ctr:"BR-01"});
  S.tab="attendance"; S.slide=0; render();
  const t = document.getElementById("view").textContent;
  const bad = [];
  if (!/pattern, act|watch, don't act|h of /.test(t)) bad.push("actionability chip missing");
  if (/NaN|undefined/.test(t)) bad.push("NaN");
  return bad; }
```

Expected: `[]`

---

### Task 8: RYG heatmaps

**Files:**
- Modify: `NSDC_Skill_Intelligence_Dashboard.html` (`heat()` ~2090–2115; the two heat call-sites ~4159 and in `pageQuality`)

- [ ] **Step 1: Verify anchors**

```bash
cd /c/KSK
grep -c 'const {rows,cols,val,fmtV=v=>fmt(v),rowLab,colLab,unit=""}=o;' NSDC_Skill_Intelligence_Dashboard.html  # expect 1
grep -c 'background:${SEQ\[idx\]}' NSDC_Skill_Intelligence_Dashboard.html   # expect 1
grep -c 'vlabel:"Mean attendance",' NSDC_Skill_Intelligence_Dashboard.html  # expect 1
grep -c 'unit:"%",vlabel:"Active share",' NSDC_Skill_Intelligence_Dashboard.html # expect 1
```

- [ ] **Step 2: Write and run `q7.py`**

```python
# -*- coding: utf-8 -*-
# Reviewer request ("इसमें RYG ला देते हैं"): heatmaps score against a
# reference instead of a magnitude-only blue ramp. heat() gains an optional
# colorOf(v, rowIndex); when present, cells take the strict red-amber-green
# reference ramp (refCol) and white ink.
F = r'C:\KSK\NSDC_Skill_Intelligence_Dashboard.html'
s = open(F, encoding='utf-8').read()
def rep(a, b, n=1):
    global s
    assert s.count(a) == n, ('anchor', a[:70], s.count(a))
    s = s.replace(a, b)

rep('  const {rows,cols,val,fmtV=v=>fmt(v),rowLab,colLab,unit=""}=o;',
    '  const {rows,cols,val,fmtV=v=>fmt(v),rowLab,colLab,unit="",colorOf}=o;')
rep('''      const t=(v-mn)/((mx-mn)||1);
      const idx=clamp(Math.floor(t*SEQ.length),0,SEQ.length-1);
      const n=el("div",{class:"hm-c",style:`background:${SEQ[idx]};color:${idx>=4?"#fff":CSSV("--ink")}`,tabindex:"0",text:fmtV(v)});''',
    '''      const t=(v-mn)/((mx-mn)||1);
      const idx=clamp(Math.floor(t*SEQ.length),0,SEQ.length-1);
      const bg=colorOf?colorOf(v,ri):SEQ[idx];
      const ink=colorOf?"#fff":(idx>=4?"#fff":CSSV("--ink"));
      const n=el("div",{class:"hm-c",style:`background:${bg};color:${ink}`,tabindex:"0",text:fmtV(v)});''')

# Peak Occupancy by Space and Day: score each room against 75% of its own roll
rep('''        fmtV:x=>fmt(x,1),vlabel:"Mean attendance",''',
    '''        fmtV:x=>fmt(x,1),vlabel:"Mean attendance",
        colorOf:(v,ri)=>refCol(v,(courseOf(roomsIn()[ri])||{enrolled:20}).enrolled*0.75),''')

# Workstation Utilisation by Hour: score against the 70% reference
rep('''unit:"%",vlabel:"Active share",''',
    '''unit:"%",vlabel:"Active share",colorOf:v=>refCol(v,70),''')

open(F, 'w', encoding='utf-8').write(s)
print("q7 ok — bytes", len(s))
```

- [ ] **Step 3: Verify — cells carry RYG hexes, not the blue ramp**

Playwright evaluate:

```js
async () => { setScope({lvl:"ctr",st:"BR",ctr:"BR-01"});
  S.tab="attendance"; S.slide=0; render();
  await new Promise(r=>setTimeout(r,400));
  const i=[...document.querySelectorAll(".slide")].findIndex(sl=>sl.textContent.includes("Peak Occupancy by Space and Day"));
  DECK_GO(i); await new Promise(r=>setTimeout(r,300));
  const cells=[...document.querySelectorAll(".hm-c")].map(c=>c.style.background);
  const RYG_HEX=["rgb(192, 57, 43)","rgb(221, 106, 53)","rgb(201, 151, 31)",
                 "rgb(106, 169, 106)","rgb(63, 148, 85)","rgb(15, 122, 69)"];
  const ryg=cells.filter(b=>RYG_HEX.some(h=>b.includes(h))).length;
  return {cells:cells.length, rygCells:ryg, ok: ryg>0}; }
```

Expected: `ok: true` with `rygCells > 0`.

---

### Task 9: Coverage — camera-off vs server-off + Commit B

**Files:**
- Modify: `NSDC_Skill_Intelligence_Dashboard.html` (helper before `pageCoverage`; Feed Integrity kg)
- Modify: `SESSION_LOG.md`

- [ ] **Step 1: Verify anchors and find pageCoverage's date variable**

```bash
cd /c/KSK
grep -n 'function pageCoverage' NSDC_Skill_Intelligence_Dashboard.html
sed -n "$(grep -n 'function pageCoverage' NSDC_Skill_Intelligence_Dashboard.html | cut -d: -f1),+6p" NSDC_Skill_Intelligence_Dashboard.html
grep -c 'lbl:"Overnight Baseline Reading"' NSDC_Skill_Intelligence_Dashboard.html  # expect 1
```

Confirm the page opens with `const d=S.date` (adjust the patch if the variable name differs).

- [ ] **Step 2: Write and run `q8.py`**

```python
# -*- coding: utf-8 -*-
# Reviewer: distinguish "one camera switched off while the rest streamed"
# (someone turned the camera off) from "every camera dark at once" (server or
# power down), and let FREQUENCY across days say intentional vs technical.
# Prior-vendor story: centres gave wrong camera details on purpose — this is
# a non-compliance signal, not telemetry trivia.
F = r'C:\KSK\NSDC_Skill_Intelligence_Dashboard.html'
s = open(F, encoding='utf-8').read()
def rep(a, b, n=1):
    global s
    assert s.count(a) == n, ('anchor', a[:70], s.count(a))
    s = s.replace(a, b)

rep('''/* ================================================================
   PAGE 5 \— COVERAGE  (feed integrity, all measured)
   ================================================================ */
function pageCoverage(){''',
    '''/* ================================================================
   PAGE 5 \— COVERAGE  (feed integrity, all measured)
   ================================================================ */
/* Splits missing feed into per-camera outages (server up, camera off) and
   whole-centre outages (server down) inside the covered window of a day. */
function offStats(date){
  const win=PARTIAL[date]; if(!win) return null;
  const s0=sl(win.from), s1=sl(win.to);
  let srv=0, cam=0;
  for(let i=s0;i<=s1;i++){
    let down=0;
    for(const c of CAMS) if(SER10[c][date][i]<0) down++;
    if(down===CAMS.length) srv++; else cam+=down;
  }
  let worst=null, worstN=0;
  CAMS.forEach(c=>{let n=0;
    for(let i=s0;i<=s1;i++) if(SER10[c][date][i]<0) n++;
    if(n>worstN){worstN=n;worst=c;}});
  return {srvMin:srv*BUCKET, camMin:cam*BUCKET, worst, worstMin:worstN*BUCKET};
}
function pageCoverage(){''')

rep('''    kcard({lbl:"Overnight Baseline Reading"''',
    '''    kcard({lbl:"Camera-Off Minutes",src:"measured",
      cam:{space:busiestRoom(),signal:"feed"},
      val:(()=>{const o=offStats(d);return fmt(o?o.camMin:0);})(),unit:"min",
      tone:(()=>{const o=offStats(d);return o&&o.camMin>120?"crit":o&&o.camMin>30?"warn":"good";})(),
      chip:(()=>{const o=offStats(d);return o&&o.worst&&o.worstMin>0
        ?{t:"worst: "+SP[o.worst].short+" \· "+fmt(o.worstMin)+" min",tone:"warn"}:null;})(),
      foot:"single cameras dark while others streamed",
      note:"Feed missing from one camera while the rest of the centre kept streaming \— the server was up, so the camera itself was off. Frequency across days decides intentional vs technical."}),
    kcard({lbl:"Server-Off Minutes",src:"measured",
      val:(()=>{const o=offStats(d);return fmt(o?o.srvMin:0);})(),unit:"min",
      tone:(()=>{const o=offStats(d);return o&&o.srvMin>60?"crit":o&&o.srvMin>10?"warn":"good";})(),
      chip:(()=>{const n=FEED_DATES.filter(x=>{const o=offStats(x);return o&&o.srvMin>10;}).length;
        return {t:n+" of "+FEED_DATES.length+" days \— "+(n>=3?"pattern":"isolated"),
                tone:n>=3?"crit":"ok"};})(),
      foot:"every camera dark at once",
      note:"All cameras missing simultaneously \— the recording server (or power) was down, not one camera. A repeated pattern here is a non-compliance finding to report."}),
    kcard({lbl:"Overnight Baseline Reading"''')

open(F, 'w', encoding='utf-8').write(s)
print("q8 ok — bytes", len(s))
```

Anchor caution: the `\—` in the pageCoverage banner anchor is the literal `—` in the file's comment. If `d` is not the date variable in `pageCoverage`, replace `offStats(d)` accordingly.

- [ ] **Step 3: Verify**

Playwright evaluate:

```js
() => { setScope({lvl:"ctr",st:"BR",ctr:"BR-01"});
  S.tab="coverage"; S.slide=0; render();
  const t = document.getElementById("view").textContent;
  const bad = [];
  ["Camera-Off Minutes","Server-Off Minutes"].forEach(k=>{
    if(!t.includes(k)) bad.push(k+" missing"); });
  if(!/pattern|isolated/.test(t)) bad.push("frequency chip missing");
  if(/NaN|undefined/.test(t)) bad.push("NaN");
  return bad; }
```

Expected: `[]`

- [ ] **Step 4: Append to SESSION_LOG.md and commit (Commit B)**

Append:

```markdown
## 2026-08-19 (Anshul's session) — Business-review insights, part 2
**Code changed:** NSDC_Skill_Intelligence_Dashboard.html — Candidates by Hour Band chart beside the selected-room headcount line (with day-peak threshold), peak-vs-mean deviation chip, instructor Active Time % now shows a day-on-day sparkline with an actionability chip (pattern = act, single dip = watch), heat() gained colorOf and both heatmaps now score RYG against references, Coverage split into Camera-Off vs Server-Off minutes with a frequency chip
**Chat summary:** These are the reviewer's own asks: hour-band occupancy ("a very good insight"), day-on-day instructor pattern ("KPIs should be actionable"), RYG on the heatmap, and the camera-off/server-off distinction where frequency decides intentional vs technical (prior vendor: centres switched cameras off or gave wrong details on purpose).
**Context updated:** none this commit (overrides were documented in the previous entry).
```

```bash
cd /c/KSK
git add NSDC_Skill_Intelligence_Dashboard.html CLAUDE.md SESSION_LOG.md
git commit -m "Add hour-band occupancy, instructor pattern chip, RYG heatmaps, camera/server-off split"
git push
```

---

### Task 10: Overview ongoing-trainings · historical T-1 labels · All-days view

**Files:**
- Modify: `NSDC_Skill_Intelligence_Dashboard.html` (map rail rows ~3608; mp-badge ~3660; kc-cam button; day selector + a new 4b aggregate block before section 5b)

- [ ] **Step 1: Verify anchor**

```bash
cd /c/KSK
grep -c '\["Learners enrolled",inr(a.enrolled),"ok"\]' NSDC_Skill_Intelligence_Dashboard.html  # expect 1
```

- [ ] **Step 2: Write and run `q9.py`**

```python
# -*- coding: utf-8 -*-
# Overview input from the meeting: "कितना ongoing training चल रहा है" — an
# ongoing-trainings count in the map's evidence-base rail. Mock definition:
# a centre counts as ongoing when its Active Time % clears 40.
F = r'C:\KSK\NSDC_Skill_Intelligence_Dashboard.html'
s = open(F, encoding='utf-8').read()
def rep(a, b, n=1):
    global s
    assert s.count(a) == n, ('anchor', a[:70], s.count(a))
    s = s.replace(a, b)

rep('      ["Learners enrolled",inr(a.enrolled),"ok"]',
    '''      ["Learners enrolled",inr(a.enrolled),"ok"],
      ["Ongoing trainings",fmt(scopeList().filter(c=>c.m.attend>=40).length)+" of "+fmt(a.n),"ok"]''')

# --- historical T-1 framing: this mock never claims to be live ---
rep('el("div",{class:"mp-badge"},[el("i"),document.createTextNode("Live intel")])',
    'el("div",{class:"mp-badge"},[el("i"),document.createTextNode("Historical · to T−1")])')
rep('[el("i"),el("span",{text:"Live view"})]',
    '[el("i"),el("span",{text:"Camera replay"})]')

open(F, 'w', encoding='utf-8').write(s)
print("q9 ok — bytes", len(s))
```

- [ ] **Step 3: Write and run `q10.py` — the "All days" aggregate view**

```python
# -*- coding: utf-8 -*-
# Multi-date: an "All days" option aggregates the whole window as a synthetic
# date key "ALL". Per-camera series are averaged slot-wise across days (so
# every curve, hour-band, scrubber and offStats works unchanged); per-room
# derivations are averaged (peak = average of the daily peaks, exactly the
# aggregation the reviewer described); AEBAS is averaged; coverage metadata is
# unioned. FEED_DATES itself is untouched, so day-on-day sparklines and the
# heatmap's four day-columns keep showing the real days.
F = r'C:\KSK\NSDC_Skill_Intelligence_Dashboard.html'
s = open(F, encoding='utf-8').read()
def rep(a, b, n=1):
    global s
    assert s.count(a) == n, ('anchor', a[:70], s.count(a))
    s = s.replace(a, b)

rep('''/* ================================================================
   5b · state — declared before anything reads it
   ================================================================ */''',
'''/* ================================================================
   4b · "All days" aggregate — synthetic date key "ALL"
   Peak here is the AVERAGE of the daily peaks (the aggregation the
   review meeting asked for), never a re-computed instantaneous max.
   ================================================================ */
const ALLD="ALL";
(function(){
  CAMS.concat(COMMON).forEach(c=>{ if(!SER10[c]) return;
    const out=new Int16Array(SLOTS);
    for(let i=0;i<SLOTS;i++){
      let sum=0,n=0;
      FEED_DATES.forEach(d=>{const v=SER10[c][d]&&SER10[c][d][i];
        if(v!=null&&v>=0){sum+=v;n++;}});
      out[i]=n?Math.round(sum/n):-1;
    }
    SER10[c][ALLD]=out;
  });
  PARTIAL[ALLD]={from:Math.min(...FEED_DATES.map(d=>PARTIAL[d].from)),
                 to:Math.max(...FEED_DATES.map(d=>PARTIAL[d].to)),full:false};
  Object.keys(COV).forEach(c=>{ if(!COV[c]) return;
    const f=Math.min(...FEED_DATES.map(d=>COV[c][d]?COV[c][d][0]:1e9));
    const t=Math.max(...FEED_DATES.map(d=>COV[c][d]?COV[c][d][1]:-1));
    if(t>=0) COV[c][ALLD]=[f,t]; });
  Object.keys(FEEDMETA).forEach(c=>{
    FEEDMETA[c][ALLD]={
      slots:Math.round(avg(FEED_DATES.map(d=>FEEDMETA[c][d]?FEEDMETA[c][d].slots:0))),
      gaps:FEED_DATES.reduce((a,d)=>a+(FEEDMETA[c][d]?FEEDMETA[c][d].gaps:0),0)};
  });
  D[ALLD]={};
  /* the instructor-model strip has no meaningful average — the All-days
     presence strip shows the latest day and says so via DAYLBL */
  MI[ALLD]=MI[FEED_DATES[FEED_DATES.length-1]];
  ROOMS.concat(COMMON).forEach(r=>{
    const recs=FEED_DATES.map(d=>D[d][r]).filter(Boolean);
    if(!recs.length) return;
    const o={cam:r};
    ["peak","avgIn","presentMin","committedMin","committedCoveredMin",
     "activeMin","unsupMin","idleMin","instrMin","nightAvg","noDataMin",
     "gapsMin","durNoLunch"].forEach(k=>{
      const vs=recs.map(x=>x[k]).filter(v=>v!=null);
      o[k]=vs.length?avg(vs):null; });
    o.running=recs.some(x=>x.running);
    const fm=recs.map(x=>x.firstMin).filter(v=>v!=null);
    o.firstMin=fm.length?Math.min(...fm):null;
    o.peakMin=null;            // an averaged peak has no single minute
    D[ALLD][r]=o;
  });
  AEBAS[ALLD]={};
  COURSES.forEach(c=>{AEBAS[ALLD][c.code]=
    Math.round(avg(FEED_DATES.map(d=>AEBAS[d][c.code])));});
  DAYLBL[ALLD]="All days · 05–08 Aug";
})();

/* ================================================================
   5b · state — declared before anything reads it
   ================================================================ */''')

rep('''FEED_DATES.forEach(d=>dSel.appendChild(el("option",{value:d,
  text:DAYLBL[d]+(PARTIAL[d]&&!PARTIAL[d].full?" · partial":"")})));''',
'''FEED_DATES.forEach(d=>dSel.appendChild(el("option",{value:d,
  text:DAYLBL[d]+(PARTIAL[d]&&!PARTIAL[d].full?" · partial":"")})));
dSel.appendChild(el("option",{value:ALLD,text:DAYLBL[ALLD]}));''')

open(F, 'w', encoding='utf-8').write(s)
print("q10 ok — bytes", len(s))
```

Pre-flight for this script: `grep -c 'AEBAS\[' NSDC_Skill_Intelligence_Dashboard.html` to confirm the AEBAS mock is keyed `AEBAS[date][courseCode]`, and `grep -n 'const COMMON='` to confirm COMMON exists. `dayIdx()` returns -1 for ALL — `sparkline` skips its current-day dot (`vv[-1]` is undefined and guarded), `prevDay()` returns null so delta chips vanish: acceptable, verify no NaN below.

- [ ] **Step 4: Verify**

```js
async () => { const bad=[];
  setScope({lvl:"in",st:null,ctr:null}); S.tab="overview"; S.slide=0; render();
  let t = document.getElementById("view").textContent;
  if (!t.includes("Ongoing trainings")) bad.push("ongoing row missing");
  const badge=document.querySelector(".mp-badge");
  if (badge && /live/i.test(badge.textContent)) bad.push("badge still says live");
  setScope({lvl:"ctr",st:"BR",ctr:"BR-01"});
  const dSel=document.getElementById("fDay");
  if(![...dSel.options].some(o=>o.value==="ALL")) bad.push("ALL option missing");
  S.date="ALL"; dSel.value="ALL";
  for(const tab of ["overview","attendance","quality","assessment","coverage"]){
    S.tab=tab;S.slide=0;render(); await new Promise(r=>setTimeout(r,350));
    const x=document.getElementById("view").textContent;
    if(/NaN|undefined|Infinity/.test(x)) bad.push("ALL/"+tab); }
  S.date=FEED_DATES[FEED_DATES.length-1]; dSel.value=S.date; render();
  if (document.querySelector(".kc-cam") &&
      /Live view/.test(document.getElementById("view").textContent))
    bad.push("Live view label still present");
  return bad; }
```

Expected: `[]`

---

### Task 11: Full verification sweep + trades-count consistency + Commit C

**Files:**
- Modify: `SESSION_LOG.md`
- Commit: everything

- [ ] **Step 1: Trades-count consistency (the "5 vs 6" catch from the meeting)**

Playwright evaluate:

```js
() => { setScope({lvl:"ctr",st:"BR",ctr:"BR-01"});
  const bad = [];
  if (COURSES.length !== 6) bad.push("COURSES != 6");
  ["overview","attendance","quality","assessment","coverage"].forEach(t => {
    S.tab=t; S.slide=0; render();
    const x = document.getElementById("view").textContent;
    // any "of N declared/courses" figure must say 6, never 5
    const m = x.match(/of (\d) (declared|courses)/g) || [];
    m.forEach(hit => { if (!hit.includes("6")) bad.push(t+": "+hit); });
  });
  return bad; }
```

Expected: `[]`. If a `5` shows up anywhere, find its source before proceeding — this is the exact inconsistency the reviewer caught live.

- [ ] **Step 2: Full sweep — every tab × every scope, settled measurement**

Playwright evaluate (measurement must run **after** the 440 ms entrance animation settles — measuring mid-animation produced false +7 px clips once before):

```js
async () => {
  const TABS=["overview","attendance","quality","assessment","coverage"];
  const settle=()=>new Promise(r=>setTimeout(r,450));
  const out={checks:0,clip:[],bad:[],camOk:0,camBad:0};
  for(const sc of [{lvl:"in",st:null,ctr:null},{lvl:"st",st:"UP",ctr:null},
                   {lvl:"ctr",st:"BR",ctr:"BR-01"}]){
    // run the centre pass twice: once on the latest day, once on "ALL"
    const dates = sc.lvl==="ctr" ? [FEED_DATES[FEED_DATES.length-1],"ALL"] : [S.date];
    for(const dd of dates){ S.date=dd;
    setScope(sc);
    for(const t of TABS){ S.tab=t;S.slide=0;render(); await settle(); out.checks++;
      if(/NaN|undefined|Infinity/.test(document.getElementById("view").textContent))
        out.bad.push(sc.lvl+"/"+t);
      const deck=document.querySelector(".deck");
      if(deck) for(let i=0;i<S.slideN;i++){ DECK_GO(i); await new Promise(r=>setTimeout(r,60));
        const cur=deck.querySelectorAll(".slide")[i];
        if(cur.scrollHeight-cur.clientHeight>10) out.clip.push(sc.lvl+"/"+t+" s"+(i+1)); }
      if(deck) DECK_GO(0); } } }
  S.date=FEED_DATES[FEED_DATES.length-1];
  // every camera button still opens its drawer
  setScope({lvl:"ctr",st:"BR",ctr:"BR-01"});
  for(const t of TABS){ S.tab=t;S.slide=0;render(); await settle();
    for(let i=0;i<S.slideN;i++){ DECK_GO(i);
      for(const b of [...document.querySelectorAll(".slide")[i].querySelectorAll(".kc-cam")]){
        b.click(); await new Promise(r=>setTimeout(r,40));
        document.querySelector(".camdrawer")?out.camOk++:out.camBad++; b.click(); } } }
  out.pageScroll=document.documentElement.scrollHeight-innerHeight;
  return out; }
```

Expected: `bad: []`, `clip: []`, `camBad: 0`, `pageScroll ≤ 0`. Also confirm zero console errors (`browser_console_messages`, level error).

- [ ] **Step 3: KPI register audit — nothing lost, overrides applied**

Playwright evaluate:

```js
() => { const seen=new Set();
  setScope({lvl:"ctr",st:"BR",ctr:"BR-01"});
  ["overview","attendance","quality","assessment","coverage"].forEach(t=>{
    S.tab=t;S.slide=0;render();
    document.querySelectorAll(".kc-h u").forEach(n=>seen.add(n.textContent)); });
  const mustHave=["Avg Attendance","Peak Occupancy","Active Classrooms","Active Class-Hours",
    "Accreditation Validity","Sector Authorization Count","Declared Space Count",
    "Camera Coverage Ratio","Total Enrolled Headcount","Total Sanctioned Capacity",
    "Center-Wide Vacant Seats","Center-Wide Enrollment Utilization Rate",
    "Cohort Detection Count","Biometric Attendance Count","Camera Headcount",
    "Avg Cohort Duration","Avg Cohort Headcount","Peak Headcount Trend",
    "Active Time %","Unsupervised Time %","Idle Time %","Instructor Presence Timeline",
    "Committed Training Hours","Committed Headcount","Committed Trainer Count",
    "Instructor Interactivity Ratio","Equipment Utilization Rate",
    "Zone Utilization Rate","Workstation Utilization Rate","Classroom Interactivity Ratio",
    "Assessment Attendance Rate","Vivas Conducted","Viva Completion Rate",
    "Average Viva Duration","People-on-System Ratio","Assessor Presence Rate",
    "Infra / Asset Readiness",
    "Theory Duration","Viva Duration","Practical Duration",
    "Camera-Off Minutes","Server-Off Minutes"];
  const mustNot=["Invigilator Presence Rate","Head-to-Machine Ratio",
    "Candidate-to-Assessor Ratio",
    "Trainer Coverage Ratio","Teacher Punctuality","Cohort Punctuality",
    "Late Start","Early Finish"];
  return {missing:mustHave.filter(k=>!seen.has(k)),
          shouldBeGone:mustNot.filter(k=>seen.has(k))}; }
```

Expected: `missing: []`, `shouldBeGone: []`.

- [ ] **Step 4: Append to SESSION_LOG.md and commit (Commit C)**

Append:

```markdown
## 2026-08-19 (Anshul's session) — Business-review fixes, final sweep
**Code changed:** NSDC_Skill_Intelligence_Dashboard.html — Ongoing trainings row in the map's evidence base; full verification pass
**Chat summary:** Closed out the business-review batch: full Playwright sweep (every tab × India/state/centre, settled measurement) — zero NaN, zero clipped slides, zero page scroll, all camera drawers opening; KPI register audit confirms all sheet KPIs plus the meeting's new ones present and the removed ones (Invigilator, Head-to-Machine, the five red-cut sheet KPIs) absent; trades-count consistency check confirms 6 everywhere (the "5 vs 6" the reviewer caught live).
**Context updated:** none this commit.
```

```bash
cd /c/KSK
git add NSDC_Skill_Intelligence_Dashboard.html SESSION_LOG.md CLAUDE.md
git commit -m "Overview ongoing-trainings row; business-review batch verified end to end"
git push
```

- [ ] **Step 5: Deliver the file to the user** (SendUserFile with a one-line caption listing the headline changes).

---

## Explicitly out of scope (parked, per the meeting itself)

- **Live-centre dashboard / live feeds / CCTV integration** — the live dashboard is the old, separately-built product and stays separate; this mockup exists to show capability once SIDH + camera data arrive. Nothing here connects to a real feed.
- **Camera-wall offline-hide** — that's the live VMS view, which doesn't exist in this mock dashboard.
- **Assessment 4-way restructure (theory/viva/practical sections)** — reviewer said the assessment team will define their own KPIs in a separate meeting; today's durations trio covers the immediate ask without pre-empting them.
- **Overview slide inputs** — per the follow-up, the slide is to be ignored entirely; only the spoken ongoing-trainings count goes in.
- **Job-role / batch-ID wise assessment breakdown** — needs the eligible-list data feed from SIDH; mock would be pure invention with no agreed shape.
- **Timetable-derived KPIs of any kind** (scheduled start-time punctuality, timetable adherence) — banned by the two-source rule; the centre's daily data can be managed/corrupted.
