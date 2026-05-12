# Template Guide — Architecture of `claude-code-project-template`

This document explains how the template is structured, what every file does, and how to choose between Options A / B / C when setting up a new project.

---

## File-by-file inventory

```
claude-code-project-template/
├── README.md                                 # User-facing quick start
├── TEMPLATE_GUIDE.md                         # This file
├── MEMORY_PATTERNS.md                        # How Claude sessions persist knowledge (auto-memory, handoffs, summaries, candidates) + discipline loop + anti-patterns
├── WORKTREE_PATTERNS.md                      # Mode 1 (single-checkout) vs Mode 2/3 (per-lead worktrees + junctions); OneDrive mirroring; failure modes
├── CLAUDE_SESSION_HANDOFF.md                 # Primer for fresh Tate sessions (you fill in placeholders at init)
├── LICENSE                                   # Apache 2.0 (template's own license; project picks its own)
├── .gitignore                                # Generic; replaced by /init-project with Python or Node variant
│
├── .claude/
│   ├── settings.json                         # Permission rules (deny/ask/allow); generic, works for any project
│   ├── agents/
│   │   ├── gauntlet-team-lead.md             # Generic team-lead persona (Tate). Loaded automatically when Cursor opens project.
│   │   ├── implementation-lead.md            # Generic teammate type — bounded code edits
│   │   ├── quality-lead.md                   # Generic teammate type — eval / tests / CI gate
│   │   ├── delivery-lead.md                  # Generic teammate type — README / docs / submission polish
│   │   ├── observability-security-teammate.md # Generic teammate type — logging review / no-PHI / security audit
│   │   ├── codebase-mapper.md                # Generic teammate type — read-only file inventory
│   │   ├── aria.md.template                  # OPTION C: named lead 1 persona (filled by /init-project)
│   │   ├── bram.md.template                  # OPTION C: named lead 2 persona
│   │   └── cleo.md.template                  # OPTION C: named lead 3 persona
│   └── skills/
│       ├── use-template/SKILL.md             # The setup walkthrough. Run this FIRST after cloning.
│       ├── session-handoff/SKILL.md          # Refresh CLAUDE_SESSION_HANDOFF.md before /clear
│       ├── story/SKILL.md                    # Capture moments as interview-ready stories
│       ├── build-audit/SKILL.md              # PM-style audit of what was built vs claimed (8 lenses)
│       ├── daily-sync/SKILL.md               # Multi-lead status sync (Option C; lead-attested)
│       ├── tate/SKILL.md.template            # /tate slash command (filled by /use-template)
│       ├── aria/SKILL.md.template            # OPTION C: /aria slash command
│       ├── bram/SKILL.md.template            # OPTION C: /bram slash command
│       └── cleo/SKILL.md.template            # OPTION C: /cleo slash command
│
├── .project/
│   ├── PROJECT/                              # Renamed to .project/<project-slug>/ by /use-template
│   │   ├── README.md                         # Directory layout doc
│   │   ├── in-flight.md.template             # Workstream rules + file ownership map
│   │   ├── kickoff/                          # Per-lead boot prompts (filled by /use-template)
│   │   │   ├── tate.md.template
│   │   │   ├── aria.md.template              # OPTION C only
│   │   │   ├── bram.md.template              # OPTION C only
│   │   │   └── cleo.md.template              # OPTION C only
│   │   ├── handoffs/                         # Per-lead handoff files (written as work happens)
│   │   │   └── .gitkeep
│   │   ├── sessions/                         # Per-session recaps (written at /clear)
│   │   │   └── .gitkeep
│   │   ├── coordination/                     # Cross-lead negotiation threads (when needed)
│   │   │   └── .gitkeep
│   │   └── candidates/
│   │       └── _candidates.md.template       # Project-specific candidate moments seed
│   └── stories/                              # Interview-story repository (used by /story skill)
│       ├── _template-design-defense.md       # Template for "why X over Y?" stories
│       ├── _template-star.md                 # Template for "tell me about a time..." stories
│       ├── _candidates.md                    # Story candidates backlog (low-threshold append)
│       └── README.md                         # Index + how to use during interviews
│
├── LICENSE_OPTIONS/                          # Pick one at /init-project; chosen file becomes ./LICENSE
│   ├── apache-2.0.txt
│   ├── mit.txt
│   └── proprietary.txt
│
└── GITIGNORE_OPTIONS/                        # Pick one at /init-project; chosen file becomes ./.gitignore
    ├── python.gitignore
    ├── node.gitignore
    └── generic.gitignore
```

