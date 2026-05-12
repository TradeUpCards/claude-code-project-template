---
name: build-audit
description: PM-style audit of what was built, why decisions were made, what's deferred, what's a differentiator beyond the PRD, what the PRD itself got wrong, what a hostile reviewer would attack, and how to communicate to graders / employers / stakeholders. Invoke when user types `/build-audit`, asks for a "build review", "scope audit", "PRD compliance check", "what did we ship this week", "what would a reviewer push back on", or "what's wrong with our PRD". Distinct from technical reviews (agent-review, system-architecture-review, ai-security-review) which audit code quality — this audits coverage, decisions, narrative, and defensibility.
config:
  # Override these per project. The skill body refers to them by name.
  AUDITS_DIR: .project/audits
  # Discovery hint paths — the skill SEARCHES for these patterns by default.
  # Override only if your project uses non-standard names.
  PRD_PATTERNS:
    - .project/week*/prd.md
    - PRD.md
    - prd.md
    - docs/prd.md
  ARCHITECTURE_PATTERNS:
    - ARCHITECTURE.md
    - W*_ARCHITECTURE.md
    - docs/architecture.md
  DECISIONS_PATTERNS:
    - DECISIONS.md
    - decisions/*.md
    - docs/adr/*.md
  HANDOFF_PATTERNS:
    - CLAUDE_SESSION_HANDOFF.md
    - .project/week*/handoff*.md
    - HANDOFF.md
    - STATUS.md
  RISKS_PATTERNS:
    - AUDIT.md
    - RISKS.md
    - SECURITY.md
    - docs/threat-model.md
  SLIDES_PATTERNS:
    - .project/**/*slides*.md
    - .project/**/*video*.md
    - docs/slides/*.md
---

# Build-audit skill

You are conducting an interactive, analyst-style audit of what was actually built versus what was claimed/required, plus what reviewers will probe.

## Configuration

Path defaults are in the frontmatter `config:` block. To use this skill in another project: copy this file, edit `config:`, done.

The `*_PATTERNS` keys are glob lists the discovery phase searches in order. The first matching file in each category becomes the primary source for that document type. The body refers to paths by config-key name (`${AUDITS_DIR}`).

## Mindset

This is a **coverage and narrative audit**, not a code-quality audit. The technical-review skills (`agent-review`, `system-architecture-review`, `ai-security-review`, `llm-observability-review`, `synthetic-data-review`) answer *"is the code good?"*. This skill answers seven different questions:

1. **Coverage** — Did we build what we said we'd build?
2. **Decisions** — Why did we make the choices we made, and is the rationale documented?
3. **Deferrals** — What's silently deferred vs explicitly documented?
4. **Differentiators** — What did we add beyond the PRD that's a competitive edge?
5. **PRD critique** — What does the PRD itself contradict, leave ambiguous, or get wrong, and how did we resolve it?
6. **Red-team** — What would a hostile reviewer / employer / grader push back on, and what's our prepared defense?
7. **Communication** — Can we explain it to a grader, a CTO, and a future employer?

## When to invoke

- Slash command `/build-audit` (default: interactive, all lenses).
- `/build-audit --scope <lens>` — run only one lens. Lenses: `coverage`, `decisions`, `deferrals`, `differentiators`, `prd-critique`, `red-team`, `risks`, `communication`.
- End-of-week / before submission deadlines.
- After a major MR merge.
- When user asks: *"what did we ship?"*, *"is our scope honest?"*, *"what would a reviewer poke at?"*, *"what's wrong with our PRD?"*.

## Modes

| Mode | When | How |
|---|---|---|
| **Interactive** (default) | First audit, end-of-week | Walk lens by lens. Ask user before generating each section. Build doc collaboratively. |
| **One-shot scope** | Repeat use, focused review | `--scope <lens>` runs only one lens. Dump the section, user edits. |

## Discovery (always run first)

Before asking the user anything:

1. **Recent merges:** `git log --oneline --merges -20` and `git log --oneline -30`.
2. **Find the PRD:** glob for the first match of `${PRD_PATTERNS}`. Read the full document.
3. **Find architecture docs:** glob for `${ARCHITECTURE_PATTERNS}`. Skim section headings.
4. **Find decisions docs:** glob for `${DECISIONS_PATTERNS}`.
5. **Find handoffs / risks:** glob for `${HANDOFF_PATTERNS}` and `${RISKS_PATTERNS}`.
6. **Find slide decks / video scripts** via `${SLIDES_PATTERNS}`.
7. **Find test/eval artifacts:** look for `tests/`, `evals/`, `**/test_*.py`, eval result files.

For each pattern category that returns no matches, **note it as a discovery gap** (not a fatal error — projects vary). The user can supply paths manually.

Show the user a one-paragraph **"what I found"** summary before starting interactive lens-by-lens questioning. Include:
- PRD path + section count (or "no PRD found, will ask")
- Architecture doc path + key sections
- Decisions doc path (or note its absence — itself an audit finding)
- Handoff path + last update timestamp
- Recent merge count + branches involved
- A two-sentence read on what was built recently

## Interactive flow

For each lens (in order), ask:

> Run **<lens-name>** lens? (y / n / skip) — *brief description of what this lens does*

If yes: ask the lens-specific seed questions, generate the section using user input + discovered evidence, then ask:

> Anything to add or correct in this section?

Move to the next lens. Default order:

1. Coverage
2. Decisions
3. Deferrals
4. Differentiators
5. PRD critique
6. Red-team
7. Risks
8. Communication

User can re-order, skip, or run a single lens.

---

## Lens 1 — Coverage

**Purpose:** PRD requirements ↔ shipped artifacts.

**Seed questions:**
- Which PRD section to audit? (default: all)
- Cutoff date for "shipped"? (default: today)

**Output: a coverage matrix.**

| PRD line / requirement | Shipped? | Evidence (file:line / commit / MR) | Status note |
|---|---|---|---|
| §X.Y *"requirement text"* | ✅ / ⚠️ / ❌ / 🚧 | concrete reference | one-line context |

Status conventions:
- ✅ shipped (full)
- ⚠️ partial (some functionality, gap noted)
- ❌ deferred (documented somewhere)
- 🚧 in-progress (active work this session)
- ⛔ silent gap (not documented, not shipped) — **flag for action**

**Flag any PRD line where you can't find evidence** — ask the user before marking it deferred or silent-gap.

---

## Lens 2 — Decisions

**Purpose:** Catalog non-trivial choices and verify each has a documented rationale.

**Seed questions:**
- Time window for decisions to audit? (default: last 7 days)
- Any specific decision categories to focus on? (e.g. *"all architectural"*, *"all security-relevant"*, *"all scope cuts"*)

**Source material:**
- Recent commit bodies (especially conventional-commits with bodies)
- MR descriptions
- The decisions doc found in discovery
- Handoff *"Decisions Made"* sections
- Slide deck *"tradeoffs"* sections

**Output: decisions catalog.**

| Decision | Options considered | Choice | Rationale | Documented at | Confidence |
|---|---|---|---|---|---|
| short label | bullet list | the path taken | why | file:line / doc § | high / medium / low |

**Flag low-confidence decisions** — the rationale exists in conversation but not in any persistent doc. Action item: write it up.

---

## Lens 3 — Deferrals

**Purpose:** Compare what's labeled "deferred" in docs vs what's silently missing in code.

**Seed questions:**
- Any specific feature areas you suspect have silent deferrals?

**Two passes:**

**Pass 1 — Documented deferrals.** Walk handoffs, slide decks, README "what's gated for next phase" sections, MR descriptions. List each documented deferral with: where documented, reason, ETA / trigger to revisit.

**Pass 2 — Silent deferrals.** Walk PRD requirements; for each one, check if the code actually implements it. If not implemented and not in the documented-deferrals list → **silent deferral** (the highest-risk class — undermines submission credibility).

**Output:**

```
## Documented deferrals
| Feature | Why deferred | When revisit | Documented at |

