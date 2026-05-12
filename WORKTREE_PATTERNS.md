# Worktree Patterns — When and How to Run Multi-Lead Work in Parallel

When you have 2+ named leads (Aria/Bram/Cleo or custom-named) producing code on different branches at the same time, you have three workable patterns. This doc captures the trade-offs and the setup recipes — including the junction discipline for `.project/` (or `.gauntlet/` in AgentForge-style projects) and `.claude/` and the OneDrive mirroring for session summaries.

---

## The three modes

### Mode 1 — Single checkout, no worktrees (default for short builds)

All leads work in the same physical folder (`<project-root>/`). Branches are created with `git switch -c crt/<short-desc>`; only one branch is checked out at a time.

**When to use:**
- 1-2 leads working sequentially (not literally in parallel)
- Short build (≤ 5 days)
- Coordination overhead of worktrees exceeds value

**Pros:**
- Zero setup
- No junction discipline needed
- `.project/` (or `.gauntlet/` in AgentForge-style projects) and `.claude/` are real directories — no broken symlinks
- Easy to switch leads in one terminal

**Cons:**
- Truly parallel work needs careful branch hygiene (no `git switch` mid-edit)
- Two open Cursor terminals on the same folder can race on the working tree
- Lost the ability to physically separate "what Aria is touching" from "what Bram is touching"

This is the mode the W3 ClinicalRedTeam template's `in-flight.md.template` ships with as default.

---

### Mode 2 — Per-lead worktrees with junctions (for sustained multi-lead parallelism)

Each lead gets a dedicated git worktree at a sibling path on disk (`../<project>-aria/`, `../<project>-bram/`, etc.). Each is on its own branch. Each opens Cursor in its own folder.

**When to use:**
- 2-3 leads producing code in parallel for sustained periods (multi-day)
- Need physical separation to prevent file-conflict surprises
- Multi-week build where the setup cost amortizes

**Pros:**
- Each lead's working tree is isolated from the others
- Two leads can `git status` independently without seeing each other's changes
- Cursor workspaces stay focused on one lead's surface
- A lead can have uncommitted work in their tree while others ship

**Cons:**
- Setup cost (worktree per lead + junctions per worktree)
- Junction discipline (or it breaks: see "Failure modes" below)
- `git worktree remove` mistakes can lose uncommitted work

This is the mode W2 used and the mode ClinicalRedTeam adopted mid-Phase-1b once Aria + Bram needed sustained parallel sessions.

---

### Mode 3 — Hybrid (Tate stays in main, leads get worktrees)

Tate (the Director) operates from the main checkout (or a `<project>-tate/` worktree on a `tate-director` branch). Each named sprint lead gets a worktree.

