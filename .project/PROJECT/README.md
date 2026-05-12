# `.project/{{PROJECT_SLUG}}/` — Coordination & Session Memory

This directory holds **coordination artifacts that don't belong in the main repo** but need to persist across Claude Code sessions for {{PROJECT_NAME}}.

## Layout

```
.project/{{PROJECT_SLUG}}/
├── README.md             ← this file
├── in-flight.md          ← workstream rules + file ownership map (Option B/C)
├── kickoff/              ← per-lead boot prompts
│   ├── tate.md
│   ├── aria.md           ← Option C only
│   ├── bram.md           ← Option C only
│   └── cleo.md           ← Option C only
├── handoffs/             ← per-lead handoff files (write at /clear time; Option C only)
├── sessions/             ← per-session summaries (substantive sessions, write at /clear time)
│                            naming: YYYY-MM-DD-<short-suffix>.md
├── coordination/         ← cross-lead negotiation threads (Option C, when needed)
│                            naming: <lead-a>-<lead-b>-<topic>.md
└── candidates/           ← _candidates.md = story-capture moments worth /story-ifying later
```

## Conventions

- **Session summaries** (`sessions/YYYY-MM-DD-<suffix>.md`): write at end of substantive sessions (>1 hr work). No raw sensitive data, no secrets, no log dumps.
- **Handoffs**: lead-specific handoffs go to `handoffs/<lead-name>-handoff.md`. The GLOBAL handoff (Tate's) lives at repo-root `CLAUDE_SESSION_HANDOFF.md`.
- **Kickoff briefs**: when a kickoff prompt becomes valuable enough to reuse, save it in `kickoff/`. Pre-populated by `/init-project` for named leads.
- **Candidates**: low-threshold append every time something interesting happens during build. Story-ify (via `/story` or equivalent) when 3+ related entries pile up or when single entry is genuinely novel.
- **Coordination threads**: only when two leads need to negotiate scope/contracts. Don't duplicate workstream state from `in-flight.md`.

## What does NOT go here

- Code → project's source tree (e.g., `src/`)
- Tests → `tests/`
- Project docs → `docs/` or repo root
- Lead persona definitions → `.claude/agents/`
- Slash command skills → `.claude/skills/`
- Tate-to-Tate handoff → `CLAUDE_SESSION_HANDOFF.md` at repo root (NOT here)