## Silent deferrals (action items)
| Feature | PRD reference | Why silent | Suggested action |
```

---

## Lens 4 — Differentiators

**Purpose:** Identify what we shipped beyond the PRD that strengthens portfolio narrative. **This is the lens that often gets missed.**

**Seed questions:**
- Any features you remember shipping that the PRD didn't explicitly require?
- Anything you're proud of that's not in the slide deck?

**Source material:**
- Walk recent commits — find features without a clean PRD-line trace
- Look at a candidates file if one exists (`${STORIES_DIR}/_candidates.md` from the story skill, if installed)
- Look at module-internal helpers and constraints (UNIQUE constraints, idempotency layers, structural-only logging, defense-in-depth)

For each candidate differentiator, ask:
- Is this scope creep, or a deliberate add?
- If deliberate: what audience does it serve? (grader / CTO / employer / clinician / operator)
- What does it differentiate against? (typical implementation / framework default / competitor)
- One-line elevator pitch?

**Output:**

| Differentiator | Audience | Differentiates against | One-line pitch | Where to surface |
|---|---|---|---|---|
| feature name | who cares | the typical alternative | the pitch | cheat sheet / story / slide / README |

**Action items often surfaced here:**
- Differentiators absent from cheat sheet
- Differentiators absent from video script
- Differentiators absent from candidates file

The most common gap: a real differentiator shipped but never promoted, so the portfolio understates the work.

---

## Lens 5 — PRD critique

**Purpose:** Find PRD contradictions, ambiguities, things-that-don't-make-sense, and document how each was resolved.

**Mindset:** the spec is fallible; finding inconsistencies isn't disrespect, it's how you defend a build that interpreted a spec carefully.

**Seed questions:**
- Did any PRD section bite during the build? (force a rework, scope confusion, ambiguous requirement)
- Were there contradictions between sections you had to resolve?

**Source material:**
- The PRD itself (re-read with skeptical eye)
- Commits that mention re-reading PRD or scope ambiguity
- Handoff *"Decisions Made"* rows that reference PRD interpretation

**Walk the PRD section by section.** For each, check for:
- **Contradictions** — does §A say X while §B says not-X?
- **Ambiguities** — open to multiple readings; which did we pick?
- **Unstated assumptions** — relies on context the spec doesn't define
- **Outdated requirements** — feature is described but the underlying tech / API has changed
- **Inconsistent terminology** — same concept named two different ways
- **Missing acceptance criteria** — requirement stated but no measurable bar

**Output:**

| PRD reference | Issue type | What was unclear | How we resolved | Documented at |
|---|---|---|---|---|

**This output is gold for the interview** — when a reviewer asks *"why did you interpret §X this way?"*, you have a documented answer.

---

## Lens 6 — Red-team / hostile reviewer

**Purpose:** Anticipate the hardest questions a reviewer / employer will ask, and have prepared defenses.

**Mindset:** adversarial, generative — *"if I were trying to make this fail, where would I poke?"*. Different cognitive mode from the other lenses; lean into it.

**Seed questions:**
- Which audience is the next hostile review? (grader / employer / CTO / red-team teammate)
- Any specific concerns the user has? ("they'll probably ask about X")

**Walk the top 5–8 shipped capabilities.** For each, generate 2–3 hardest questions:

- *"Why did you choose X — isn't Y obviously better?"*
- *"How does this fail under condition Z?"*
- *"Where's the evidence this actually works at scale?"*
- *"What's the security / privacy / cost / latency surface?"*
- *"What would break first?"*
- *"How would you re-architect with full hindsight?"*

For each generated question, ask the user: *"do you have a prepared answer? (y / draft / no)"*. Capture the answer or flag it as gap.

**Output:**

| Capability | Hostile question | Prepared defense | Gap? |
|---|---|---|---|
| feature | the question | the answer | (empty if ready) / "needs facts" / "no defense yet" |

**Action items:**
- Questions with no prepared answer → write a story / draft a defense before the review
- Defenses that depend on a number you can't cite → measure or estimate before the review

---

## Lens 7 — Risks

**Purpose:** Verify each claimed risk mitigation has actual code evidence.

**Source material:** files matched by `${RISKS_PATTERNS}`, plus the architecture doc's risks section if present.

**Walk each documented risk + claimed mitigation.** For each:
- Find the code path that implements the mitigation
- Verify it's wired into the runtime path (not orphaned)
- Cite file:line

**Flag claimed-mitigated-but-actually-not gaps.** These are the highest-credibility-cost gaps.

**Output:**

| Risk | Claimed mitigation | Code evidence | Status |
|---|---|---|---|
| risk name | what doc says we do | file:line where it lives | ✅ wired / ⚠️ partial / ❌ orphaned |

---

## Lens 8 — Communication

**Purpose:** For each shipped capability + each deferral + each differentiator, ensure we have a sentence-length defense for three audiences.

**Source material:**
- The outputs of the other lenses
- Existing cheat sheet / video script / stories

**For top 5 shipped + top 3 deferrals + top 3 differentiators, draft:**

| Item | One sentence to a grader | One sentence to a CTO | One sentence to a future employer |
|---|---|---|---|
| feature | precision-focused | cost / safety-focused | judgment-focused |

**Flag any item without all three** — that's a communication gap. Source material for the story skill, the cheat sheet, the README.

---

## Output

Write to `${AUDITS_DIR}/<YYYY-MM-DD>-build-audit.md`. Create the dir if it doesn't exist; prompt the user once.

Top-of-file structure:
```
# Build Audit — <date>

