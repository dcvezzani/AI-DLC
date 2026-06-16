---
name: git-workflow
description: Git workflow standards including commit messages, branch management, worktree lifecycle, and success criteria. Use when committing code, creating branches, preparing changes for review, or cleaning up worktrees in Learn.
type: skill
aidlc_phases: [build, test, review, validate]
tags: [git, version-control, workflow, commits, branches, worktree]
requires: []
author: Melissa Benua
created_at: 2026-03-07
updated_at: 2026-06-15
---

# Git Workflow Standards

## When to Use

- When creating commits
- When deciding whether to create a branch
- Before declaring work complete
- When preparing changes for pull requests
- During **`/learn`** — remove feature git worktrees after merged PR (§ Worktree lifecycle)

## Branch Protection

1. **Never commit directly to `main`** - create a feature branch first
2. **Always pull from main before branching** - run `git fetch origin && git rebase origin/main` (or `git pull origin main`) before creating a feature branch or starting work. This prevents merge conflicts in the PR.
3. **Branch naming**: Use kebab-case that describes the change (e.g., `add-user-authentication`, `fix-memory-leak`)
4. **Stay on feature branches** - don't create feature branches off feature branches

## Success Criteria

### Always (any commit or handoff)

Changes are considered successful when ALL of the following are met:

1. **Requirements satisfied** - original task goals are achieved
2. **Build passes** - code compiles without errors
3. **Tests pass** - all existing and new tests succeed
4. **No linting errors** - code follows project style guidelines
5. **Files formatted** - code is properly formatted
6. **Complete changes** - no partial or incomplete modifications
7. **Dependencies included** - all necessary imports and packages added
8. **Documentation updated** - relevant docs reflect the changes

### Build phase (`/build`) — additional bar

Per [skills/build/SKILL.md](../build/SKILL.md) and consumer `docs/AIDLC.md` Phase 3:

9. **Committed and pushed** on a feature branch (not only local or uncommitted)
10. **Open pull request** exists targeting the integration branch (`main`, `develop`, or parent epic branch per repo)
11. **CI green** — required status checks passing on the PR’s latest commit

**Branch only** or **local tests only** does **not** satisfy Build. Invoking `/build` authorizes commit, push, and PR creation for that feature.

### Language-Specific Checks

| Language | Build Command | Test Command |
|----------|---------------|--------------|
| JavaScript/TypeScript | `npm run build` | `npm test` |
| Python | N/A | `pytest` or `python -m unittest` |
| Go | `go build` | `go test ./...` |
| Rust | `cargo build` | `cargo test` |
| C# | `dotnet build` | `dotnet test` |

## Commit Message Format

```
<type>(<scope>): <brief description>

<detailed description>

Cursor-Task: <original task description>
```

### Types

| Type | When to Use |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Code restructuring without behavior change |
| `style` | Formatting, whitespace changes |
| `docs` | Documentation only |
| `test` | Adding or modifying tests |
| `infra` | Infrastructure, CI/CD, deployment |
| `chore` | Maintenance tasks |

### Guidelines

- **Brief description**: Max 72 characters, imperative mood ("add" not "added")
- **Scope**: Optional, indicates area affected (e.g., `auth`, `api`, `ui`)
- **Detailed description**: 1-2 sentences explaining the why
- **Cursor-Task**: Reference the original task that prompted the change

## Examples

### Good Commit

```bash
git commit -m "feat(auth): implement OAuth2 user authentication

Add Google OAuth2 authentication flow with JWT token handling.
Update user model to support OAuth providers.

Cursor-Task: Add OAuth2 authentication for user login"
```

### Good Infrastructure Commit

```bash
git commit -m "infra(aws): update ECS task definitions

Increase memory allocation and add CloudWatch logging.

Cursor-Task: Optimize ECS resource allocation"
```

### Bad Commits

- `git commit -m "fixed stuff"` - Missing type, not descriptive
- `git commit -m "wip"` - Never commit work-in-progress
- Committing with failing tests - Always verify success criteria first

## Tools

- Prefer git command line for operations
- **Pull requests:** use `gh pr create` / `gh pr view` / `gh pr checks` during **`/build`** (see [skills/build/SKILL.md](../build/SKILL.md) § Build completion checklist)
- GitHub MCP is an alternative when `gh` is unavailable
- If both fail (auth, network), report the blocker and the exact `gh pr create` command — do **not** treat “ask the user to open a PR” as Build complete

## Pre-Commit Checklist

Before every commit, verify:

- [ ] **Pulled from main first** — `git fetch origin && git rebase origin/main` before branching
- [ ] Build compiles successfully
- [ ] All tests pass
- [ ] No linting errors
- [ ] Changes match the original task
- [ ] No secrets or credentials in the commit
- [ ] Documentation updated if needed

## Worktree lifecycle (Learn)

Optional pattern for **parallel Build** on child units. Single-checkout repos skip this section.

### Record at Plan or Design

Create `feature/<slug>/AIDLC.md` (copy from [feature/_template/AIDLC.md](../../feature/_template/AIDLC.md)) with **Worktree** (absolute path) and **Branch** rows. Consumer convention: [CONSUMER-SETUP.md](../../docs/CONSUMER-SETUP.md) § Git worktrees. **`/plan`** creates the worktree when the consumer documents the pattern ([skills/plan/SKILL.md](../plan/SKILL.md) § Consumer worktree pre-flight).

### Create at Build (optional)

When Plan did **not** already create the worktree (single-checkout repo or no `AGENTS.md` worktree block), from the main repo checkout:

```bash
git worktree add <path> -b <branch> <base-branch>
```

See [skills/build/SKILL.md](../build/SKILL.md) for PR requirements. Worktree creation is **not** required for every feature.

### Allocate ports at Build

Run **`git-worktree-port-registry`** ([skills/git-worktree-port-registry/SKILL.md](../git-worktree-port-registry/SKILL.md)) for the slug when using parallel worktrees — before local dev, integration tests, or UI validation. Writes `aidlc-ports.sqlite`, updates `feature/<slug>/AIDLC.md`, and **`.aidlc/dev.env`**.

### Discover path and branch

See **`git-worktree-cleanup`** ([skills/git-worktree-cleanup/SKILL.md](../git-worktree-cleanup/SKILL.md) § Discovery).

### Remove after merged PR

Run **`git-worktree-cleanup`** for the slug — typically from **`/learn`**, or standalone after merge. Full procedure, preconditions, and safety rules: [skills/git-worktree-cleanup/SKILL.md](../git-worktree-cleanup/SKILL.md).
