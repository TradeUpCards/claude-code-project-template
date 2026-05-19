---
name: use-template
description: Interactive walkthrough to finish setting up a new project after cloning from claude-code-project-template. Asks the user about project name, license, gitignore flavor, lead-setup option (A/B/C), deadlines, and target URL. Then fills placeholders, renames PROJECT/ to project slug, removes unused option files, makes the first commit, and optionally creates a GitHub/GitLab repo. Run this FIRST after cloning the template. Distinct from the user-level `project-init` skill (general-purpose project bootstrap) and `agent-team-setup` (multi-agent workflow installer for ANY existing project).
---

You are running the project initialization walkthrough for a new Claude Code project built from `claude-code-project-template`.

# Mission

Turn the placeholder-filled template into a fully customized, ready-to-use project. End state: user can run `/tate` and start their first day of work.

# Hard rules

**Do NOT skip user confirmation between phases.** This skill makes irreversible file changes (renames, deletions, first git commit). Confirm explicitly at each major step.

**`.project/` and `.claude/` are NEVER committed to git.** They hold local-only coordination state — kickoffs, handoffs, agent personas, slash-command skills, PRD source docs, lesson sketches, demo plans — which lives in OneDrive/iCloud sync, not in the cohort-visible repo. Do NOT ask the user whether to commit them. Do NOT remove the `/.project/` or `/.claude/` entries from the chosen gitignore. If you find them tracked after the first commit, that's a bug — fix it before phase 4.

**Project docs go under `.project/<slug>/docs/`, not at repo root.** The template ships an empty `docs/{prd,lesson-design,research,demo}/` scaffold under `.project/PROJECT/`. After the rename in Phase 3.4 it becomes `.project/<slug>/docs/`. The user drops PRD PDFs and lesson sketches there. Code-level docs (README.md, ARCHITECTURE.md) still live at repo root.

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

All three options already exclude `/.project/` AND `/.claude/` by default — do NOT prompt the user about this and do NOT edit those lines out. If the user asks "but won't that break the slash commands across machines?" the answer is: yes, they break across machines via git, but they sync via OneDrive (see Phase 3.9.7). Git is for the public/cohort repo; OneDrive is for personal coordination state.

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
Rename `.project/PROJECT/` to `.project/<PROJECT_SLUG>/`. This brings the empty `docs/{prd,lesson-design,research,demo}/` scaffold along. After rename, point out to the user that PRD PDFs go in `.project/<slug>/docs/prd/` (they typically have them in `~/Downloads/` and the skill can `cp` them over if the user names the files).

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

### 3.9.7 OneDrive mirror — move .project/ + .claude/ out of working tree (BEFORE first commit)

Ask:
**Header:** `OneDrive mirror`
**Question:** `Move .project/ and .claude/ into OneDrive (or iCloud/Dropbox) now, with junctions back to the working tree? Strongly recommended — survives 'rm -rf' of the working tree, syncs across machines, and means the coordination state lives outside the cohort-visible repo by physical placement, not just by .gitignore.`
**Options:**
- Yes, set up OneDrive mirror now (Recommended) — runs `scripts/setup-onedrive-mirror.sh` (or `.ps1` on Windows)
- Skip — leave .project/ and .claude/ in the working tree (still gitignored, still local-only)

If "Yes":
1. Confirm OneDrive sync path with the user. Default for Cory's machines is `$HOME/OneDrive/Documents/GauntletAI/<PROJECT_NAME>/`. The script accepts `ONEDRIVE_ROOT=...` override.
2. Run the script: `bash scripts/setup-onedrive-mirror.sh` (or `powershell -File scripts/setup-onedrive-mirror.ps1`).
3. Verify: `ls -la .project .claude` — should show them as symlinks/junctions pointing into OneDrive.
4. Verify the OneDrive target has the actual content: `ls -la $ONEDRIVE_ROOT/<PROJECT_NAME>/.project`.

Why this happens **before** the first commit:
- Avoids any chance of staging `.project/` or `.claude/` content even momentarily
- Once junctioned, git operations on the working tree still respect .gitignore (junctions look like dirs to git, which then sees the dotfiles via the gitignore rule)
- If anything goes wrong, the working tree is still the source of truth — you haven't committed yet

Warn the user:
- "Don't `rm -rf .project/` from the working tree afterwards — follows the junction, nukes OneDrive."
- "First OneDrive upload of the small text files takes a few minutes; subsequent syncs are instant."
- "On a new machine: install OneDrive, sync, then re-run this script on a fresh clone — it'll create junctions pointing at the already-synced files."

### 3.10 First commit

Verify `.project/` and `.claude/` are NOT staged before committing:

```bash
git status --short | grep -E "^(A|\?\?).*(\.project|\.claude)/"  # MUST be empty
```

If anything matches, the gitignore is wrong — fix it before continuing. Then commit:

```bash
git add .
git commit -m "chore: init <PROJECT_NAME> from claude-code-project-template

Set up Option <X> coordination infra for the <PROJECT_NAME> project.
Coordination state (.project/, .claude/) is local-only / OneDrive-mirrored;
not committed to this repo.

Generated by /use-template from claude-code-project-template."
```

Confirm zero `.project/` or `.claude/` files in the commit:

```bash
git show --stat HEAD | grep -E "(\.project|\.claude)/" || echo "✓ clean: no coordination state in commit"
```

### 3.11 (Optional) Create GitHub repo
**Header:** `Create GitHub repo?`
**Options:** `Yes, public`, `Yes, private`, `No, I'll do it later`

If yes:
```bash
gh repo create <PROJECT_SLUG> --<public|private> --source=. --push
```

### 3.12 (Optional) Create GitLab repo

