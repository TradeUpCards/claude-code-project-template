---
name: daily-sync
description: Get the lay of the land across all active leads at start of day, end of day, after a recovery, or any time before a cross-lead coordination decision. Each lead self-attests current state by refreshing their own handoff (read-only attestation, no code edits). Main session then synthesizes a coordinated status report. Default flow is lead-attested — fresh handoffs from each lead — because synthesis from stale handoffs gives stale conclusions. Hard rule — no secrets, no raw sensitive data (PHI / PII), no unnecessary log dumps.
---

You are running a multi-lead status sync from the main team-lead session.

## Purpose

Cross-lead state surveys are the highest-leverage moment in a multi-worktree workflow:

- **Start of day:** ground today's plan in actual current state, not yesterday's recovered notes.
- **End of day:** capture today's progress while it's fresh, before context evaporates.
- **After a recovery / disaster / merge / outage:** flush any stale state assumptions.
- **Before any cross-lead coordination decision:** confirm who's doing what, what's blocked, what's queued.

Without this discipline you fall into "I'll guess what each lead is doing from the artifacts" — and a prior session that synthesizes from stale handoffs can recommend a destructive command based on assumptions that no longer hold. The cure is fresh attestation from each lead.

## When to invoke

- Slash command `/daily-sync` (defaults to `start` mode if neither flag passed)
- `/daily-sync start` — morning sync
- `/daily-sync end` — evening sync (also triggers global `CLAUDE_SESSION_HANDOFF.md` refresh)
- `/daily-sync --quick` — escape hatch, read-only synthesis from on-disk handoffs (DISCOURAGED — produces stale conclusions; use only when leads are unavailable)
- User says: "where are we", "lay of the land", "team status", "lead status sync", "what's the state of each lead", "morning sync", "evening sync"
- Anytime stale handoffs would yield wrong conclusions (post-recovery, post-merge, after a multi-hour gap)

## When NOT to invoke

- Mid-task interrupting a lead session that's actively producing — the lead's own session-handoff at end-of-session is the right channel
- For a single lead's status (just read that lead's handoff directly)
- When the leads aren't running and won't be soon (use `--quick` or skip)

## Modes

### Default — lead-attested (use this unless explicitly overridden)

