# Consumer repo setup (AIDLC)

How to **vendor** this library in an application repo and wire **tracker-agnostic** AIDLC without copying Vega-specific automation.

**Related:** [ISSUE-TRACKER-PORTABILITY.md](ISSUE-TRACKER-PORTABILITY.md), [GITHUB-ISSUE-SPEC-LINKS.md](GITHUB-ISSUE-SPEC-LINKS.md), [INTERACTIVE-UI-VALIDATION.md](INTERACTIVE-UI-VALIDATION.md), [templates/AIDLC.md](templates/AIDLC.md).

---

## Layout

```
.claude/deps/ai-dlc/          # git submodule (pin a release tag)
.claude/skills → deps/ai-dlc/skills
.cursor/skills/<name>/       # optional repo-specific overrides
docs/AIDLC.md                # copy from docs/templates/AIDLC.md
AGENTS.md                      # Issue tracker + UI validation environments
.cursor/mcp.json               # from docs/templates/mcp.json.example
```

### Submodule

```bash
git submodule add https://github.com/dcvezzani/AI-DLC.git .claude/deps/ai-dlc
cd .claude && ln -s deps/ai-dlc/skills skills
```

Pin to a **release tag** (e.g. `v1.0.0`), not floating `main` — see [INSTALL.md](INSTALL.md).

### Cloud Agent install

`.cursor/environment.json`:

```json
{
  "install": "git submodule update --init --recursive && <your-dependency-install>"
}
```

---

## Two skill roots

| Path | Role |
|------|------|
| `.claude/skills` (symlink → submodule) | Generic AIDLC orchestrators and library skills |
| `.cursor/skills` | **Optional overrides** — consumer specialists, env-specific Ship/Review deltas |

**Override rule:** When both exist, **consumer `.cursor/skills` wins** for the same skill name. Overrides must state what they replace and link to the generic skill.

**Exception:** UI validation procedure is **not** overridden for tool choice — use [INTERACTIVE-UI-VALIDATION.md](INTERACTIVE-UI-VALIDATION.md) (Chrome DevTools MCP). Consumers only customize URLs and credentials below.

---

## Consumer specialist dispatch (Build example)

Default **`/build`** uses horizontal skills (`frontend-web`, `backend-saas`, `testing`). Larger repos may add **`.cursor/skills/build/SKILL.md`** that routes to vertical specialists (backend, frontend, infra) per Tech Spec sections — see [skills/build/SKILL.md](../skills/build/SKILL.md) § Consumer specialist dispatch.

Do **not** copy entire agent libraries into the submodule; keep specialists in the app repo’s `.cursor/agents/` or equivalent.

---

## `AGENTS.md` blocks to add

### Issue tracker (AIDLC)

Full template: [ISSUE-TRACKER-PORTABILITY.md](ISSUE-TRACKER-PORTABILITY.md).

For **GitHub Issues**, Product Spec links in issue bodies must use **`blob/<branch>/…`** URLs — see [GITHUB-ISSUE-SPEC-LINKS.md](GITHUB-ISSUE-SPEC-LINKS.md). Add a **Notes** row in `AGENTS.md` for your default blob branch (`main` or an integration branch).

Dual-tracker pattern (common): **Linear** (or Jira) = product backlog; **GitHub Projects v2** = optional headless phase transport — link issues, do not duplicate scope.

### Git worktrees (AIDLC)

Optional — use when **parallel Build** runs on child units in separate checkouts. Single-checkout repos can omit this block; `/learn` will record `N/A` for worktree cleanup.

**Record at Plan or Design:** copy [feature/_template/AIDLC.md](../feature/_template/AIDLC.md) to `feature/<slug>/AIDLC.md` with absolute **Worktree** path and **Branch** name.

**Create at Build (optional):** `git worktree add <path> -b <branch> <base>` — see [skills/build/SKILL.md](../skills/build/SKILL.md).

**Allocate ports at Build:** run **`git-worktree-port-registry`** ([skills/git-worktree-port-registry/SKILL.md](../skills/git-worktree-port-registry/SKILL.md)) for the slug — writes shared SQLite registry, updates `AIDLC.md` port rows, and **`.aidlc/dev.env`** in the worktree.