---

## Placeholder convention

Template files use `{{PLACEHOLDER}}` syntax. `/init-project` substitutes:

| Placeholder | Source | Example value |
|---|---|---|
| `{{PROJECT_NAME}}` | User input | `ClinicalRedTeam` |
| `{{PROJECT_SLUG}}` | Lowercased + dash-separated from PROJECT_NAME | `clinical-redteam` |
| `{{PROJECT_DESCRIPTION}}` | User input (1-3 sentences) | `Multi-agent adversarial security platform...` |
| `{{DEFAULT_BRANCH}}` | User input (default: `main`) | `main` |
| `{{MVP_DEADLINE}}` | User input | `Tuesday 2026-05-13 11:59 PM` |
| `{{FINAL_DEADLINE}}` | User input (optional) | `Friday 2026-05-16 noon` |
| `{{TARGET_URL}}` | User input (optional; for integration/security projects) | `https://target.example.com` |
| `{{ARIA_WORKSTREAM}}` | User input (Option C only) | `Implementation: agent code + persistence + CLI` |
| `{{ARIA_OWNED_FILES}}` | User input (Option C only) | `src/clinical_redteam/**`, `tests/agents/**`, ... |
| `{{BRAM_WORKSTREAM}}` | User input (Option C only) | `Quality + Eval Content` |
| `{{BRAM_OWNED_FILES}}` | User input (Option C only) | `evals/**`, `tests/meta/**`, ... |
| `{{CLEO_WORKSTREAM}}` | User input (Option C only) | `Delivery + Polish` |
| `{{CLEO_OWNED_FILES}}` | User input (Option C only) | `README.md`, `SETUP.md`, `dashboard.html`, ... |

After `/init-project` runs:
- Files lose their `.template` extension
- Placeholders are replaced
- `.project/PROJECT/` is renamed to `.project/<project-slug>/`
- Unused option files (e.g., aria/bram/cleo if Option B chosen) are removed

---

## Option A / B / C decision tree

### Option A: Solo Tate (no agent infrastructure)

**Pick this if:**
- Solo work, no plans for parallel sessions
- Build is < 2 days
- No need for persistent identity across sessions

