---
name: build
description: AIDLC Build + Test orchestrator (TDD). Delivers an open PR with green CI per Tech Spec; re-enters to triage /review PR comments — fix or reply+resolve. Not for spec-only work.
type: skill
aidlc_phases: [build, test]
tags: [aidlc, orchestrator, build, test, tdd]
requires: [git-workflow]
author: Melissa Benua
created_at: 2026-04-12
updated_at: 2026-06-15
---

# /build — Build + Test (phase orchestrator)

You are the **phase orchestrator** for AIDLC **Build** and **Test** as **one practice**: tests are written **with** the code (TDD), not in a separate follow-up stage. Canonical definitions:

- **AIDLC:** `docs/AIDLC.md` at the repository root — Build phase, Test phase, V-model.

**Library skills:** [docs/SKILLS.md](../../docs/SKILLS.md). Resolve bundles from your install or `.claude/skills/<bundle>/` in the workspace.

## Build authorization

**Invoking `/build <slug>` explicitly authorizes** everything required to finish Build for that feature:

- create commits on the feature branch
- push to `origin`
- open (or update) the pull request
- iterate until **CI is green**

Do **not** stop at “branch only”, “uncommitted locally”, or “say if you want a PR”. Do **not** ask the human to commit or open the PR unless a **hard blocker** prevents it (auth failure, missing `gh`, branch protection you cannot satisfy). Consumer repos may add a repo-local **`git-commit.sh`** or commit template — use those when present.

## Inputs

