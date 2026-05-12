---
name: use-template
description: Interactive walkthrough to finish setting up a new project after cloning from claude-code-project-template. Asks the user about project name, license, gitignore flavor, lead-setup option (A/B/C), deadlines, and target URL. Then fills placeholders, renames PROJECT/ to project slug, removes unused option files, makes the first commit, and optionally creates a GitHub/GitLab repo. Run this FIRST after cloning the template. Distinct from the user-level `project-init` skill (general-purpose project bootstrap) and `agent-team-setup` (multi-agent workflow installer for ANY existing project).
---

You are running the project initialization walkthrough for a new Claude Code project built from `claude-code-project-template`.

# Mission

Turn the placeholder-filled template into a fully customized, ready-to-use project. End state: user can run `/tate` and start their first day of work.

# Hard rule

**Do NOT skip user confirmation between phases.** This skill makes irreversible file changes (renames, deletions, first git commit). Confirm explicitly at each major step.

---

## Phase 1 — Gather inputs (ask ONE question at a time, wait for answer)

Use the `AskUserQuestion` tool for each. Suggested questions and headers:

### 1. Project name
**Header:** `Project name`
**Question:** `What's the name of this project? (Used for branding, CLAUDE_SESSION_HANDOFF, in-flight.md, kickoff files)`
**Free text** (no options) — use a single AskUserQuestion with one option that says "I'll type it" and then capture the user's text from their response.

Derive `{{PROJECT_SLUG}}` from PROJECT_NAME by lowercasing + replacing spaces/underscores with dashes (e.g., "Clinical Red Team" → "clinical-red-team").

### 2. Project description
**Header:** `Description`
**Question:** `Describe the project in 1-3 sentences (used in README + kickoff files)`
**Free text.**

### 3. Default branch
**Header:** `Default branch`
**Options:** `main` (recommended), `master`, `develop`

### 4. License
**Header:** `License`
**Options:**
- Apache 2.0 (recommended; permissive + patent grant)
- MIT (permissive minimal)
- Proprietary / All rights reserved (private project)

### 5. Gitignore flavor
**Header:** `.gitignore`
**Options:**
- Python (Recommended if Python primary)
- Node / JavaScript
- Generic (only IDE + OS noise)

### 6. Lead setup
**Header:** `Lead setup`
**Question:** `How will you coordinate work? (Option A / B / C — see TEMPLATE_GUIDE.md if unsure)`
**Options:**
- Option A — Solo Tate (no agent infra; for short solo builds)
- Option B — Generic teammate types (for short builds with occasional dispatches)
- Option C — Named leads Aria/Bram/Cleo (Recommended for builds 5+ days with parallel work)
- Option C+ — Named leads with custom names (instead of Aria/Bram/Cleo)

### 6.5. Worktree mode

**Header:** `Worktree mode`
**Question:** `When 2+ leads work in parallel, do you want each lead in their own git worktree (Mode 2), or all leads sharing the main checkout (Mode 1)? See WORKTREE_PATTERNS.md for the full decision tree.`
**Options:**
- Mode 1 — Single checkout (Recommended for short builds)
- Mode 2 — Per-lead worktrees with .project/ + .claude/ junctions (For sustained multi-day parallel work)
- Decide later — set up Mode 1 now; can escalate to Mode 2 mid-build if needed

If user picks Mode 2, also ask:
**Header:** `Worktree paths`
**Question:** `What sibling-path naming convention? Defaults to '<project>-<lead>' (e.g., MyProject-aria, MyProject-bram, MyProject-cleo).`
**Options:**
- `<project>-<lead>` (Recommended; matches W2/W3 convention)
- Custom pattern (provide via Other)

### 7. Hard deadlines
**Header:** `MVP deadline`
**Question:** `What's the MVP deadline (or "none")?`
**Free text** (e.g., `Tuesday 2026-05-13 11:59 PM`)

If MVP given:
**Header:** `Final deadline`
**Question:** `What's the Final deadline (or "same as MVP")?`
**Free text.**

### 8. Target URL (optional)
**Header:** `Target URL`
**Question:** `If this project integrates with or attacks a specific external system, what's its URL? Otherwise enter "N/A".`
**Free text.**

### 9. (Option C / C+ only) Lead workstreams
For each lead, ask:
**Header:** `<Name>'s workstream`
**Question:** `Describe <Name>'s workstream in one phrase (e.g., "Implementation: agent code + persistence + CLI")`
**Free text.**

**Header:** `<Name>'s owned files`
**Question:** `What files does <Name> own? (Glob patterns OK; e.g., "src/agents/**, tests/agents/**")`
**Free text.**

(For Option C+ also ask the lead names first.)

---

## Phase 2 — Confirm before changes

Restate ALL captured inputs as a summary table. Ask:
**Header:** `Confirm setup`
**Options:** `Confirm and proceed`, `Edit some answers`

