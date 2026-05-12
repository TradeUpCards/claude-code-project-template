# Memory Patterns — How Claude Sessions Persist Knowledge

This doc captures the memory + session-summary patterns developed across GauntletAI AgentForge. It applies to projects built from this template AND to any other Claude Code project. Read this once; the patterns become muscle memory.

---

## Three layers of persistence

Claude Code sessions can lose context for three reasons: `/clear`, `/compact`, or hitting the token limit. Persistence is layered across three places, each with different staying power:

| Layer | Where | When used | What goes here |
|---|---|---|---|
| **Auto-memory** | `~/.claude/projects/<project-slug>/memory/MEMORY.md` | Auto-loaded by Claude at every session start in that project | Short-form facts the user wants permanently remembered (file paths, decisions, conventions, gotchas). Cross-session, persistent forever. |
| **Session handoff** | `./CLAUDE_SESSION_HANDOFF.md` (repo root) | Read at every session start by the gauntlet-team-lead persona; refreshed at session end | Operational state: current objective, decisions THIS session, files touched, tests/evals status, next PM prompt, recommended team formation |
| **Session summary** | `.gauntlet/<slug>/sessions/<YYYY-MM-DD>-<suffix>.md` | Written at end of substantive sessions; read on-demand for historical context | Long-form narrative recap: what happened, why decisions were made, what worked, what didn't |

Plus, in Option C named-lead projects:

| Layer | Where | When used | What goes here |
|---|---|---|---|
| **Lead handoff** | `.gauntlet/<slug>/handoffs/<lead-name>-handoff.md` | Per-lead — written by Aria/Bram/Cleo at their session exit | Lead-specific operational state mirroring the global handoff but scoped to that lead's workstream |

---

## Auto-memory — the smallest, most-load-bearing layer

### What it is

Claude Code auto-loads the user-level memory file at every session start. The file's content gets prepended to the system prompt invisibly so Claude "remembers" without you typing anything.

Path on Windows: `C:/Users/<you>/.claude/projects/<project-slug>/memory/MEMORY.md`
Path on macOS / Linux: `~/.claude/projects/<project-slug>/memory/MEMORY.md`

`<project-slug>` is auto-derived from your project's filesystem path with `/` replaced by `-` (e.g., `C--Dev-GauntletAI-AgentForge`).

### How facts get added

Three pathways:

1. **Explicit** — user says "remember this" / "save this to memory" / "make this persistent" → Claude appends a short entry
2. **Implicit via /remember** — if the harness has a `/remember` skill installed, that's the formal entry point
3. **End-of-session refresh** — when Claude writes the session-handoff, particularly important facts can also be promoted to memory

### What belongs in memory vs handoff

