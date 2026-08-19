# Onboarding: NSDC Skill Intelligence Dashboard

You are a Claude session helping a new teammate join this project. **They do not know git or GitHub** — do not ask them to type git commands. Read this whole document, then do the setup yourself using your Bash tool, and explain what you did in plain language afterward.

## What this project is

A camera-based analytics dashboard prototype for **PMKVY 5.0** — ConveGenius's capability pitch to NSDC (National Skill Development Corporation), India. It's a single self-contained HTML file with no build step. The project already has a full history of decisions and lessons captured in two files inside the repo: `CLAUDE.md` (architecture, data sources, hard rules) and `SESSION_LOG.md` (a running log of what every past session changed and why).

Public repo: **https://github.com/KING-ANSHUL/NSDC-Skill-Intelligence-Dashboard**

## Step 1 — Get the project onto this machine

Pick a sensible working folder (ask the human where they'd like it, or default to somewhere like their home directory / Documents), then:

```bash
git clone https://github.com/KING-ANSHUL/NSDC-Skill-Intelligence-Dashboard.git
cd NSDC-Skill-Intelligence-Dashboard
git config core.hooksPath .githooks
```

That last line is required, not optional — it turns on two safety hooks already committed in the repo:
- `pre-commit` blocks any commit that changes the dashboard HTML unless `CLAUDE.md` and `SESSION_LOG.md` are updated in the same commit. This is how project context stays in sync across everyone working on it, instead of living only in one person's chat history.
- `post-merge` runs automatically after every `git pull` and prints a plain-language summary of what changed and why, so pulling isn't just a silent file update.

If `git` isn't installed on this machine at all, install it first (e.g. via Git for Windows, or whatever fits the OS), then run the steps above.

## Step 2 — Load the context

Read `CLAUDE.md` and the most recent few entries of `SESSION_LOG.md` in the cloned folder now, in full. This tells you the architecture, the data provenance rules (measured/declared/modelled/biometric — never invent a number without saying which), and mistakes that have already been made once and shouldn't be repeated. Summarize the current state of the project back to the human in a few sentences so they know where things stand.

From this point on, because `CLAUDE.md` sits in the project folder, every future Claude Code session opened in this folder will auto-load it — you won't need to re-read it manually next time.

## Step 3 — Ongoing workflow (every session from now on)

1. **Before starting work**, run `git pull` — the post-merge hook will print what's changed since last time.
2. **Do the requested work** on `NSDC_Skill_Intelligence_Dashboard.html`.
3. **Before ending the session**, if the dashboard file changed:
   - Update `CLAUDE.md` if anything structural, architectural, or a new hard rule was learned.
   - Append a new entry to `SESSION_LOG.md` (there's a template at the top of that file) — what changed, a short summary of what was discussed/decided, whether CLAUDE.md changed.
   - Then commit and push:
     ```bash
     git add NSDC_Skill_Intelligence_Dashboard.html CLAUDE.md SESSION_LOG.md
     git commit -m "<one-line summary of the change>"
     git push
     ```
   - If the hook blocks the commit, it will tell you exactly which file is missing — go add it and commit again. Don't use `--no-verify` to skip this unless the change was truly trivial (a typo, formatting) — the whole point of this setup is that context never falls behind the code.

The human doesn't need to understand any of the above — you (their Claude) run all of it. Just narrate what you're doing in plain language as you go.