If "Edit", re-ask the relevant questions.

---

## Phase 3 — Apply changes (do these in order)

### 3.1 License
Copy `LICENSE_OPTIONS/<chosen>.txt` to `./LICENSE`. For Apache 2.0, substitute the copyright line with `Copyright <CURRENT_YEAR> <USER_NAME_FROM_GIT_CONFIG>`.

### 3.2 Gitignore
Copy `GITIGNORE_OPTIONS/<chosen>.gitignore` to `./.gitignore`. Overwrites the placeholder.

### 3.3 Substitute placeholders in template files

Find every `.template` file. For each:
1. Read content
2. Substitute placeholders:
   - `{{PROJECT_NAME}}` → captured PROJECT_NAME
   - `{{PROJECT_SLUG}}` → derived slug
   - `{{PROJECT_DESCRIPTION}}` → captured description
   - `{{DEFAULT_BRANCH}}` → captured branch
   - `{{MVP_DEADLINE}}` → captured deadline (or `(no MVP deadline set)`)
   - `{{FINAL_DEADLINE}}` → captured deadline (or `(no Final deadline set)`)
   - `{{TARGET_URL}}` → captured URL (or `N/A`)
   - `{{ARIA_WORKSTREAM}}`, `{{ARIA_OWNED_FILES}}`, etc. (Option C only)
   - `{{LEAD_NAME_1}}`, etc. (Option C+ only — replace Aria/Bram/Cleo with custom names too)
3. Write back to the file's path WITHOUT the `.template` extension
4. Delete the original `.template` file

### 3.4 Rename PROJECT/ directory
Rename `.project/PROJECT/` to `.project/<PROJECT_SLUG>/`.