- Approved `feature/<slug>/tech-spec.md`
- **If re-entering after `/review`:** open **PR** with **AIDLC Review — …** comments (see `/review` orchestrator).
- **If code already exists but no PR:** treat as **PR completion only** — skip re-implementation; run the [Build completion checklist](#build-completion-checklist-mandatory) from step 3 onward.

## Orchestration — initial implementation

1. **Branch:** use a descriptive branch (e.g. `feature/<slug>`). Apply **`git-workflow`** ([skills/git-workflow/SKILL.md](../git-workflow/SKILL.md)). Read `feature/<slug>/AIDLC.md` or consumer **`AGENTS.md`** for integration base branch (e.g. `main` vs parent epic branch).
2. **Worktree ports (when parallel):** run **`git-worktree-port-registry`** ([skills/git-worktree-port-registry/SKILL.md](../git-worktree-port-registry/SKILL.md)) for the slug before local dev or integration tests. Load **`.aidlc/dev.env`** (consumer `dev:aidlc` or `set -a && . ./.aidlc/dev.env`). Skip when single-checkout repo with no worktree.
3. **Implement by Tech Spec section:** in PR/commits, reference which section you are implementing (AIDLC Build guidance).
4. **TDD:** for each unit of work, prefer **test first or alongside** — frontend (`npm test` / vitest as applicable), backend (`dotnet test`, etc.). Load **`testing`** ([skills/testing/SKILL.md](../testing/SKILL.md)); use **`frontend-web`** for UI, **`backend-saas`** for API layers.
5. **Do not** “finish code” and add tests only at the end unless the Tech Spec explicitly sequenced an exception.
6. **Run the [Build completion checklist](#build-completion-checklist-mandatory)** — Build is **not complete** until every step passes.
7. **Local checks** before push; treat **remote CI** as authoritative for handoff to Test/Review.

## Build completion checklist (mandatory)

Execute **in order**. Do not declare Build complete, update `feature/README.md` to “Build complete”, or set the GitHub issue phase to Build/review-ready until step 6 succeeds.

| Step | Action | Done when |
|------|--------|-----------|
| 1 | Local **build + tests** pass | Required project commands green; load `.aidlc/dev.env` when in a worktree |
| 2 | **Commit** all implementation + spec/doc updates on the feature branch | Working tree clean; use consumer `git-commit.sh` if the repo has one |
| 3 | **Push** branch | `git push -u origin HEAD` (or push from the feature worktree) |
| 4 | **Open PR** if none exists | `gh pr create` or GitHub MCP — see [Opening the PR](#opening-the-pr) |
| 5 | **Verify CI** on latest commit | Required checks **passing** on the PR (`gh pr checks` or GitHub UI) |
| 6 | **Return PR URL** in your summary | Human and `/review` can find the handoff surface |

If step 5 fails, fix and push; repeat from step 1 locally, then 3–5. **Do not** hand off to `/review` with a failing or missing PR.

### Opening the PR

Before `gh pr create`, gather context in parallel:

```bash
git status
git diff <base>...HEAD          # base = main, develop, or parent integration branch
git log <base>..HEAD --oneline
gh pr list --head "$(git branch --show-current)"
```

If a PR already exists for the branch, **update** it (push new commits; edit body if scope changed) — do not open a duplicate.

**Create** (adapt title/body to the feature; link issue and spec blobs):

```bash
git push -u origin HEAD   # if not already on origin

gh pr create --base <integration-branch> --head "$(git branch --show-current)" \
  --title "#<issue> <short feature title>" \
  --body "$(cat <<'EOF'
## Summary
- <1–3 bullets: user-facing outcome>

## Spec
- Tech Spec: <blob link on branch>
- Product Spec: <blob link>

Closes #<issue>

## Test plan
- [x] <commands run locally>
- [ ] CI green on this PR
EOF
)"
```

Prefer **`gh`** CLI. Use GitHub MCP only when `gh` is unavailable. On auth errors, report the blocker and the exact command the human can run — do **not** treat “ask the user” as a successful Build exit.

### Definition of done (Build)

Build **succeeds** only when **all** are true:

- Tech Spec implemented with tests
- **Open PR** exists (URL returned)
- **CI green** on the latest commit
- Issue/README metadata may say “Build complete → `/review`” **only after** the PR URL is known

**Branch only**, **uncommitted changes**, or **local tests only** = Build **incomplete**.

## Review feedback loop (after `/review` has posted on the PR)

When **`/review`** has run, each **dimension** (Tech Spec, Testing, DevOps, Frontend/UX, Security) should have left **GitHub PR comments** (preferred). The **build** orchestrator **owns the response**:

1. **Read** all open **review threads** on the PR — especially comments titled `AIDLC Review — …`.
2. **For each finding** (or each thread), decide:
   - **Valid:** implement the fix (code/tests/config/docs as appropriate), push commits, and **reply** on the same thread briefly stating what changed **or** mark the conversation **resolved** once the fix is on the branch (per team habit).
   - **Invalid / won’t fix (with cause):** **reply** on the **same GitHub comment thread** with a **clear rationale** (cite Tech Spec section, intentional scope, or false positive). Then **resolve the conversation** so reviewers see closure.
3. **Do not** silently ignore review feedback — every thread gets either a **code change** or a **documented reply**.
4. Re-run **local build/tests**; ensure **CI** is green.
5. If changes were substantive, run **`/review`** again for a **follow-up pass**; otherwise proceed toward merge per team rules.

**Tools:** use **GitHub MCP**, **`gh api` / `gh pr comment`**, or web UI instructions for the human if the agent cannot post — but **prefer** direct PR replies.

## Consumer specialist dispatch (optional)

Default routing uses horizontal library skills below. Consumer repos may add **`.cursor/skills/build/SKILL.md`** or an **`AGENTS.md` dispatch table** that routes Tech Spec sections to vertical specialists (backend, frontend, infra, testing) in parallel when files do not overlap.

See [docs/CONSUMER-SETUP.md](../../docs/CONSUMER-SETUP.md). Generic **`/build`** contract (TDD, open PR, green CI) still applies.

## Nested library skills (typical)

| When | Skill |
|------|--------|
| Implementation patterns | `frontend-web`, `backend-saas`, `architecture` |
| Tests | `testing` |
| Commits / branches / PR | `git-workflow` |
| Manual UI check before PR (optional) | [INTERACTIVE-UI-VALIDATION.md](../../docs/INTERACTIVE-UI-VALIDATION.md) |

## Outputs

- **Initial Build pass:** **PR URL** + **green CI** + code/tests implementing the Tech Spec; traceability to Tech Spec sections in commits/PR body.
- **After `/review`:** same PR with **addressed or replied-to** review threads, CI still green.

## Do not

- Mark Build complete when work exists only on a local branch or in uncommitted files
- End with “say if you want a commit/PR” after `/build` — that **is** the request
- Run `/review` or update issue phase to review-ready without an open PR
- Open a second PR for the same branch
