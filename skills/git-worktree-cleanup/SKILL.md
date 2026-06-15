---
name: git-worktree-cleanup
description: >-
  Remove a feature git worktree and prune its local branch by feature slug.
  Use after a merged PR, during /learn, or when the user asks to clean up a
  worktree for a slug.
type: skill
aidlc_phases: [validate]
tags: [git, worktree, cleanup, learn, hygiene]
requires: []
author: Melissa Benua
created_at: 2026-06-15
updated_at: 2026-06-15
---

# Git worktree cleanup (by slug)

Remove the **git worktree** and **local feature branch** associated with a **feature slug**. Run from the **main repo checkout** — not from inside the worktree being removed.

**Typical callers:** **`/learn <slug>`** (after Validate PASS), or direct invocation when the user names a slug after merge.

**Related:** [git-workflow](../git-workflow/SKILL.md) § Worktree lifecycle (record/create conventions), [CONSUMER-SETUP.md](../../docs/CONSUMER-SETUP.md) § Git worktrees.

## When to use

- Feature PR is **merged** and the worktree is no longer needed
- **`/learn`** step 5 — git hygiene for the specified slug
- User asks: “remove the worktree for `<slug>`”, “clean up worktree”, post-merge housekeeping
- Periodic cleanup when `git worktree list` shows stale feature checkouts

## Inputs

Resolve **slug** from `$ARGUMENTS` or ask the user. Scope cleanup to **one slug** per run (epic children need separate invocations).

## Discovery (path + branch)

Stop at the **first** match:

| # | Source | What to parse |
|---|--------|----------------|
| 1 | `feature/<slug>/AIDLC.md` | `**Worktree**`, `**Branch**` table rows |
| 2 | `feature/*/sub-features/<slug>/AIDLC.md` | same (child units) |
| 3 | `AGENTS.md` → **Git worktrees (AIDLC)** | path pattern + branch pattern for slug |
| 4 | `git worktree list` | line whose path or bracketed branch contains the slug |

If **none** found: report `N/A — no worktree recorded for <slug>` and **exit successfully** (not an error). When called from `/learn`, record that in `learn-notes.md` → **Git hygiene**.

### AGENTS.md path pattern fallback

When only a **path pattern** is documented (e.g. `../<repo>-worktrees/<slug>`), expand `<slug>` and resolve relative to repo root (`git rev-parse --show-toplevel`).

## Preconditions (before remove)

All required unless the user **explicitly** requests cleanup without merge (e.g. abandoned spike — confirm first):

1. **Merged PR** — `gh pr list --state merged --head <branch>` or user confirms merge
2. **Not current workspace** — `git rev-parse --show-toplevel` must **not** equal the worktree path; if it does, ask the human to open the main checkout first
3. **On main checkout** — `git switch main` (or team default integration branch) before `git worktree remove`

When invoked from **`/learn`**, also require Validate scorecard **PASS** for the slug.

## Workflow

Run from main repo root:

```bash
REPO_ROOT="$(git rev-parse --show-toplevel)"
cd "$REPO_ROOT"

# 1. Discover <path> and <branch> (see Discovery above)
git worktree list

# 2. Safety preview
test "$(git rev-parse --show-toplevel)" = "<path>" && echo "BLOCKED: currently inside worktree" && exit 1

git -C "<path>" status --short   # if dirty, stop and ask human before --force

# 3. Prune remote-tracking refs
git fetch --prune

# 4. Remove worktree
git worktree remove "<path>"
# git worktree remove --force "<path>"   # only if dirty AND human approved

# 5. Prune local branch when remote is gone
git branch -vv | awk '/: gone]/{print $1}' | while read -r b; do
  case "$b" in main|master) ;; *) git branch -d "$b" ;; esac
done
# Or target discovered branch explicitly:
# git branch -d "<branch>"   # use -D only with human confirmation if -d fails
```

Report a short summary:

- Slug
- Worktree path removed (or `N/A`)
- Branch pruned (or skipped + reason)
- Merged PR link (if verified via `gh`)

## Safety rules

- Never `git worktree remove --force` without **human approval** when the worktree has uncommitted changes
- Never delete `main` or `master`
- `git branch -D` only when `-d` fails **and** human confirms squash-merge or intentional discard
- Do **not** delete the **remote** branch on GitHub — that stays merge UI / `gh` policy
- Do **not** remove worktrees for **other slugs** in the same run

## What this does not do

- Create worktrees (`git worktree add`) — see [git-workflow](../git-workflow/SKILL.md) § Create at Build
- Archive `feature/<slug>/` to `00-shipped/` — Learn documentation step
- Close GitHub issues — tracker step in `/learn`

## Learn integration

When called from **`/learn`**, append to `feature/<slug>/learn-notes.md`:

```markdown
## Git hygiene

| Item | Result |
|------|--------|
| Worktree | `<path>` removed / N/A |
| Branch | `<branch>` pruned / skipped — reason |
| PR | <merged PR URL> |
```
