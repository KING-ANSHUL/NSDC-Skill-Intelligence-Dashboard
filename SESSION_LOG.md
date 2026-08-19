# Session Log

Append-only record of every session that changed the dashboard. This is the
practical substitute for syncing raw chat transcripts — Claude's actual
conversation history lives only on the machine/account it ran on and cannot
be shared through git — so each session writes a short summary here instead.
This file is what makes `git pull` tell you "what changed and why", not just
"these lines of HTML changed".

**New entries go at the bottom**, in this format:

```
## YYYY-MM-DD HH:MM (who/whose session) — one-line title
**Code changed:** which files, and a one-line description of the change
**Chat summary:** 3-6 lines — what was asked, what was explored, what was
  rejected and why, anything a teammate would otherwise have to ask about
**Context updated:** what changed in CLAUDE.md, if anything
```

---

## 2026-08-18 (Anshul's session) — Repo collaboration setup
**Code changed:** none (tooling only) — added CLAUDE.md, .gitignore, .gitattributes, `.githooks/pre-commit`, `.github/workflows/context-sync.yml`, this file
**Chat summary:** Made the project collaborative through Claude Code rather than a plain file share, so a teammate's Claude session gets full context automatically instead of the user re-explaining it each time. Set up a private GitHub repo, wrote CLAUDE.md documenting the dashboard's architecture and hard-won lessons, then hardened it: a pre-commit hook blocks any commit touching the dashboard HTML unless CLAUDE.md (and now this log) is updated in the same commit, and a GitHub Action re-checks the same rule server-side so it still catches a fresh clone that skipped the one-time local hook setup. Also removed `NSDC_Skill_Intelligence_Prototype.html`, which was superseded and no longer referenced anywhere.
**Context updated:** CLAUDE.md now documents the enforcement mechanism and the one-time collaborator setup (`git config core.hooksPath .githooks`).

## 2026-08-18 (Anshul's session) — Session summary + pull notification
**Code changed:** none (tooling only) — added `.githooks/post-merge`, extended `.githooks/pre-commit` and `.github/workflows/context-sync.yml` to also require this file
**Chat summary:** User wanted raw conversation to sync too, and for the teammate to be told what changed as soon as they pull. Raw Claude transcripts live only on the machine that ran them and can't be synced through git, so built the practical substitute instead: this SESSION_LOG.md, where every dashboard-changing session appends a short "code changed / chat summary / context updated" entry, enforced by the same pre-commit hook and CI check that already covered CLAUDE.md. Added a `post-merge` hook that fires automatically right after `git pull` and prints the new log entries plus a code diff stat. Verified the whole loop end-to-end with a throwaway second clone: made a test commit (hook correctly blocked it once for missing CLAUDE.md, then again required SESSION_LOG.md), pushed a valid one, ran `git pull` in the second clone, confirmed the notification printed, then reverted the test commit and deleted the throwaway clone.
**Context updated:** CLAUDE.md now documents SESSION_LOG.md's purpose and the post-merge notification hook.

## 2026-08-19 (Anshul's session) — Business-review corrections, part 1
**Code changed:** NSDC_Skill_Intelligence_Dashboard.html — Invigilator→Assessor, fingerprint→face-auth, eligible denominator on assessments, Head-to-Machine removed outright, People-on-System benchmark language (1.2 / assessor-at-screen), Round Durations trio (theory/viva/practical), Infra Readiness card, illustrative batch sizes (30/35/40/45, cap 45), centre-wide batch-cap note
**Chat summary:** Business team reviewed the demo with the NSDC-side stakeholder. Terminology corrections were direct quotes ("this is a face auth device", "it's called assessor, not invigilator"). Bigger conceptual fixes: assessment counts only eligible candidates (always ≤ enrolled, SIDH later); per-machine ratios are wrong for group practicals so that KPI is gone with no replacement; utilisation is the wrong lens for bed-type assets (presence check instead). Follow-up answers confirmed illustrative batch sizes and dropped the Candidate-to-Assessor ratio that had been proposed as a replacement. All still mock data — no live centre.
**Context updated:** CLAUDE.md hard-won lessons now carry the business-review overrides, the absolute two-source rule, the batch-size convention, and the historical-T−1/separate-from-live framing.

## 2026-08-19 (Anshul's session) — Business-review insights, part 2
**Code changed:** NSDC_Skill_Intelligence_Dashboard.html — Candidates by Hour Band chart beside the selected-room headcount line (with a day-peak threshold line), peak-vs-mean deviation chip, instructor Active Time % now carries a day-on-day sparkline with an actionability chip (repeated pattern = act, single dip = watch), heat() gained an optional colorOf so both heatmaps score RYG against references, Coverage split into Camera-Off vs Server-Off minutes with a frequency chip
**Chat summary:** These are the reviewer's own asks: hour-band occupancy ("this is a very good insight" — how many candidates per hour band against the day's peak), day-on-day instructor pattern ("KPIs should be actionable" — one low day is a sick day, a repeated pattern is a red flag), RYG on the heatmap, and the camera-off vs server-off distinction where frequency decides intentional vs technical. That last one comes from a real prior-vendor failure: of 18 integrated centres only 7-8 stayed on, and some gave deliberately wrong camera details so integration would fail. RYG thresholds were chosen for meaningfulness since the business hasn't defined them yet — occupancy scores against 75% of each room's own roll, workstation utilisation against 70%.
**Context updated:** CLAUDE.md Key mechanisms now documents `heat({colorOf})`, `hourBand()` and `offStats()`. The plan had said "none this commit" and the pre-commit hook correctly refused it — these are reusable mechanisms, and an undocumented mechanism is one the next person reimplements.

## 2026-08-19 (Anshul's session) — Business-review fixes, final batch and sweep
**Code changed:** NSDC_Skill_Intelligence_Dashboard.html — Ongoing trainings row (count of trades running across centres in scope, not an activity measure), "Camera Signals" button label, Historical·T−1 map badge, and the "All days" aggregate date option with every KPI/visualisation working on it
**Chat summary:** Closed the business-review batch. Follow-up answers settled the open questions: keep Committed Training Hours with its committed-vs-detected comparison, skip arbitrary-subset date selection, name the camera button "Camera Signals", and define ongoing trainings as the number of trades running (5 trades = 5 ongoing trainings, explicitly not active time). Three real bugs surfaced during execution that the plan had not predicted: DAYLBL was referenced in section 4b before its section-9 declaration; and the ALL aggregate's hand-written field list silently omitted cov, anyMin and personMin, crashing Coverage — replaced with generic numeric-field averaging so a future field cannot break it. The pre-commit hook also correctly rejected commit B, where the plan had claimed no CLAUDE.md change was needed while three reusable mechanisms had in fact been added.
**Context updated:** CLAUDE.md Key mechanisms now documents the ALL aggregate, its generic-averaging rule and the DAYLBL ordering trap.