**Scope:** <full audit | --scope <lens>>
**Discovery summary:** <one paragraph from discovery phase>

---

## 1. Coverage matrix
...
## 2. Decisions catalog
...
## 3. Deferrals ledger
...
## 4. Differentiators
...
## 5. PRD critique
...
## 6. Red-team
...
## 7. Risks audit
...
## 8. Communication-ready summary
...

---

## Action items
- [ ] item — context — suggested next step
- [ ] ...
```

If lenses were skipped, omit those sections (don't leave empty headers).

## Action items section is the most valuable output

The audit's worth is measured by what it surfaces for follow-up. Common categories:

- **Document a decision** — code reflects a choice that's not in the decisions doc
- **Document a deferral** — code skips a feature without documented reason
- **Promote a differentiator** — feature shipped but absent from cheat-sheet / video / portfolio
- **Verify a mitigation** — doc claims X is handled but code path can't be found
- **Update a status** — handoff says "deferred" but code now ships it
- **Resolve a PRD ambiguity** — re-reading surfaced a gap in our recorded interpretation
- **Prepare a red-team defense** — a likely-asked question with no current answer

Action items should be **specific and small** — *"add `Citation` rationale to DECISIONS.md §X (10 min)"* not *"document better"*.

## What NOT to do

- Don't grade code quality. Use the technical-review skills for that.
- Don't fabricate evidence. If you can't find a file:line for a claim, ask the user.
- Don't skip the differentiators lens. This is where portfolio narrative comes from.
- Don't skip the PRD-critique lens. The interview asks *"why did you interpret §X that way?"* often.
- Don't skip the red-team lens. The hardest questions are predictable; not preparing is a choice.
- Don't generate the audit doc without the user confirming each lens.
- Don't blend PRD-critique and red-team mindsets. Critique is methodical; red-team is adversarial. Blending dilutes both.

## Portability

Project-local skill, designed to copy. Two steps to use in another project:

1. Copy this file to the target project's `.claude/skills/`.
2. Edit the frontmatter `config:` block — `AUDITS_DIR` plus the discovery `*_PATTERNS` glob lists.

The body refers to paths by config-key name throughout. No project-specific assumptions in the body. Discovery patterns can be left at defaults — they're glob lists that match common conventions.
