---
name: story
description: Capture a moment from the build as a structured interview-ready story (design defense, STAR, or war story). Invoke when user types `/story`, says "save this as a story" / "let's capture that" / "good answer — write it up", or selects a candidate from the candidates file. Outputs to `${STORIES_DIR}/<slug>.md`. Enforces a 5-property quality bar — refuses to write if any are missing.
config:
  # Override per project. The body derives all sibling files from STORIES_DIR
  # using fixed filenames (skill-internal convention, not configurable).
  STORIES_DIR: .gauntlet/stories
---

# File-naming conventions inside `${STORIES_DIR}` (don't change these — they're internal to the skill)

- Candidates backlog: `_candidates.md`
- Templates: `_template-design-defense.md`, `_template-star.md`
- Index: `README.md`
- Stories: `<slug>.md` (no `_` prefix — that's reserved for templates and the candidates file)

# Story-writing skill

You are converting a moment from the conversation (or a candidate row) into a structured, interview-ready story file.

## Configuration

Path defaults are in the frontmatter `config:` block. To use this skill in another project: copy this file, edit `config:`, done. The skill body refers to paths by their config-key name (e.g. `${STORIES_DIR}`).

If `${STORIES_DIR}` doesn't exist in the project, prompt the user once before creating it.

## When to invoke

- User types `/story` or `/story <candidate-slug>`.
- User says: *"save this as a story"*, *"let's capture that"*, *"write it up"*, *"good answer — capture it"*.
- During session-handoff, **prompt** the user about unrealized candidates (do not auto-invoke).

## Inputs

Three possible sources:

1. **Recent conversation turn** — user pointed at content in this session.
2. **Candidate row** in `${STORIES_DIR}/_candidates.md` — invoke as `/story <slug>` or pick interactively.
3. **Cold start** — user wants to write a story but hasn't pointed at material; ask one question to anchor the moment.

## Workflow

### Step 1 — Identify the source

If the user just said *"save this as a story"* in response to a structured answer you produced, that's the source. Otherwise ask:

> Which moment are we capturing? Paste a quote, point at a candidate row in `${STORIES_DIR}/_candidates.md`, or describe the situation in one line.

### Step 2 — Pick the shape

Three shapes available:

| Shape | Use when... | Template |
|---|---|---|
| **design-defense** | Defending a technical choice ("why X over Y?") | `${STORIES_DIR}/_template-design-defense.md` |
| **STAR** | Behavioral, scope decision, response to feedback | `${STORIES_DIR}/_template-star.md` |
| **war-story** | Bug diagnosed and fixed with a teaching moment (uses STAR template) | `${STORIES_DIR}/_template-star.md` |

Ask the user if it isn't obvious from the source.

### Step 3 — Walk the structured questions

**First (both shapes): pick a category.** Ask: *"Category? `architecture` / `security` / `scope` / `incident` / `meta` — pick the one whose audience-fit you most want this story to surface for."* If the user is unsure, suggest based on the source — see the taxonomy in `${STORIES_DIR}/_candidates.md`.

#### For design-defense

Ask in this order, capturing answers as you go:

1. The question an interviewer would ask (verbatim or paraphrased).
2. The hook line — one sentence that lands the position.
3. N reasons (3–5), ordered by interview impact. For each: a concrete number, a file:line if applicable, a citation from the spec/PRD if applicable.
4. Trade-offs to own (2–4). Each is a real weakness with the reason it's accepted.
5. Failure modes hit while building (if any) — symptom, root cause, fix, file:line.
6. The 30-second spoken version.
7. Two or three follow-up questions you're ready for, with prepared answers.

#### For STAR / war-story

1. The prompt this answers (paraphrased behavioral question).
2. The hook line.
3. **Situation:** when, stakes, constraints (concrete dates and numbers).
4. **Task:** what *I* specifically owned (not what the team did).
5. **Action:** 3–6 specific steps with file/decision references. If it's a war story, include the diagnostic chain (symptom → log → hypothesis → confirmation → fix).
6. **Result:** measurable outcomes (time vs estimate, cost, tests passing, eval rate, lasting artifact).
7. What I'd do differently (1–3 honest reflections — *"nothing"* is a tell).
8. The 60-second spoken version.
9. Follow-ups ready.

### Step 4 — Enforce the quality bar

A story is not ready to write unless **all five** are present:

1. **Concrete numbers** (cost, latency, line count, percentage, time).
2. **File:line references** for any code-level claim.
3. **A trade-off owned** — no story claims zero downside.
4. **A spoken version under 60 seconds** when read at normal pace.
5. **First-person artifact** — *"I caught this when..."* not third-person.

If any is missing, ask the user for the data. **Do not fabricate** numbers or citations to fill the gap.

### Step 5 — Write the file

- Path: `${STORIES_DIR}/<slug>.md` where `<slug>` is kebab-case derived from the title.
- Do NOT prefix the filename with `_` (that prefix is reserved for templates and the candidates file).
- Use the appropriate template structure verbatim, filled with this story's content.
- Include a `**Category:** <category>` line near the top of the file (just after the title, matching the template).
- Match the prose style of existing stories in `${STORIES_DIR}` — no emojis, no AI-tells (*"the hard problem isn't X, it's Y"*, em-dashes everywhere, *"robust"*, *"leverage"*).

### Step 6 — Update the candidates file

If the story came from a candidate row in `${STORIES_DIR}/_candidates.md`:

- Mark the row `[WRITTEN]` and add the story file path, OR
- Delete the row outright (user's preference)

Default: ask the user once, remember the answer for the session.

### Step 7 — Update the README index

Add a one-line entry to `${STORIES_DIR}/README.md` under the appropriate category (Design defenses / War stories / etc.). If unsure which category, ask.

### Step 8 — Confirm and stop

Show the user the file path written + the candidates / README updates. Stop. Do not chain into another story.

## What NOT to do

- Don't write more than one story per invocation. Depth over breadth.
- Don't fabricate a number to fill the quality bar. Ask.
- Don't include emojis.
- Don't restate the templates verbatim — fill them with this story's content.
- Don't write if the source is genuinely thin. Stash as a candidate row instead and tell the user.
- Don't insert AI-tells: *"hard problem isn't X, it's Y"*, *"robust"*, *"leverage"*, *"navigate the challenges of"*, em-dashes scattered through prose, listicles without connecting prose.

## Portability

Project-local skill, designed to copy. Two steps to use in another project:

1. Copy this file to the target project's `.claude/skills/`.
2. Edit the frontmatter `config.STORIES_DIR` to point at the target project's stories folder.

The body derives every other path from `STORIES_DIR` using fixed filenames (skill-internal convention). One config key is the only thing to change per project.