**Remove at Learn:** `/learn` runs **`git-worktree-cleanup`** ([skills/git-worktree-cleanup/SKILL.md](../skills/git-worktree-cleanup/SKILL.md)) for the slug after the feature PR is merged (worktree, branch, and port registry rows).

```markdown
## Git worktrees (AIDLC)

| Field | Value |
|--------|--------|
| **Parent directory** | `../<repo>-worktrees/` (sibling of repo root) |
| **Path pattern** | `../<repo>-worktrees/<slug>` |
| **Branch pattern** | `feature/<slug>` |
| **Port registry** | `../<repo>-worktrees/aidlc-ports.sqlite` |
| **Port base** | `18000` (optional; default in skill) |
| **Ports per slug** | `10` (optional block stride) |
| **Learn cleanup** | `/learn` or **`git-worktree-cleanup`** — [skills/git-worktree-cleanup/SKILL.md](../skills/git-worktree-cleanup/SKILL.md) |

### Port role mapping

| Role | Consumer env var | Used by |
|------|------------------|---------|
| app | `PORT` (or `VITE_PORT`) | `npm run dev`, UI validation |
| api | `API_PORT` | backend / integration tests |
| debug | `DEBUG_PORT` | Node inspector, dotnet debug |

**Dev env file:** `.aidlc/dev.env` in the worktree root (gitignored). Load before `npm run dev`, `npm test`, or integration tests:

```bash
set -a && [ -f .aidlc/dev.env ] && . ./.aidlc/dev.env; set +a
```

Add a **`dev:aidlc`** script (or equivalent) that sources `.aidlc/dev.env` then starts the dev server. Integration tests must read `AIDLC_APP_URL` / `AIDLC_APP_PORT` — not a hardcoded `localhost:8080`.
```

#### Consumer checklist (parallel worktrees)

1. Add **Git worktrees** and **Port role mapping** blocks to `AGENTS.md`.
2. Ensure the dev server honors `PORT` / stack-specific vars (no hardcoded local port).
3. Add `dev:aidlc` (or equivalent) that sources `.aidlc/dev.env`.
4. Wire test runner setup to load `.aidlc/dev.env` when present (Vitest/Jest `globalSetup`, Playwright `baseURL`, etc.).
5. Add `.aidlc/` to `.gitignore`.
6. Document integration test commands in Tech Spec or `AGENTS.md`.

#### CI vs local worktrees

| Context | Port source |
|---------|-------------|
| Feature worktree (parallel local) | `.aidlc/dev.env` + `aidlc-ports.sqlite` |
| Main checkout, single feature | `AGENTS.md` default **Local dev URL** |
| GitHub Actions CI | Workflow `env:` — not the local SQLite registry |

### UI validation environments

```markdown
## UI validation environments

Procedure: [INTERACTIVE-UI-VALIDATION.md](INTERACTIVE-UI-VALIDATION.md) (or vendored copy under `docs/`).
Tool: **Chrome DevTools MCP** (`chrome-devtools`) in `.cursor/mcp.json` — see AI-DLC `docs/templates/mcp.json.example`.

| Field | Value |
|--------|--------|
| **Local dev URL** | e.g. `http://127.0.0.1:8080` (fallback when not using worktree ports) |
| **Local dev port role** | `app` (optional — when set, build URL from `feature/<slug>/AIDLC.md` or `.aidlc/dev.env`) |
| **Staging / pre-prod URL** | e.g. `https://app.staging.example.com` |
| **Test credential env vars** | e.g. `STAGING_TEST_EMAIL`, `STAGING_TEST_PASSWORD` (values in Cloud Agent env only) |
| **Login flow notes** | Optional |
```

---

## Terminology

| Term | Meaning |
|------|---------|
| **Validate phase** | AIDLC phase 6; `/ship`; scorecard vs Product Spec |
| **UI validation** | Chrome DevTools MCP technique; used in Review and inside Validate — **not** a separate phase |

---

## GitHub automation (optional)

See **[GITHUB-AIDLC-QUEUE.md](GITHUB-AIDLC-QUEUE.md)** for the recommended Projects v2 queue (copy templates from `docs/templates/github-workflows/`). Tier B/C overview: [GITHUB-AIDLC-PROJECT.md](GITHUB-AIDLC-PROJECT.md).
