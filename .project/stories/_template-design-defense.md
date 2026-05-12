# [TITLE — what the design choice is]

**Category:** `architecture` | `security` | `scope` | `incident` | `meta` (pick one — see `_candidates.md` taxonomy)

> **Use when:** "Why did you choose X over Y?" / "Walk me through your decision to..."
>
> Copy this file, rename it, fill in each section. Delete this header block.

---

## The question

> [Verbatim or paraphrased — what an interviewer might ask. Helps your future self pattern-match the right story to the right prompt.]

---

## Hook line (memorize this — say it first)

> [One sentence. The headline answer. Lands the position before the reasoning.]

---

## N reasons, ordered by interview impact

### 1. [Strongest reason — usually cost or a hard requirement]

[2–4 sentences. Include a concrete number where possible — dollars, milliseconds, percentages. Cite a file:line if it's a code-level claim.]

| Approach | Concrete number |
|---|---|
| The path you chose | $X |
| The path you didn't | $Y |

### 2. [Second strongest reason — usually a safety or correctness property]

[Same pattern. Reference the spec / PRD section if there's an external requirement that pushed the decision.]

### 3. [Third reason — often observability or maintainability]

[Same pattern.]

### 4. [Optional fourth — only include if it's actually distinct, not a restatement]

### 5. [Optional fifth — last reason, often a smaller win that becomes a closer]

---

## Trade-offs I'd own

[2–4 honest weaknesses. The interviewer probably knows them already, and admitting them first is stronger than getting caught.]

- **[Weakness 1]** — [why I accept it / when I'd revisit]
- **[Weakness 2]** — [same]
- **[Weakness 3]** — [same]

---

## Failure modes I hit (if any)

[Brief — 2–3 bullets max. Real bugs / friction the design caused, with how I caught + fixed each. Demonstrates the design has been stress-tested in practice, not just on paper.]

- **[Failure 1]** — [symptom → root cause → fix → file:line of fix]
- **[Failure 2]** — [same shape]

---

## What I'd say in the interview (~30 sec)

> [Spoken version. First-person. Lead with the hook, then the top 2 reasons, then the strongest trade-off owned. Cut to under 60 seconds when read aloud at normal pace. Practice it.]

---

## Follow-up questions I'm ready for

- **"What if X were different?"** → [your answer]
- **"How would you re-architect this?"** → [your answer]
- **"What's the production gap?"** → [your answer]