| Goes in MEMORY.md | Goes in CLAUDE_SESSION_HANDOFF.md |
|---|---|
| Permanent facts (file paths, conventions, person names, repo URLs) | Current operational state (what we're doing right now) |
| Project-wide patterns (e.g., "this fork is GitLab-primary, GitHub is mirror only") | Decisions made this session |
| Gotchas that bit us once and shouldn't bite again | Files touched this session |
| Pointer entries to longer artifacts ("see project_week2_state.md") | Tests/evals run this session |
| Tooling preferences (e.g., "user prefers `glab` CLI") | Risks + blockers right now |

A useful test: if the answer to "is this still true 6 months from now?" is yes → MEMORY.md. If "this is just where we are this week" → handoff.

### Pattern: short entries with pointers to long-form details

The auto-memory file gets injected into every session's context. **Keep entries short** — 1-2 lines each. For details, point at a longer artifact in the project's `.gauntlet/` directory.

Good entry:
```markdown
- [AgentForge week 2 state](project_week2_state.md) — multimodal evidence agent (Docling+Haiku, LangGraph, Qdrant, hybrid RAG); 50-case eval CI is the hard gate
```

Bad entry (too long, eats context):
```markdown
- AgentForge week 2 is about multimodal evidence agents using Docling for PDF parsing
  and Haiku for extraction. We chose LangGraph over CrewAI because of the fine-grained
  state control + Langfuse instrumentation. Qdrant runs as a sidecar container...
  [50 more lines]
```

### Hard rules for memory

- **No secrets** — API keys, HMAC tokens, OAuth credentials never in memory
- **No raw sensitive data** — PHI, PII, real patient identifiers, customer data
- **No log dumps** — memory is meant for facts, not transcripts
- **No screenshots / file contents pasted in** — reference them by path

---

## Session handoff — the operational layer

### What it is

A single markdown file at repo root: `CLAUDE_SESSION_HANDOFF.md`. Refreshed at end of every session by the `/session-handoff` skill (built-in to this template). Read at start of every session by the `gauntlet-team-lead` persona before any work.

### Why it matters

Without this file, every fresh session starts from zero — re-reads the codebase, re-discovers what was decided, re-asks the user "where are we?". With it, the next session opens with full operational context in one read.

### Standard sections

```markdown
# Claude Session Handoff — <Project Name>

**Date:** YYYY-MM-DD
**Session phase:** <e.g., "MVP build, Phase 1b">
**Next hard gate:** <gate name + date>
**Current branch / SHA:** <branch> @ <SHA>

## TATE — START HERE
[1-paragraph orientation: what state is the project in, what to do first]

### Read in this order before any work
1. ...

### Recommended first action this session
[concrete first action]

## Current Objective
## Decisions Made
## Files Touched
## Tests / Evals Status
## Risks + Blockers
## Recommended Next PM Prompt
## Recommended Next Agent-Team Formation
## Hard Rules (do not violate)
```

### Hard rules

- Refresh at every session end. **Never `/clear` without refreshing.**
- No secrets, no raw sensitive data, no log dumps, no API keys
- Reference don't reproduce — link to permanent docs (architecture / decisions) rather than copy rationale inline

---

## Session summary — the audit trail

### What it is

A long-form narrative recap of a substantive session, written to `.gauntlet/<project-slug>/sessions/<YYYY-MM-DD>-<short-suffix>.md`.

### When to write one

- Session was >1 hour of substantive work
- Shipped a meaningful artifact (feature, doc, refactor)
- Made non-trivial decisions worth remembering
- Hit a notable failure mode or recovered from one

NOT every session — short bug fixes, doc tweaks, conversational planning don't need recaps.

### What goes in it

Chronological narrative + decision rationale + what worked + what didn't + what to repeat. Long-form prose, not just bullet lists.

The session-handoff is the operational primer; the session summary is the historical record. Both can exist for the same session, with different scopes.

### Mirror to OneDrive (or equivalent)

If you have cloud-synced personal storage, mirror the session summary there too. Reasons:
- Survives if the repo is deleted / reorganized
- Searchable across all projects from one place
- Cross-references in personal notes / portfolio

### Naming convention

`YYYY-MM-DD-<short-suffix>.md` — e.g., `2026-05-11-w3-kickoff.md` or `2026-05-13-phase1a-shipped.md`. Sortable alphabetically = sortable chronologically.

---

## Lead handoffs — Option C only

### Why per-lead

In Option C named-lead projects (Aria/Bram/Cleo with persistent identity), each lead gets its own handoff at `.gauntlet/<slug>/handoffs/<lead>-handoff.md`. This separates lead-scoped operational state from the global state.

The global `CLAUDE_SESSION_HANDOFF.md` is owned by Tate (main session). Each lead's handoff is owned by that lead.

### When `/daily-sync` invokes them

The `/daily-sync` skill (built into this template) coordinates lead handoffs in a "lead-attested" pattern: each lead refreshes its own handoff with current state, then the main session synthesizes a coordinated status report.

This prevents the failure mode where the main session synthesizes from stale handoffs and produces wrong conclusions.

---

## Story candidates — the future-portfolio layer

### Where

`.gauntlet/<slug>/candidates/_candidates.md` — project-specific story candidates (within the workstream coordination dir)
`.gauntlet/stories/_candidates.md` — cross-project / portfolio-level story candidates

### What

Low-threshold append every time something interesting happens during the build. Format: 1-line note with date + category + min facts to recover later.

When a candidate accumulates enough material (or a single moment is genuinely novel), promote via `/story` skill (built into this template) to a full structured story with quality-bar enforcement.

### Why two locations

- **Project-specific candidates** stay with the project (in `.gauntlet/<slug>/candidates/`)
- **Cross-project / portfolio candidates** live in `.gauntlet/stories/` (the `/story` skill's default `STORIES_DIR`)

Both are valid; pick based on whether the moment is project-specific or portfolio-relevant.

---

## The discipline loop

```
Session start
├── Claude auto-loads MEMORY.md (invisible)
├── Tate persona reads CLAUDE_SESSION_HANDOFF.md
├── Tate gives 4-line morning report
└── Tate waits for user confirmation before any work

During session
├── Substantive work happens
├── Story-worthy moments → append to candidates
└── Mid-day status check (Option C) → /daily-sync

Session end
├── /session-handoff → refreshes CLAUDE_SESSION_HANDOFF.md
├── If substantive (>1 hr): write session summary to .gauntlet/<slug>/sessions/
├── Promote permanent facts to MEMORY.md (via /remember or manual edit)
└── Safe to /clear or exit
```

---

## Anti-patterns to avoid

- **`/clear` without `/session-handoff`** — guaranteed context loss; the next session has no primer. The gauntlet-team-lead persona is configured to BLOCK exit if the handoff is stale.
- **Session summary written but no handoff refresh** — handoff is what the next session reads first; summary is read on-demand. If only one gets written, it should be the handoff.
- **Memory entries that are 50 lines long** — eats context permanently. Use pointer entries.
- **Memory file with secrets / PHI** — auto-loaded into every session's context; one leak persists everywhere.
- **Lead writing to global handoff** — only Tate owns `CLAUDE_SESSION_HANDOFF.md`. Leads write to `.gauntlet/<slug>/handoffs/<lead>-handoff.md`.
- **Synthesis from stale handoffs treated as ground truth** — this is what `/daily-sync`'s lead-attested mode prevents. The `--quick` escape hatch must always be banner-flagged.

---

## Discoverability — how to find old facts later

When you need to recover something from past sessions:

| Question | Where to look |
|---|---|
| "What was decided about X?" | Search `CLAUDE_SESSION_HANDOFF.md` decisions table; if not there, search `.gauntlet/<slug>/sessions/*.md` |
| "When did we ship Y?" | `git log --grep="Y"` + `.gauntlet/<slug>/sessions/*.md` for context |
| "Why did we choose X over Y?" | Decisions table in handoff; if architectural, also `ARCHITECTURE.md` decision-log section |
| "What's the current operational state?" | `CLAUDE_SESSION_HANDOFF.md` Current Objective + Files Touched |
| "What were Aria/Bram/Cleo doing?" | Their handoff files at `.gauntlet/<slug>/handoffs/` |
| "What story candidates exist?" | `.gauntlet/<slug>/candidates/_candidates.md` (project) + `.gauntlet/stories/_candidates.md` (cross-project) |
| "Permanent facts about this project?" | `~/.claude/projects/<slug>/memory/MEMORY.md` |

---

## Integration with skills built into this template

| Skill | Memory layer it touches | When |
|---|---|---|
| `/use-template` | First-run; doesn't write to memory but sets up the directory structure all other layers depend on | Day 0 |
| `/session-handoff` | Writes/refreshes `CLAUDE_SESSION_HANDOFF.md`; optionally writes session summary | End of every session |
| `/daily-sync` | Coordinates per-lead handoff refreshes; produces synthesis report | Start of day / end of day / after recovery |
| `/story` | Reads `.gauntlet/stories/_candidates.md`; writes structured story file; updates candidates + index | When a candidate is ready to promote |
| `/build-audit` | Reads handoffs + decisions docs as discovery input; writes audit report to `.gauntlet/audits/<date>-build-audit.md` | End of week / before submission |
| `/tate`, `/aria`, `/bram`, `/cleo` | Each reads its own handoff at session start; persona system prompt enforces handoff discipline at exit | Lead session boot/exit |

User-level skills that interact with memory:
- `presearch-interview` — produces ARCHITECTURE.md (read by handoff sections referencing architecture)
- `prd-checklist` — produces PRD-REQUIREMENTS.html (read by sessions tracking build progress)
- `weekly-prd` — produces `.gauntlet/week<N>/prd.md` + `tasks.md`
- `agent-team-setup` — bootstraps multi-agent infra (replicates parts of this template's structure into existing projects)

---

## TLDR for new users of this template

1. Read this doc once; don't reread. The patterns become automatic.
2. Always `/session-handoff` before `/clear`. The gauntlet-team-lead persona will block exit otherwise.
3. Use `/story` aggressively — append candidates at low threshold; promote to full stories at high threshold.
4. Use `/daily-sync` (Option C) when you've been working multi-lead for >2 hours.
5. Use `/build-audit` end-of-week or before submission deadlines.
6. Promote truly permanent facts (file paths, conventions) to `~/.claude/projects/<slug>/memory/MEMORY.md` so they're auto-loaded next session.

The whole system exists to prevent the failure mode of "I worked on this for 3 weeks and now I can't remember what I decided or why." If you find yourself re-asking yourself questions you should already have answered, check whether the answer is captured in one of the layers above. If not, capture it.
