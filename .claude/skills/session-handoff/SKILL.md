---
name: session-handoff
description: Refresh CLAUDE_SESSION_HANDOFF.md before /clear or session exit. Captures current objective, decisions made this session, files touched, commands run, tests/evals status, risks, blockers, recommended next PM prompt, and recommended next agent-team formation. Hard rule: no secrets, no raw PHI, no log dumps, no API keys.
---

You are running the session-handoff skill. Your job is to refresh `./CLAUDE_SESSION_HANDOFF.md` (the canonical primer at the repo root) so the next session — whether the same user / Tate or a fresh session — can pick up cleanly without losing context.

This is the most important skill in the template. **Block the user's `/clear` if this hasn't run since the last meaningful state change.**

---

## Step 1 — Capture current state

Without asking the user, gather:

1. **Current git state:** `git status`, `git log -5 --oneline`, `git rev-parse HEAD`, current branch
2. **Files touched this session:** `git diff --stat HEAD` plus any uncommitted changes
3. **Tests / evals run this session:** scan recent terminal output, look for pytest results, eval outputs at `evals/results/`, CI status
4. **Risks surfaced this session:** any user-flagged concerns, any blockers raised, any "we should fix this later" comments

If you don't have visibility into one of these, ask the user briefly. Don't over-ask — most info is in your context already.

---

## Step 2 — Capture decisions

Decisions are the most important content. List every meaningful decision made this session:
- Architecture choices
- Trade-offs accepted (e.g., "deferred multi-turn attacks to Phase 2")
- Lead assignments / file ownership changes
- Scope changes
- Deadline adjustments

Format each as a short row in a Decisions table with: # / Decision / Chosen / Rationale.

---

## Step 3 — Recommend next moves

Two outputs here:

1. **Recommended next PM prompt** — the verbatim text the user (or the next session) should paste to begin the next session productively. Includes which docs to read first.

2. **Recommended next agent-team formation** — based on current state, what's the right next dispatch?
   - Solo Tate? Which work item next?
   - Dispatch a named lead (`/aria`, `/bram`, `/cleo`)? Which kickoff scope?
   - Multi-lead parallel dispatch? Provide both kickoff briefs ready-to-paste

---

## Step 4 — Write/refresh the handoff file

Open `./CLAUDE_SESSION_HANDOFF.md`. Either:
- It exists → REWRITE the major sections in place (preserve structure; refresh content)
- It doesn't exist → create it with full structure

Standard sections (in this order):

```markdown
# Claude Session Handoff — <Project Name>

**Date:** <YYYY-MM-DD>
**Session phase:** <e.g., "MVP build, Phase 1b">
**Next hard gate:** <gate name + date>
**Current branch / SHA:** <branch> @ <SHA>

---

## TATE — START HERE
[1-paragraph orientation: what state is the project in right now, what to do first]

### Read in this order before any work
1. ...
2. ...

### Recommended first action this session
[concrete first action; reference docs/MVP-WORK-PLAN.md item if applicable]

---

## Current Objective
[what we're trying to ship right now]

## Decisions Made (this session)
[table from Step 2]

## Files Touched (this session)
[bullets with brief why]

## Tests / Evals Status
[X pass / Y fail at SHA Z; or "N/A" if no evals exist yet]

## Risks + Blockers
[bullets; mark P0 / P1 / P2]

## Recommended Next PM Prompt
[verbatim copy-paste text]

## Recommended Next Agent-Team Formation
[Solo Tate, OR /aria + /bram parallel dispatch with kickoff briefs, etc.]

## Hard Rules (do not violate)
[project-specific hard rules; carry over from prior handoff if exists]

## Session Handoff Discipline
[reminder: refresh this file before /clear; no secrets / PHI / API keys / log dumps]
```

---

## Step 5 — Verify the handoff

Read it back. Check:
- [ ] No secrets / API keys / HMAC tokens / OpenRouter keys / Langfuse keys
- [ ] No raw PHI (real or sentinel patient IDs in narrative prose)
- [ ] No log dumps (>20 lines of output)
- [ ] All sections filled meaningfully (not "TBD" stubs)
- [ ] "Recommended Next PM Prompt" is paste-ready
- [ ] Decisions table reflects the session's actual decisions

If any check fails, fix and re-verify.

---

## Step 6 — (Option C only) Per-lead handoff coordination

If this session was a NAMED LEAD session (not Tate's main session), write to the lead-specific handoff file instead of the global one:

| Lead session | Write to |
|---|---|
| Tate | `./CLAUDE_SESSION_HANDOFF.md` (global; THIS skill's primary target) |
| Aria | `.project/handoffs/aria-handoff.md` |
| Bram | `.project/handoffs/bram-handoff.md` |
| Cleo | `.project/handoffs/cleo-handoff.md` |

Per-lead handoffs use a similar structure but scope is the lead's own workstream, not the global project state.

---

## Step 7 — Optional: write session summary

If the session was substantive (>1 hr of work, or shipped meaningful artifacts), also write a session recap to `.project/sessions/<YYYY-MM-DD>-<short-suffix>.md`. This is the long-form audit-trail version; the handoff file is the operational primer.

Format: chronological narrative + decision rationale + what worked + what didn't + what to repeat.

If the user has OneDrive / cloud-sync configured per their MEMORY discipline, mirror the session recap there too.

---

## Step 8 — Final confirmation

Print:
```
✓ CLAUDE_SESSION_HANDOFF.md refreshed at <YYYY-MM-DD HH:MM>
✓ Handoff covers: <decisions count> decisions, <files count> files touched, <risks count> risks
✓ Recommended next action: <one-line summary>
✓ Safe to /clear or exit.
```

If you wrote a session recap, mention its path too.