**Pre-flight check — detect existing cohort-named repos.** GauntletAI cohort GitLab often auto-creates a repo like `gauntlet-week-N-<slug>` before the user starts. Check before creating a new one:

```bash
glab repo list --mine 2>&1 | grep -iE "(${PROJECT_SLUG}|gauntlet.*${PROJECT_SLUG}|gauntlet-week)" || echo "(no matching repos)"
```

If a cohort-named repo exists, ask the user via AskUserQuestion:
- **Use existing** — keep the cohort-named repo as `origin`, skip creating a new one. Best for staying consistent with how the cohort tracks projects.
- **Create new clean-named + delete cohort one** — `glab repo delete <cohort-repo>`, then `glab repo create <PROJECT_SLUG>`. Cleaner name; you lose the cohort's auto-naming convention.
- **Create new clean-named + keep cohort one as backup** — wastes a slot but you don't lose anything yet.

**Header:** `Create GitLab repo?`
**Question:** `If you use GitLab as your primary remote, create one?`
**Options:** `Yes, private (Recommended for cohort work)`, `Yes, internal (Gauntlet visibility)`, `Yes, public`, `No`

For Gauntlet cohort projects, prefer `--internal` (visible to cohort members + staff). Use `--private` for solo / pre-demo state. `--public` is for portfolio-ready post-demo state.

If yes:
```bash
glab repo create <PROJECT_SLUG> --<private|internal|public> --description "<PROJECT_DESCRIPTION>"
# Note: does NOT auto-add a remote; we configure dual-push manually below
```

**Use HTTPS URLs, not SSH, for cohort GitLab.** The `glab` CLI auth uses an API token (HTTPS); SSH keys are a separate setup step that often isn't done. HTTPS pushes work with the cached glab token. The earlier `glab auth status` output saying "Git operations protocol: ssh" is just a default for what `glab clone` uses — it doesn't mean SSH keys are configured.

If both GitHub + GitLab, set up dual-push:
```bash
# Determine GitLab namespace from glab (typically the user's username)
GITLAB_NAMESPACE=$(glab auth status 2>&1 | grep -oP "as \K\w+" | head -1)
GITLAB_HOST=$(glab auth status 2>&1 | grep -oP "labs\.\w+\.\w+|gitlab\.\w+" | head -1)
GITLAB_URL="https://${GITLAB_HOST}/${GITLAB_NAMESPACE}/<PROJECT_SLUG>.git"
GITHUB_URL="https://github.com/<GITHUB_NAMESPACE>/<PROJECT_SLUG>.git"

# Fetch from GitLab; push to BOTH on every `git push`
git remote add origin "${GITLAB_URL}"
git remote set-url --add --push origin "${GITLAB_URL}"
git remote set-url --add --push origin "${GITHUB_URL}"
git remote -v   # verify: 1 fetch URL + 2 push URLs
```

### 3.12.1 GitLab protected-main force-push (only if needed)

If the user later needs to force-push to GitLab `main` (e.g., to rewrite history that accidentally included `.project/`), the default protection blocks it. Workaround using glab API:

```bash
# Allow force push temporarily
glab api "projects/${GITLAB_NAMESPACE}%2F<PROJECT_SLUG>/protected_branches/main" \
  -X PATCH -f allow_force_push=true

git push --force origin main

# Lock back down
glab api "projects/${GITLAB_NAMESPACE}%2F<PROJECT_SLUG>/protected_branches/main" \
  -X PATCH -f allow_force_push=false
```

Document this recipe in the final report (Phase 4) under "Useful recipes" so the user has it when needed.

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

Files in git (visible to graders / collaborators / public if you make the repo public):
- README.md (project README, edit as you build)
- LICENSE (<chosen>)
- .gitignore (<flavor>; excludes /.project/ and /.claude/)
- CLAUDE_SESSION_HANDOFF.md (fill in as you do the first session)
- scripts/setup-worktrees.sh (Mode 2 only)
- scripts/setup-onedrive-mirror.sh (if user opted in)

Files local-only / OneDrive-mirrored (NOT in git, never pushed):
- .project/<slug>/ — coordination scaffold (in-flight, kickoffs, candidates, handoffs, sessions, stories)
- .project/<slug>/docs/{prd,lesson-design,research,demo}/ — PRD source, sketches, demo materials
- .claude/agents/ (<count> personas active)
- .claude/skills/ (<list active slash commands>)

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
- **Classifier blocks `rm -rf` on `LICENSE_OPTIONS/` and `GITIGNORE_OPTIONS/`** — the deny-list `Bash(rm -rf *)` is intentional safety; don't try to bypass. Instead, print to user verbatim and ask them to run in their own terminal:
  ```bash
  rm -rf LICENSE_OPTIONS GITIGNORE_OPTIONS
  ```
  Or on Windows PowerShell:
  ```powershell
  Remove-Item -Recurse -Force LICENSE_OPTIONS, GITIGNORE_OPTIONS
  ```
- **Classifier blocks `git update-ref -d HEAD` or `git push --force`** — happens during cleanup if `.project/` was accidentally committed. Don't try to work around the block. Print the exact recovery sequence and ask the user to paste it in their terminal:
  ```bash
  git update-ref -d HEAD
  git add -A
  git commit -m "chore: init <PROJECT_NAME> (cleaned)"
  git push --force origin main
  ```
- **GitLab `main` is protected, force-push rejected** — print the `glab api allow_force_push` workaround from Phase 3.12.1 verbatim and have the user run it.
- **Initial commit accidentally included `.project/` or `.claude/`** — this is the most common setup bug. The Phase 3.10 verification step (`git status --short | grep ...`) catches it. If it's already been pushed: cleanup is force-push-rewrite per Phase 3.12.1 + the recovery sequence above. Update the project's memory (if `/use-template` has access to it) with a feedback note: this user wants coordination state never in git.
