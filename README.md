# Claude Code Project Template

A template repo for spinning up Claude Code projects with **named-lead agent coordination, file-ownership enforcement, session handoffs, and PR-ready scaffolding** built in from minute one.

This template captures the patterns developed across multiple GauntletAI AgentForge weeks: the gauntlet-team-lead persona, named sprint leads (Aria/Bram/Cleo), in-flight workstream coordination, session handoff discipline, and Option A/B/C team-setup trade-offs.

---

## Quick start

```bash
# 1. Click "Use this template" on GitHub (NOT "Fork") to create your new repo
# 2. Clone your new repo
git clone https://github.com/YOUR_USER/YOUR_PROJECT.git
cd YOUR_PROJECT

# 3. Open in Cursor (or VS Code with Claude Code extension)
cursor .

# 4. Open a terminal, start Claude Code, then run:
/init-project
```

`/init-project` walks you through the setup interactively (project name, license, gitignore flavor, named-lead setup, hard deadlines, dual-use considerations) and fills in all the placeholders.

After init, `/tate` boots your team lead with a morning report.

---

## What you get out of the box

### Agent infrastructure
- **`.claude/agents/gauntlet-team-lead.md`** — generic team-lead persona; coordinates leads, enforces file ownership, watches deadlines
- **5 generic teammate types** — `implementation-lead`, `quality-lead`, `delivery-lead`, `observability-security-teammate`, `codebase-mapper` — usable on any project
- **3 named lead templates** (`aria.md.template`, `bram.md.template`, `cleo.md.template`) — filled by `/init-project` if you opt into Option C

### Skills (slash commands)
- **`/init-project`** — interactive project setup
- **`/session-handoff`** — refresh `CLAUDE_SESSION_HANDOFF.md` before `/clear`
- **`/tate`, `/aria`, `/bram`, `/cleo`** — load named-lead identity (filled by `/init-project`)

### Coordination scaffolding
- **`.gauntlet/PROJECT/`** (renamed to `.gauntlet/<project-name>/` on init) — `in-flight.md` (workstream rules + file ownership map), `kickoff/` (per-lead boot prompts), `handoffs/` (per-lead handoff files), `sessions/` (per-session recaps), `coordination/` (cross-lead negotiation threads), `candidates/_candidates.md` (story-capture seed)

### Repo-root primers
- **`CLAUDE_SESSION_HANDOFF.md`** — fresh-Tate-session primer; refreshed at every session exit
- **`LICENSE`** — chosen at init time (Apache 2.0 / MIT / Proprietary)
- **`.gitignore`** — chosen at init time (Python / Node / Generic)
- **`.claude/settings.json`** — permission rules (deny dangerous git, ask before destructive, allow safe reads)

### Documentation
- **`TEMPLATE_GUIDE.md`** — architecture of the template + Option A/B/C decision tree + when to use which pattern

---

## The Option A / B / C decision tree

When you're setting up a new project, you'll choose:

| Option | What you get | When to use |
|---|---|---|
| **A — Solo Tate, no agent infra** | Just `/session-handoff` skill + `CLAUDE_SESSION_HANDOFF.md`. No leads, no in-flight coordination, no kickoff files. | Solo work, < 2 days, no parallelism needed |
| **B — Generic teammate types** | Full agent infra; Tate dispatches generic `implementation-lead` / `quality-lead` / etc. on demand. No persistent named leads. | Short builds (3-5 days), max 1-2 concurrent dispatches, ad-hoc work |
| **C — Named leads (Aria/Bram/Cleo)** | Full agent infra + persistent named-lead identities + lead-specific kickoff/handoff paths + `/aria` `/bram` `/cleo` slash commands | Longer builds (5+ days), recurring multi-lead parallel work, identity persistence valuable |

`/init-project` asks which option you want and configures accordingly. You can also upgrade B → C later (~2-3 hrs).

See `TEMPLATE_GUIDE.md` for the full decision tree.

---

## What's NOT in this template (yet)

Tier 2 (could add in v0.2):
- `.pre-commit-config.yaml` templates (Python / Node)
- `.gitlab-ci.yml` template
- `pyproject.toml` / `package.json` templates
- `docs/ARCHITECTURE.md` outline template
- `docs/MVP-WORK-PLAN.md` template

Tier 3 (could add in v1.0):
- PRD-PDF → interactive HTML checklist extraction skill
- Interactive HTML work-plan tracker generator
- `THREAT_MODEL.md` template (security projects)
- `RESPONSIBLE_USE.md` template (dual-use AI projects)
- Multi-language scaffolds (Rust / Go)

If you want Tier 2 or Tier 3 ahead of those landing in the template, copy the relevant files from a project that has them (e.g., `ClinicalRedTeam` has Tier 3 examples).

---

## Lifecycle of a project built from this template

1. **Day 0:** Click "Use this template" → clone → `/init-project`
2. **Day 0–1:** Tate Phase 1a solo build (interfaces, schemas, first end-to-end work)
3. **Day 1–N:** Phase 1b parallel work — `/aria` + `/bram` (or generic teammate types) — each session ends with `/session-handoff`
4. **Day N–M:** Phase 2 polish — `/cleo` (or solo Tate) — same handoff discipline
5. **Final day:** Submission, demo, social — Tate drives directly

At every session boundary: `/session-handoff` writes/refreshes `CLAUDE_SESSION_HANDOFF.md` so the next session picks up cleanly.

---

## License

Apache 2.0 (this template). Projects you create from it choose their own license at `/init-project`.

---

## Acknowledgments

Patterns developed during GauntletAI AgentForge experimentation (Weeks 1-3). The named-lead pattern (Tate / Aria / Bram / Cleo), the file-ownership enforcement, the in-flight coordination doc, and the session-handoff discipline are battle-tested across multiple builds.
