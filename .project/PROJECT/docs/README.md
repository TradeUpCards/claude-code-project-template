# docs/ — {{PROJECT_NAME}} build docs

Single source of truth for everything Tate, the leads, and a future grader
need to find quickly. Keep it shallow — these are sprint docs, not a manual.

**This directory lives under `.project/{{PROJECT_SLUG}}/`, which is gitignored.**
That means everything in here is **local-only and OneDrive-mirrored**, never
committed to GitLab / GitHub. PRD source material, paper sketches, and demo
materials are coordination artifacts — they don't belong in a public mirror or
a cohort submission repo.

## Layout

```
.project/{{PROJECT_SLUG}}/docs/
├── prd/              PRD source documents (PDFs, original-language specs, etc.)
├── lesson-design/    Paper-before-pixels sketches, dialogue scripts, lesson flow,
│                     event contracts between leads. (Or for non-education projects:
│                     workplan, architecture sketches, ADR drafts.)
├── research/         Playtest notes, prior-art references, screenshots, links
└── demo/             Demo plan, video script, deliverable-specific docs (iPad
                      roadmap, screen recordings, etc.) — the things you ship at
                      the end of the sprint.
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

## Reading order for a new session

If you're Tate at session start:
1. `CLAUDE_SESSION_HANDOFF.md` (repo root) — your pickup point
2. `.project/{{PROJECT_SLUG}}/docs/prd/` — source spec
3. `.project/{{PROJECT_SLUG}}/docs/lesson-design/` — what the team has decided
4. `.project/{{PROJECT_SLUG}}/in-flight.md` — coordination state

If you're Aria/Bram/Cleo, read your kickoff at
`.project/{{PROJECT_SLUG}}/kickoff/<name>.md` first; it'll point you here.