**When to use:**
- Most multi-lead projects (this is W2's actual pattern)
- Want Tate to push directly to `main` for docs/handoff regen without disturbing lead branches

**Pros:**
- Tate's coordination work doesn't compete with lead branches
- Each lead is fully isolated
- The "global" handoff at repo-root is owned by Tate's worktree (no junction needed for that file specifically)

**Cons:**
- One more worktree to set up
- Slightly higher cognitive load (4 physical folders instead of 3)

---

## Setup recipe — Mode 2 / Mode 3

### Step 1 — Create the worktree

From the **main checkout**:

```bash
# bash (Windows Git Bash, macOS, Linux)
cd /c/Dev/MyProjects/MyProject     # or wherever your main checkout lives
git worktree add ../MyProject-aria -b crt/aria-day1
git worktree add ../MyProject-bram -b crt/bram-day1
git worktree add ../MyProject-cleo -b crt/cleo-day1
git worktree add ../MyProject-tate -b crt/tate-director   # optional Mode 3
```

```powershell
# PowerShell equivalent
cd C:\Dev\MyProjects\MyProject
git worktree add ..\MyProject-aria -b crt/aria-day1
# ... etc
```

`git worktree list` should now show all of them.

### Step 2 — Junction the coordination directories

This is the critical step. Each worktree has its OWN `.project/` and `.claude/` after creation (well, actually it has nothing for `.project/` because that dir is gitignored — the worktree is empty there). You need each worktree's `.project/` and `.claude/` to point at the main checkout's copies, so all leads see the same handoffs/in-flight/persona files.

**Windows (Git Bash):**

```bash
cd ../MyProject-aria
rmdir .project 2>/dev/null              # remove if it exists as empty dir
rmdir .claude 2>/dev/null
cmd //c mklink /J .project ..\\MyProject\\.project
cmd //c mklink /J .claude  ..\\MyProject\\.claude
```

**Windows (PowerShell):**

```powershell
cd ..\MyProject-aria
if (Test-Path .project) { Remove-Item .project -Recurse -Force }
if (Test-Path .claude)  { Remove-Item .claude -Recurse -Force }
New-Item -ItemType Junction -Path .project -Target ..\MyProject\.project
New-Item -ItemType Junction -Path .claude  -Target ..\MyProject\.claude
```

**macOS / Linux / WSL:**

```bash
cd ../MyProject-aria
rm -rf .project .claude 2>/dev/null
ln -s ../MyProject/.project .project
ln -s ../MyProject/.claude  .claude
```

Repeat for `MyProject-bram`, `MyProject-cleo`, etc.

### Step 3 — Verify the junctions

```bash
ls -la .project    # should show -> ../MyProject/.project (symlink)
                    # or "Junction" attribute on Windows
cat .project/<slug>/in-flight.md | head -5    # should match main checkout's content
```

### Step 4 — Open Cursor in the lead's worktree

```bash
cursor ../MyProject-aria
```

Then in Cursor's terminal, start Claude Code and run `/aria` (or whichever lead).

### Step 5 — Tear down when the lead's workstream is done

From the **main checkout** (NOT the worktree):

```bash
# 1. Confirm the worktree is clean + branch is merged
cd /c/Dev/MyProjects/MyProject
git -C ../MyProject-aria status            # clean?
git -C ../MyProject-aria log --oneline -5  # what's there?
git fetch origin                            # latest origin
git branch --merged origin/main | grep crt/aria   # merged?

# 2. Remove the junctions FIRST (otherwise git worktree remove can be confused)
cd ../MyProject-aria
rm .project .claude    # Unix
# OR on Windows:
# cmd //c rmdir .project
# cmd //c rmdir .claude

# 3. Now remove the worktree
cd ../MyProject
git worktree remove ../MyProject-aria
git branch -d crt/aria-day1   # if merged
```

**W2 had `start_<lead>` / `finish_<lead>` shell launchers** that automated all of this. They're worth porting if you do this 3+ times. Until then, the manual recipe above is fine.

---

## Why junctions for `.project/` and `.claude/`

### `.project/` (or `.gauntlet/` in AgentForge-style projects) is gitignored

The coordination directory holds work-in-progress handoffs, kickoff prompts, candidates, session summaries — all stuff that's local to your team's workflow and shouldn't ship to the public mirror. So the directory is in `.gitignore`.

But that means **a fresh git worktree has no `.project/` directory at all** — git only creates tracked content in new worktrees. If Aria's worktree has no `.project/`, she can't read in-flight.md, can't read the kickoff, can't write her handoff. The whole coordination layer is gone.

The junction (Windows) / symlink (Unix) makes Aria's worktree's `.project/` a pointer to the main checkout's `.project/`. Single source of truth on disk; edits in any worktree are immediately visible from every other worktree; nothing leaks to public mirrors.

### `.claude/` is committed but synchronization matters

`.claude/` IS tracked in git (persona files, settings, skills). A fresh worktree DOES have `.claude/` with whatever was committed at branch-creation time.

But during the build, you might add a new persona file or update a skill in one worktree. Without a junction, those changes wouldn't appear in the other worktrees until you committed + pulled them in each worktree separately. Junction means the change is visible everywhere immediately.

(Caveat: if `.claude/` is junctioned, then any `git add` of a `.claude/*` file from a worktree will resolve to the main checkout's path. Be deliberate about this.)

---

## OneDrive mirroring (session summaries + critical handoffs)

For the most failure-resistant version of memory, mirror critical files to a cloud-synced personal folder (OneDrive, iCloud Drive, Dropbox, etc.) outside the repo. Reasons:

1. **Survives `rm -rf` accidents** — if you wipe the repo, sessions are still in OneDrive
2. **Survives git mistakes** — if a `git reset --hard` loses uncommitted work, OneDrive has the previous state
3. **Cross-project searchability** — search across all your projects' session summaries from one place
4. **Portfolio integration** — easy to reference past work in personal notes / blog drafts

### What to mirror

| File class | Mirror? | Why |
|---|---|---|
| `.project/<slug>/sessions/*.md` | YES | Long-form audit trail of substantive sessions |
| `.project/<slug>/handoffs/*.md` | OPTIONAL | Operational state; can be regenerated from git log + recent context. Mirror if you want survivor copy. |
| `.project/stories/*.md` | YES | Portfolio assets — interview-ready stories you'll reference for years |
| `CLAUDE_SESSION_HANDOFF.md` | OPTIONAL | Recreatable but useful to mirror for cross-machine continuity |
| `.project/<slug>/candidates/_candidates.md` | YES | Story seed material; lose this and you lose pre-promotion moments |
| `~/.claude/projects/<slug>/memory/MEMORY.md` | YES | Auto-loaded into every session; if lost, all permanent facts vanish |

### Setup recipe

The simplest pattern: drop a one-liner in a script that copies on demand.

**Bash / WSL:**

```bash
#!/usr/bin/env bash
# scripts/mirror-to-onedrive.sh
ONEDRIVE_ROOT="/c/Users/$USER/OneDrive/projects/<project-slug>"
mkdir -p "$ONEDRIVE_ROOT/sessions" "$ONEDRIVE_ROOT/stories" "$ONEDRIVE_ROOT/handoffs"
cp -u .project/<slug>/sessions/*.md "$ONEDRIVE_ROOT/sessions/"
cp -u .project/stories/*.md         "$ONEDRIVE_ROOT/stories/"
cp -u .project/<slug>/handoffs/*.md "$ONEDRIVE_ROOT/handoffs/"
cp -u CLAUDE_SESSION_HANDOFF.md     "$ONEDRIVE_ROOT/"
```

Run after each substantive session, or on a cron / Task Scheduler.

**More automated:** add a hook in `.claude/settings.json` that runs this script after `/session-handoff`.

---

## Failure modes

### Junction broken / pointing at wrong path

**Symptom:** `cat .project/<slug>/in-flight.md` returns "No such file or directory" even though the file exists in the main checkout.

**Cause:** The junction was removed, the main checkout was moved/renamed, or the worktree was set up without re-running the junction step.

**Fix:** From the worktree, `rm .project` (removes the broken junction, not the target), then re-run the junction setup from "Step 2" above.

### Worktree's `.gitignore` shows `.project/` as untracked

**Symptom:** `git status` in the worktree shows `.project/` as an untracked directory.

**Cause:** The junction wasn't created — the worktree has a real empty `.project/` (or one populated by a buggy script).

**Fix:** Verify with `ls -la .project` — should show the symlink/junction marker. If not, run "Step 2" again.

### Two leads simultaneously edit `in-flight.md`

**Symptom:** Last-writer wins; one lead's status update vanishes.

**Cause:** Concurrent edits to a shared file via the junction. Junctions don't prevent this.

**Mitigation:** `in-flight.md` is meant for SMALL update-on-change edits (lock a row when starting work, release it when done). Don't do bulk edits. The `/daily-sync` skill's lead-attestation pattern explicitly addresses this — each lead writes ONLY its own handoff, never the shared in-flight.

### Worktree removed without removing junctions first

**Symptom:** `git worktree remove ../MyProject-aria` warns about "uncommitted changes" or "incomplete removal."

**Cause:** The junctions in the worktree look like real directories to git, and git tries to inspect them.

**Fix:** Always `rm .project .claude` (junctions only, not targets) BEFORE `git worktree remove`. The W2 launcher's `finish_<lead>` script verifies junctions exist + are removed before invoking worktree removal — this is one reason it's worth scripting if you do this often.

### "Lost" file because it was edited in a worktree that's been removed

**Symptom:** A file you remember editing is gone from main.

**Cause:** Edited in a worktree (committed locally), worktree was removed before the branch was merged or pushed.

**Mitigation:** Always push the branch (`git push origin crt/lead-day1`) before removing the worktree. The launcher's `finish_<lead>` script verifies "all local commits pushed to upstream" as a precondition.

---

## Decision tree

```
Are 2+ leads doing sustained parallel code work (multi-day)?
├── NO → Mode 1 (single checkout). Move on.
└── YES
    ├── Is the build < 5 days?
    │   └── PROBABLY YES → Mode 1 with branch hygiene; only escalate if Mode 1 is biting.
    └── Is the build ≥ 5 days OR will leads run > 2 days in parallel?
        ├── YES → Mode 2 or 3 (worktrees)
        │   ├── Does Tate need to push directly to main for handoff regen frequently?
        │   │   └── YES → Mode 3 (Tate gets a worktree too, on a director branch)
        │   └── NO → Mode 2 (Tate stays in main, leads get worktrees)
        └── Are the leads physically remote / different machines?
            └── YES → Mode 1 + git pushes; worktrees don't help across machines
```

---

## What this template ships with

The template's `.project/PROJECT/in-flight.md.template` defaults to **Mode 1 (single checkout)** because most projects start there and only escalate if needed.

If you upgrade to Mode 2 or 3 mid-project (which ClinicalRedTeam did during Phase 1b), follow the recipe above and update `in-flight.md` to reflect the new mode + per-lead worktree paths.

The `/use-template` skill could prompt for worktree mode at init time. v0.1.x doesn't (Mode 1 default + this doc as the upgrade path); a future v0.2.0 could add the prompt.

---

## TLDR

- **Default to Mode 1** (single checkout) for short builds and small teams.
- **Escalate to Mode 2/3** (per-lead worktrees) when sustained parallel work over multiple days makes single-checkout coordination friction-heavy.
- **Junction `.project/` and `.claude/`** in every worktree — they're the shared coordination layer and need to be visible everywhere.
- **Mirror session summaries + stories + memory to OneDrive** (or equivalent) — survives repo accidents and gives cross-project searchability.
- **Always push branches before removing worktrees** — uncommitted/unpushed work is the most common loss vector.
- **Use `start_<lead>` / `finish_<lead>` shell scripts** if you do this 3+ times — they automate the setup + teardown discipline. Worth ~30 min one-time investment.
