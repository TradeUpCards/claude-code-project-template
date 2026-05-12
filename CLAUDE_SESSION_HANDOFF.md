# Claude Session Handoff — {{PROJECT_NAME}}

**Date:** _(filled by /session-handoff at first run)_
**Session phase:** _(e.g., "Day 0 — initial setup; first session not yet run")_
**Next hard gate:** {{MVP_DEADLINE}}
**Current branch / SHA:** {{DEFAULT_BRANCH}} @ _(filled by /session-handoff)_

---

## TATE — START HERE

This is a freshly-initialized project from `claude-code-project-template`. Run `/tate` to get your morning report. Then your first action will likely be:

- Verify git state (commit done, remote(s) set up)
- Set up `.env` from `.env.example` if applicable
- Read `README.md` to confirm project framing
- If Option B/C: read `.project/{{PROJECT_SLUG}}/in-flight.md` for workstream coordination
- Begin Phase 1 work per your project's plan

After your first substantive session, run `/session-handoff` to refresh THIS file with real state.

---

## Current Objective

{{PROJECT_DESCRIPTION}}

_(Update this section after each session to reflect what we're trying to ship right now.)_

---

## Decisions Made

| # | Decision | Chosen | Rationale |
|---|---|---|---|
| _(filled as decisions accumulate)_ | | | |

---

## Files Touched (this session)

_(filled by /session-handoff with files modified + brief why)_

---

## Tests / Evals Status

_(N/A on Day 0; filled with X pass / Y fail at SHA Z once tests exist)_

---

## Risks + Blockers

### P0 (blocks shipping)
_(none on Day 0)_

### P1 (significant)
_(none on Day 0)_

### P2 (track but not blocking)
_(none on Day 0)_

---

## Recommended Next PM Prompt

_(filled by /session-handoff — verbatim text the user pastes to start the next session productively)_

For Day 0:
```
Read CLAUDE_SESSION_HANDOFF.md and README.md.
Restate the project mission and the first concrete action for Day 0.
Wait for my confirmation before any work.
```

---

## Recommended Next Agent-Team Formation

_(filled by /session-handoff)_

For Day 0 / first session: Solo Tate. No leads dispatched yet.

---

## Hard Rules (do not violate)

- No secrets / API keys / OAuth tokens / HMAC secrets in any committed file. Use `.env` (in `.gitignore`).
- No raw PHI / sensitive personal data in any committed file. Use synthetic / sentinel data only.
- No log dumps (>20 lines of raw output) in handoffs or session recaps.
- No `git push --force` to {{DEFAULT_BRANCH}} without explicit user request.
- Block session exit if `CLAUDE_SESSION_HANDOFF.md` hasn't been refreshed since the last meaningful state change.

_(Add project-specific hard rules here as they emerge.)_

---

## Session Handoff Discipline

Before `/clear` or session exit, run `/session-handoff` to refresh this file. The skill at `.claude/skills/session-handoff/SKILL.md` walks the steps.

If you're a named lead (Aria/Bram/Cleo in Option C), write to your lead-specific handoff at `.project/{{PROJECT_SLUG}}/handoffs/<name>-handoff.md` instead of this global file. Tate owns this global file.
