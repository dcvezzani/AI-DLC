# AIDLC — <slug>

| Field | Value |
|-------|-------|
| **Work item** | [#N](https://github.com/<owner>/<repo>/issues/N) |
| **Product Spec** | [product-spec.md](./product-spec.md) |
| **Branch** | `feature/<slug>` |
| **Worktree** | `/absolute/path/to/<repo>-worktrees/<slug>` |
| **PR target** | `main` (or parent integration branch) |
| **App port** | *(filled by `git-worktree-port-registry`)* |
| **API port** | *(filled by `git-worktree-port-registry`)* |
| **Debug port** | *(filled by `git-worktree-port-registry`)* |
| **Dev env file** | `.aidlc/dev.env` |

## Phase commands

| Phase | Status | Command |
|-------|--------|---------|
| Plan | | `/plan <slug>` |
| Design | | `/design <slug>` |
| Build | | `/build <slug>` — runs **`git-worktree-port-registry`** before local dev/tests |
| Review | | `/review <slug>` |
| Ship | | `/ship <slug>` |
| Learn | | `/learn <slug>` — runs **`git-worktree-cleanup`** after merged PR |

## Ports registry

Parallel worktrees share **`../<repo>-worktrees/aidlc-ports.sqlite`**. Ports are allocated per slug by **`git-worktree-port-registry`** ([skills/git-worktree-port-registry/SKILL.md](../../skills/git-worktree-port-registry/SKILL.md)). Load **`.aidlc/dev.env`** before `npm run dev` or integration tests.

## File ownership

List primary paths this feature owns (minimize overlap with parallel worktrees):

- `path/to/file`

## Notes

Optional: delivery wave, rebase order, open PR link.