### 3.5 (Option A) Remove unused infra
If Option A:
- Delete `.claude/agents/{aria,bram,cleo}.md`
- Delete `.claude/skills/{aria,bram,cleo}/`
- Delete `.project/<slug>/kickoff/{aria,bram,cleo}.md`
- Delete `.project/<slug>/in-flight.md`
- Delete `.project/<slug>/handoffs/`, `coordination/`
- Delete `.claude/agents/implementation-lead.md`, `quality-lead.md`, `delivery-lead.md`, `observability-security-teammate.md`, `codebase-mapper.md` (Option A doesn't use generic teammates either)

### 3.6 (Option B) Remove only Option-C-specific infra
If Option B:
- Delete `.claude/agents/{aria,bram,cleo}.md`
- Delete `.claude/skills/{aria,bram,cleo}/`
- Delete `.project/<slug>/kickoff/{aria,bram,cleo}.md`
- Keep `.project/<slug>/in-flight.md`, `handoffs/`, `coordination/` (still useful for ad-hoc dispatches)
- Keep generic teammate types in `.claude/agents/`

### 3.7 (Option C / C+) Keep everything; ensure custom names applied
If Option C+:
- Rename `.claude/agents/aria.md` → `.claude/agents/<custom_name_1>.md` (and update content's `name:` field)
- Same for bram, cleo
- Same for `.claude/skills/aria/` etc.
- Same for `.project/<slug>/kickoff/aria.md` etc.
- Update gauntlet-team-lead.md's "Available teammates" section to reference custom names

### 3.8 Update CLAUDE_SESSION_HANDOFF.md
Substitute its placeholders. Remove the "[REPLACE WITH ACTUAL CONTENT]" markers and leave clean section headers ready for the user's first session-handoff to fill in.

### 3.9 Remove template-only files
- Delete `LICENSE_OPTIONS/` directory
- Delete `GITIGNORE_OPTIONS/` directory
- Delete `TEMPLATE_GUIDE.md` (template's own docs; new project doesn't need them — but ASK first; some users want to keep them as reference)
- Delete `.claude/skills/init-project/` (its job is done — but ASK first; user might want to re-run for a sub-project)

### 3.9.5 (Mode 2 only) Generate worktree-setup script

If the user picked Mode 2 in question 6.5, generate `scripts/setup-worktrees.sh` with the per-lead worktree creation + junction setup commands populated for the chosen lead names. Include both Windows (mklink) and Unix (symlink) paths in commented branches.

Script template:
```bash
#!/usr/bin/env bash
# Sets up per-lead git worktrees with .project/ + .claude/ junctions.
# Run AFTER /use-template has finished initial setup, from the main checkout.
# Idempotent: safe to re-run; skips worktrees / junctions that already exist.

set -euo pipefail

PROJECT_NAME="<PROJECT_NAME>"
PROJECT_DIR="$(pwd)"
LEADS=(aria bram cleo)  # adjust to actual lead names from /use-template

for lead in "${LEADS[@]}"; do
  WORKTREE_PATH="../${PROJECT_NAME}-${lead}"
  BRANCH="crt/${lead}-init"

  if [[ -d "${WORKTREE_PATH}" ]]; then
    echo "Worktree ${WORKTREE_PATH} already exists; skipping creation"
  else
    echo "Creating worktree at ${WORKTREE_PATH} on branch ${BRANCH}"
    git worktree add "${WORKTREE_PATH}" -b "${BRANCH}"
  fi

  cd "${WORKTREE_PATH}"

  # Junction .project/
  if [[ ! -L .project ]] && [[ ! -d .project ]]; then
    if [[ "$OS" == "Windows_NT" ]]; then
      cmd //c "mklink /J .project ..\\${PROJECT_NAME}\\.project"
    else
      ln -s "../${PROJECT_NAME}/.project" .project
    fi
    echo "  ✓ junctioned .project/"
  fi

  # Junction .claude/
  if [[ ! -L .claude ]] && [[ "$(ls -A .claude 2>/dev/null | wc -l)" -eq 0 ]]; then
    rm -rf .claude
    if [[ "$OS" == "Windows_NT" ]]; then
      cmd //c "mklink /J .claude ..\\${PROJECT_NAME}\\.claude"
    else
      ln -s "../${PROJECT_NAME}/.claude" .claude
    fi
    echo "  ✓ junctioned .claude/"
  fi

  cd "${PROJECT_DIR}"
done

echo ""
echo "Done. Each lead now has its own worktree:"
git worktree list
echo ""
echo "Open Cursor in each worktree and run /aria, /bram, /cleo respectively."
```

Print after generation:
```
✓ scripts/setup-worktrees.sh generated
✓ Run it now? (y/n) — or run later when you actually need parallel leads
```

If user picked Mode 1 or "Decide later," skip this step (Mode 2 setup is documented in WORKTREE_PATTERNS.md as the upgrade path).

### 3.10 First commit
```bash
git add .
git commit -m "chore: init <PROJECT_NAME> from claude-code-project-template

Set up Option <X> coordination infra for the <PROJECT_NAME> project.

Generated by /init-project from claude-code-project-template v0.1.0."
```

### 3.11 (Optional) Create GitHub repo
**Header:** `Create GitHub repo?`
**Options:** `Yes, public`, `Yes, private`, `No, I'll do it later`

If yes:
```bash
gh repo create <PROJECT_SLUG> --<public|private> --source=. --push
```

### 3.12 (Optional) Create GitLab repo
**Header:** `Create GitLab repo?`
**Question:** `If you use GitLab as your primary remote (e.g., AgentForge GauntletAI workflow), create one?`
**Options:** `Yes, public`, `Yes, private`, `No`

If yes, walk through `glab repo create` (or instruct user to create on GitLab UI; capture URL).

If both GitHub + GitLab, set up dual-push remote per the AgentForge pattern:
```bash
git remote set-url origin <GITLAB_URL>
git remote set-url --add --push origin <GITLAB_URL>
git remote set-url --add --push origin <GITHUB_URL>
git remote -v  # verify dual-push
```

---

## Phase 4 — Final report

Print a summary:

```
✓ Project initialized: <PROJECT_NAME>
✓ License: <license>
✓ Branch: <branch>
✓ Lead setup: Option <X>
✓ MVP deadline: <deadline>
✓ Final deadline: <deadline>
✓ Target URL: <url>
✓ First commit made: <SHA>
✓ Remote(s): <gitlab/github/none>

Files now in your repo:
- README.md (project README, edit as you build)
- LICENSE (<chosen>)
- .gitignore (<flavor>)
- .claude/agents/ (<count> personas active)
- .claude/skills/ (<list active slash commands>)
- .project/<slug>/ (coordination scaffolding)
- CLAUDE_SESSION_HANDOFF.md (fill in as you do the first session)

NEXT STEPS:
1. Run /tate to boot the team lead and get your morning report
2. Tate will tell you the first concrete action for Day 0
3. Before /clear at end of session: run /session-handoff to refresh CLAUDE_SESSION_HANDOFF.md

Optional follow-ups:
- Add .pre-commit-config.yaml + .gitlab-ci.yml + pyproject.toml when you're ready (not in this template's v0.1)
- If this is a security/AI project: consider adding RESPONSIBLE_USE.md (template can be copied from ClinicalRedTeam)
- If you have a PRD PDF: extract requirements into an interactive HTML checklist (manual for now; Tier 3 skill planned)
```

---

## Failure modes to handle

- **User runs `/init-project` twice** — detect on first run by checking if `.template` files still exist. If they don't, refuse with "It looks like /init-project has already run for this project. To reinitialize, you'd need to revert via `git reset --hard` first."
- **`gh` or `glab` not installed** — skip remote creation; tell user to do it manually + provide the commands.
- **User git config missing** — abort with "Set `git config --global user.name` and `user.email` first, then re-run."
- **User wants Option C+ but the custom names collide with reserved words** — refuse names like `tate`, `gauntlet-team-lead`, or any existing teammate-type name.
