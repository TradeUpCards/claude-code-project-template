# docs/ — {{PROJECT_NAME}} build docs

Single source of truth for everything Tate, the leads, and a future grader
need to find quickly. Keep it shallow — these are sprint docs, not a manual.

**This directory lives under `.project/`, which is gitignored.**
That means everything in here is **local-only and OneDrive-mirrored**, never
committed to GitLab / GitHub. PRD source material, paper sketches, and demo
materials are coordination artifacts — they don't belong in a public mirror or
a cohort submission repo.

## Layout

```
.project/docs/
├── prd/              PRD source documents (PDFs, original-language specs, etc.)
├── lesson-design/    Paper-before-pixels sketches, dialogue scripts, lesson flow,
│                     event contracts between leads. (Or for non-education projects:
│                     workplan, architecture sketches, ADR drafts.)
├── research/         Playtest notes, prior-art references, screenshots, links
├── demo/             Demo plan, video script, deliverable-specific docs (iPad
│                     roadmap, screen recordings, etc.) — the things you ship at
│                     the end of the sprint.
└── references/       Cross-project reference docs that ship with every new
                      project from this template (e.g., Presearch.pdf for the
                      Gauntlet pre-coding discovery process). Don't edit; these
                      are the shared playbook every project should have.
```

## What lives where

### `prd/`
Cohort/customer-provided source-of-truth documents. Don't edit these — they're
the spec you're shipping against. Examples: PRD PDFs, RFCs, design briefs.

### `lesson-design/` (rename if your project isn't a lesson)
Your team's own decisions about the work: paper sketches, scripts, event-
contracts, ADRs. Where leads coordinate the seams between their workstreams.

### `research/`
Notes from playing reference products, reading prior art, screenshots, links.
"What did I lean in to on the original?" notes — the input to design decisions.

### `demo/`
Fri-noon (or whenever-shipping) deliverables. Owned by whichever lead handles
deploy + demo prep. Demo video script, iPad roadmap, deliverable-specific docs.

### `references/`
Cross-project reference material that ships with **every** new project from
this template. Currently:
- `Presearch.pdf` — Gauntlet pre-coding discovery / project-defense framework.
  Read this before starting any new cohort project; it scopes what to ask
  yourself before you write code (problem framing, architecture, evals,
  verification, security, deployment, tradeoffs, rationale). The user-level
  `presearch-interview` skill is the conversational version of this doc.

Add to this folder when you find documents you want every future cohort
project to start with as default reading.

## Reading order for a new session

If you're Tate at session start:
1. `CLAUDE_SESSION_HANDOFF.md` (repo root) — your pickup point
2. `.project/docs/prd/` — source spec
3. `.project/docs/lesson-design/` — what the team has decided
4. `.project/in-flight.md` — coordination state

If you're Aria/Bram/Cleo, read your kickoff at
`.project/kickoff/<name>.md` first; it'll point you here.