1. Main session emits the lead-side attestation prompt (below)
2. User pastes the prompt into each open lead session (Aria's, Bram's, Cleo's, or any custom-named leads in this project per `.project/<project-slug>/in-flight.md`). If a lead isn't currently open, user opens a Cursor terminal in the project root and runs `/aria` (or the relevant lead slash command) to start one.
3. Each lead executes its existing `session-handoff` skill in mid-session mode (target = its own `<lead>-handoff.md`), prints a 4-line TLDR, and returns to whatever it was doing — does NOT `/exit`
4. User says "done" or pastes the 3 TLDRs back to main session
5. Main session reads the freshly-attested handoffs + default-branch git state + `in-flight.md` + recent merges, produces the synthesis report

Wall-clock cost: ~5 min if leads run in parallel terminals, ~15 min if sequential. Real cost is each lead's ~5 min of attestation work.

### Escape hatch — `--quick` (read-only synthesis)

Only when leads are genuinely unavailable. Banner the output: ⚠️ **STALE-SYNTHESIS — leads not attested. Trust at your own risk.** Same synthesis report shape, but every claim is suspect because it's based on the on-disk handoffs which may be hours or days old.

## Lead-side attestation prompt (the thing the user pastes into each lead)

Emit this exact block, with `<lead>` replaced by the relevant identifier (`aria`, `bram`, `cleo`, etc.). The prompt is invocation-agnostic — works whether the lead just started or has been running for hours.

```
You are <lead>. Pause your current work for ~5 minutes to attest your status.
This is a /daily-sync attestation request. Read-only — no code edits, no test
runs, no commits, no merges.

1. Read your own current state:
   - `git status` (uncommitted changes)
   - `git log --oneline -5` (recent commits)
   - `git rev-list --count <default-branch>..HEAD` (commits ahead of the project's default branch)
   - Your row in `.project/<project-slug>/in-flight.md`
   - Your most recent ~30 turns of conversation history in this session for what's truly current vs. what's stale in your handoff

2. Run your existing session-handoff skill at `.claude/skills/session-handoff/SKILL.md`
   with these MID-SESSION adjustments:
   - Output target: `.project/<project-slug>/handoffs/<lead>-handoff.md` (not the global
     `./CLAUDE_SESSION_HANDOFF.md` — that's owned by main-checkout sessions)
   - At the top of the file, set:
     **Last attested:** <YYYY-MM-DD HH:MM Central> via /daily-sync
   - "Current Objective" = what you're ACTIVELY working on right now (not what
     was true at session start, not what the recovered handoff says)
   - "Last Session Status" = one of: active | paused | blocked
   - "Files In Flight" = currently locked or being edited (you'll re-edit)
   - "Blockers" = anything stopping forward progress (or "None")
   - "Next Action" = next concrete step + rough time estimate
   - All other sections per the session-handoff skill spec

3. After writing the handoff, print this exact 4-line TLDR block to stdout:

   ATTESTED: <lead> at <YYYY-MM-DD HH:MM Central>
   STATUS: <active | paused | blocked>
   WORKING ON: <one sentence — what you're doing right now>
   NEXT: <one sentence — next concrete step>
   BLOCKERS: <one sentence or "none">

4. Return to whatever you were doing. Do NOT `/exit`. Do NOT `/clear`. The
   main session will read your refreshed handoff after all leads attest.

Hard rules (per session-handoff skill):
- No secrets, no API keys, no raw sensitive data (PHI / PII / patient data) in the handoff or the TLDR
- No log dumps
- Reference don't reproduce — link to permanent docs (architecture / decisions / etc.)
  rather than copy rationale inline
```

## Synthesis report (what the main session produces after attestation)

Once the leads have attested (user signals "done" or pastes the TLDRs), main session reads the 3 fresh handoffs and default-branch git state, then produces this report:

```
# /daily-sync — <start | end> @ <YYYY-MM-DD HH:MM Central>

## Default-branch state
- SHA: <short hash>
- Recent merges (last 5): <list>
- Open MRs (best-effort from `git ls-remote origin refs/merge-requests/*/head` if available, else "check GitLab UI"): <list>

## Per-lead status
| Lead | Status | Branch | Ahead | Uncommitted | Working on | Next | Blockers | Handoff age |
|---|---|---|---|---|---|---|---|---|
| Aria | active | <feature-branch> | 3 | yes (2 files) | <one sentence> | <one sentence> | none | 5 min |
| Bram | active | <feature-branch> | 7 | clean | ... | ... | ... | 5 min |
| Cleo | paused | <feature-branch> | 0 | clean | ... | ... | none | 5 min |

## Cross-cutting

- File-ownership conflicts: <list any file owned by multiple leads, or "none">
- Schema-migration dependencies: <if any lead's branch needs another's migration first>
- In-flight blockers from in-flight.md cross-workstream notes: <list or "none">
- Stale handoffs: <flag any handoff > 24 h old or with no Last-attested timestamp>

## Suggested next moves

1. <Highest-priority action with the lead who should do it>
2. <Second>
3. <Third>

## Risks

| Risk | Likelihood | Mitigation |
|---|---|---|
| ... | ... | ... |
```

In `end` mode, also refresh `./CLAUDE_SESSION_HANDOFF.md` capturing the cross-lead state for tomorrow's session.

## Hard rules

1. **Lead-attested by default.** `--quick` is the escape hatch, not the default. The whole point of this skill is fidelity; speed-optimizing the default would defeat it.
2. **Each lead writes ITS OWN handoff.** Main session does not write to `<lead>-handoff.md` — only the owning lead does.
3. **Read-only attestation.** Leads MUST NOT make code edits, run tests, or commit during attestation. The instruction prompt enforces this.
4. **No `/exit` from leads during attestation.** They return to their work after the TLDR.
5. **No sensitive data / secrets.** Per session-handoff skill rules — references not reproductions, no log dumps, no API keys, no real patient/user identifiers (synthetic / sentinel data only).
6. **Skip leads not currently active.** If a lead's row in `in-flight.md` is "(empty)" or absent, no attestation needed for them.

## What this skill deliberately does NOT do

- Doesn't write to lead handoff files (lead self-attests)
- Doesn't open lead sessions (use `start_<lead>` for that)
- Doesn't enforce or correct lead behavior (purely diagnostic + coordinating)
- Doesn't decide who works on what (synthesis surfaces options; user decides)
- Doesn't replace `session-handoff` (which is end-of-session per-Claude-session) or `finish_<lead>` (which is workstream teardown)

## Verification checklist before producing the synthesis

- [ ] All currently-active leads (per `in-flight.md`) have attested within the last hour
- [ ] Each lead's handoff has a "Last attested" timestamp from this run
- [ ] Each TLDR's STATUS field matches the lead's handoff `Last Session Status`
- [ ] No sensitive data / secret material in any handoff or TLDR
- [ ] Master git state matches what's reported in the report (verify with a fresh `git log --oneline -5`)
- [ ] If `--quick` was used, the synthesis carries a STALE-SYNTHESIS banner

## Failure modes to actively prevent

- **Synthesis from stale handoffs being treated as ground truth.** This is the trap. The `--quick` escape hatch must always be banner-flagged.
- **Main session writing to lead handoff files.** Violates ownership; lead loses provenance over its own state.
- **Leads doing actual work during "attestation."** Read-only is the rule. Edits during attestation pollute the state being measured.
- **Skipping the synthesis step.** Without it the user has 3 attested handoffs but no coordinated picture; they'll do the synthesis in their head and miss cross-cutting issues.

---

*Companion to `session-handoff` (per-session). Use whenever stale handoffs would yield wrong conclusions.*
