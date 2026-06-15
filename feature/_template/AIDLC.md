# AIDLC — <slug>

| Field | Value |
|-------|-------|
| **Work item** | [#N](https://github.com/<owner>/<repo>/issues/N) |
| **Product Spec** | [product-spec.md](./product-spec.md) |
| **Branch** | `feature/<slug>` |
| **Worktree** | `/absolute/path/to/<repo>-worktrees/<slug>` |
| **PR target** | `main` (or parent integration branch) |

## Phase commands

| Phase | Status | Command |
|-------|--------|---------|
| Plan | | `/plan <slug>` |
| Design | | `/design <slug>` |
| Build | | `/build <slug>` |
| Review | | `/review <slug>` |
| Ship | | `/ship <slug>` |
| Learn | | `/learn <slug>` — runs **`git-worktree-cleanup`** after merged PR |

## File ownership

List primary paths this feature owns (minimize overlap with parallel worktrees):

- `path/to/file`

## Notes

Optional: delivery wave, rebase order, open PR link.