**What `/init-project` removes:**
- All `.claude/agents/aria.md.template`, `bram.md.template`, `cleo.md.template`
- All `.claude/skills/aria/`, `bram/`, `cleo/`
- `.project/PROJECT/kickoff/aria.md.template`, `bram.md.template`, `cleo.md.template`
- `.project/PROJECT/in-flight.md.template`
- `.project/PROJECT/handoffs/`, `coordination/` (you don't need them)

**What you keep:**
- `.claude/agents/gauntlet-team-lead.md`
- `.claude/skills/init-project/`, `session-handoff/`, `tate/`
- `.project/<project-slug>/sessions/`, `candidates/`
- `CLAUDE_SESSION_HANDOFF.md`

**Upgrade path:** A → B is ~30 min (just keep the generic teammate type files); A → C is ~2-3 hrs.

---

### Option B: Generic teammate types (no named leads)

**Pick this if:**
- Short build (3-5 days)
- Max 1-2 concurrent agent dispatches
- Ad-hoc work where teammate identity doesn't need to persist
- Future-proofing for occasional parallel work without committing to full named-lead overhead

**What `/init-project` keeps:**
- All Option A files
- The 5 generic teammate types in `.claude/agents/`

**What `/init-project` removes:**
- All Option C files (aria/bram/cleo personas, slash commands, kickoff files)

**How you dispatch in Option B:**
```python
Agent(
  subagent_type: implementation-lead,  # or quality-lead, delivery-lead, etc.
  prompt: "[full kickoff brief inline]",
  run_in_background: true
)
```

Multiple dispatches in a single message run in parallel.

**Upgrade path:** B → C is ~2-3 hrs (write 3 named-lead persona files + 3 slash command skills + 3 kickoff files; update gauntlet-team-lead.md).

---

### Option C: Named leads (Aria / Bram / Cleo with persistent identity)

**Pick this if:**
- Longer build (5+ days)
- Recurring multi-lead parallel work
- Persistent identity / voice across sessions valuable
- Lead-specific kickoff + handoff paths needed for cross-session continuity

**What `/init-project` keeps:**
- All Option B files
- Named-lead personas (aria/bram/cleo)
- Slash command skills (/aria, /bram, /cleo)
- Per-lead kickoff files in `.project/PROJECT/kickoff/`
- Per-lead handoff dirs (handoffs/)
- Cross-lead coordination dir (coordination/)
- `.project/PROJECT/in-flight.md` workstream rules + file ownership map

**How you dispatch in Option C:**

Method 1 (slash commands in separate Cursor terminals):
```
# Terminal 1
/aria

# Terminal 2
/bram
```

Method 2 (background dispatches from Tate's session):
```python
Agent(subagent_type: aria, prompt: "...", run_in_background: true)
Agent(subagent_type: bram, prompt: "...", run_in_background: true)
```

Both methods work; pick per session preference.

**Customization:** If you want different lead names (not Aria/Bram/Cleo), `/init-project` asks. The persona structure stays the same; just renames.

---

## Lifecycle of a project built from this template

```
Day 0
├── Click "Use this template" on GitHub → clone → cd into repo
├── Open Cursor / Claude Code in the new repo
├── Run /init-project
│   ├── Answer questions (project name, license, gitignore, option A/B/C, deadlines, target URL)
│   ├── Template fills placeholders, renames files, removes unused options
│   ├── First commit ("chore: init <project> from claude-code-project-template")
│   └── Optionally: gh repo create (or glab repo create) + first push
└── Run /tate to get morning report

Day 1
├── Run /tate at session start → 4-line morning report
├── Solo Tate work on Phase 1a / setup items
├── Mid-session: capture story moments to .project/<slug>/candidates/_candidates.md
└── Before /clear: run /session-handoff → updates CLAUDE_SESSION_HANDOFF.md

Day 2-N (Option C example)
├── Run /tate, get morning report (which now includes lead status from yesterday's handoffs)
├── Tate decides: dispatch /aria + /bram in parallel, OR continue solo
├── Each lead session: read kickoff → read own handoff → do work → write own handoff before exit
└── Tate synthesizes, updates global CLAUDE_SESSION_HANDOFF.md, runs /session-handoff

Final day
├── Polish + demo work
├── Push final
└── /session-handoff captures final state
```

---

## When to evolve this template

The template is opinionated based on AgentForge experience. Evolve it when:

- **A pattern repeats across 3+ projects** → bake it in (e.g., dual-push GitLab+GitHub remote setup is a candidate)
- **A skill saved >30 min in 2+ projects** → port it (e.g., PRD-PDF → interactive HTML checklist extraction)
- **A hole bit you twice** → add the file/skill that prevents it (e.g., I forgot to copy `/session-handoff` to ClinicalRedTeam — that's why it's in this template)

Don't evolve it when:
- A pattern only fit one project's specific shape (e.g., HMAC-signed target client is ClinicalRedTeam-specific, not template-worthy)
- A skill is faster to write fresh per project than to template
- The template would balloon with rarely-used scaffolding

---

## Versioning

Tag releases as you make breaking changes:
- `v0.1.0` — Initial Tier 1 release (this version)
- `v0.2.0` — Tier 2 additions (CI / pre-commit / pyproject scaffolds)
- `v1.0.0` — Tier 3 additions (PRD extraction skill, interactive HTML generator, security/AI doc templates)

Projects pin to a tag at clone time so template evolution doesn't surprise existing builds.
