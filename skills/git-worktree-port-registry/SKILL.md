---
name: git-worktree-port-registry
description: >-
  Allocate and register dedicated app/api/debug ports for a feature slug and
  git worktree. Writes shared SQLite registry, feature/<slug>/AIDLC.md port
  rows, and .aidlc/dev.env in the worktree. Use during /build, /review, /ship,
  or when starting parallel local dev.
type: skill
aidlc_phases: [build, review, validate]
tags: [git, worktree, ports, registry, parallel, dev]
requires: []
author: Melissa Benua
created_at: 2026-06-15
updated_at: 2026-06-15
---

# Git worktree port registry (by slug)

Allocate **collision-free ports** for a feature **slug** and its **git worktree**. Persist assignments in a **shared SQLite database** (sibling of all worktrees), update **`feature/<slug>/AIDLC.md`**, and write **`.aidlc/dev.env`** in the worktree root for the consumer app and integration tests.

**Typical callers:** **`/build <slug>`** (before local dev/tests), **`/review`** / **`/ship`** (before UI validation), or direct invocation when opening a parallel worktree.

**Related:** [git-workflow](../git-workflow/SKILL.md) § Worktree lifecycle, [git-worktree-cleanup](../git-worktree-cleanup/SKILL.md) (releases ports at Learn), [CONSUMER-SETUP.md](../../docs/CONSUMER-SETUP.md) § Git worktrees + port mapping.

## When to use

- Starting or resuming work in a **feature worktree** for parallel AIDLC
- Before **local dev server**, **integration tests**, or **Chrome DevTools UI validation** in a worktree
- User asks: “allocate ports for `<slug>`”, “register worktree ports”, parallel feature dev setup
- Re-run is **idempotent** — existing slug/role assignments are reused

## Standard port roles

| Role | Env var (port) | Env var (URL) | Typical consumer mapping |
|------|----------------|---------------|--------------------------|
| `app` | `AIDLC_APP_PORT` | `AIDLC_APP_URL` | `PORT`, `VITE_PORT`, UI validation |
| `api` | `AIDLC_API_PORT` | `AIDLC_API_URL` | `API_PORT`, backend integration tests |
| `debug` | `AIDLC_DEBUG_PORT` | — | Node inspector, `DEBUG_PORT`, dotnet debug |

Consumer repos map roles to stack-specific vars in **`AGENTS.md`** — see [CONSUMER-SETUP.md](../../docs/CONSUMER-SETUP.md).

## Inputs

- **slug** — from `$ARGUMENTS` or ask the user
- **worktree_path** — discover (see below) or `git rev-parse --show-toplevel` when run inside the worktree
- **roles** — default `app`, `api`, `debug`; override only when consumer `AGENTS.md` documents fewer/more
- **register-only** — optional explicit ports per role (skip allocation); human must confirm no collision

## Discovery (worktree path + registry location)

### Worktree path

Stop at the **first** match (same order as [git-worktree-cleanup](../git-worktree-cleanup/SKILL.md)):

| # | Source | What to parse |
|---|--------|----------------|
| 1 | `feature/<slug>/AIDLC.md` | `**Worktree**` row |
| 2 | `feature/*/sub-features/<slug>/AIDLC.md` | same |
| 3 | `AGENTS.md` → **Git worktrees (AIDLC)** | expand `**Path pattern**` with `<slug>` |
| 4 | `git rev-parse --show-toplevel` | current checkout when already inside the worktree |

If none found and not inside a worktree: report that port registry requires a recorded or active worktree; exit successfully with `N/A` when single-checkout repo (no parallel worktrees).

### Registry DB path

Derive from consumer **`AGENTS.md` → Git worktrees (AIDLC)**:

- Default parent: `../<repo>-worktrees/` (sibling of repo root)
- Default DB file: `<parent>/aidlc-ports.sqlite`
- Optional override: `**Port registry**` row in the same `AGENTS.md` block

Resolve `<parent>` relative to main repo root (`git rev-parse --show-toplevel` from main checkout, or walk up from worktree path).

## SQLite schema

Create the DB and tables on first use:

```sql
CREATE TABLE IF NOT EXISTS worktrees (
  slug          TEXT PRIMARY KEY,
  worktree_path TEXT NOT NULL UNIQUE,
  repo_root     TEXT NOT NULL,
  created_at    INTEGER NOT NULL,
  updated_at    INTEGER NOT NULL
);

CREATE TABLE IF NOT EXISTS ports (
  slug  TEXT NOT NULL,
  role  TEXT NOT NULL,
  port  INTEGER NOT NULL,
  PRIMARY KEY (slug, role),
  FOREIGN KEY (slug) REFERENCES worktrees(slug) ON DELETE CASCADE,
  UNIQUE (port)
);

CREATE INDEX IF NOT EXISTS idx_ports_port ON ports(port);
```

## Port allocation defaults

Read overrides from **`AGENTS.md`** when present; otherwise:

| Setting | Default |
|---------|---------|
| **Port base** | `18000` |
| **Ports per slug** | `10` (block stride: slug slot × 10 + role offset) |
| **Role offsets** | `app` = 0, `api` = 1, `debug` = 2 |
| **Host** | `127.0.0.1` |

**Allocation algorithm** (inside `BEGIN IMMEDIATE`):

1. If `ports` row exists for `(slug, role)` → reuse.
2. Else compute candidate: `port_base + (slot * ports_per_slug) + role_offset`.
3. **Slot** = lowest non-negative integer where the full role block does not collide with any `ports.port` in the DB.
4. Verify port is not in use on the host (`lsof -i :<port>` or `nc -z 127.0.0.1 <port>`); if busy, try next slot.
5. `INSERT` worktrees + ports rows; commit.

**Register-only mode:** upsert explicit port numbers; fail if `port` already assigned to another slug.

## Workflow

Run from the **worktree checkout** (preferred) or main repo root after discovering paths:

```bash
REPO_ROOT="$(git rev-parse --show-toplevel)"   # may be worktree root
SLUG="<slug>"
WORKTREE_PATH="<absolute-worktree-path>"
REGISTRY_DB="<parent>/aidlc-ports.sqlite"
NOW="$(date +%s)"
HOST="127.0.0.1"

mkdir -p "$(dirname "$REGISTRY_DB")"

sqlite3 "$REGISTRY_DB" "PRAGMA foreign_keys = ON;"

# BEGIN IMMEDIATE — allocate or reuse (see algorithm above)
# Example upsert after ports are known:
sqlite3 "$REGISTRY_DB" <<SQL
BEGIN IMMEDIATE;
INSERT INTO worktrees (slug, worktree_path, repo_root, created_at, updated_at)
VALUES ('$SLUG', '$WORKTREE_PATH', '$REPO_ROOT', $NOW, $NOW)
ON CONFLICT(slug) DO UPDATE SET
  worktree_path = excluded.worktree_path,
  repo_root = excluded.repo_root,
  updated_at = excluded.updated_at;
-- repeat INSERT for each role into ports ...
COMMIT;
SQL

# Write .aidlc/dev.env in the worktree root
mkdir -p "$WORKTREE_PATH/.aidlc"
cat > "$WORKTREE_PATH/.aidlc/dev.env" <<EOF
AIDLC_SLUG=$SLUG
AIDLC_APP_PORT=<app-port>
AIDLC_API_PORT=<api-port>
AIDLC_DEBUG_PORT=<debug-port>
AIDLC_APP_URL=http://${HOST}:<app-port>
AIDLC_API_URL=http://${HOST}:<api-port>
EOF
```

Ensure **`.aidlc/`** is in consumer `.gitignore` (documented in CONSUMER-SETUP).

### Update `feature/<slug>/AIDLC.md`

Fill or update table rows (in the main repo copy of the feature folder when it lives there, or in the worktree):

| Field | Value |
|-------|-------|
| **App port** | `<app-port>` |
| **API port** | `<api-port>` |
| **Debug port** | `<debug-port>` |
| **Dev env file** | `.aidlc/dev.env` |

### Report summary

- Slug
- Worktree path
- Registry DB path
- Ports per role (allocated vs reused)
- Paths to `.aidlc/dev.env` and updated `AIDLC.md`

## Consumer handoff

After this skill runs, the consumer app and tests must load **`.aidlc/dev.env`**:

```bash
set -a && [ -f .aidlc/dev.env ] && . ./.aidlc/dev.env; set +a
npm run dev    # must honor AIDLC_APP_PORT / PORT mapping from AGENTS.md
npm test       # integration tests target AIDLC_APP_URL
```

See [CONSUMER-SETUP.md](../../docs/CONSUMER-SETUP.md) § Port role mapping and consumer checklist.

## UI validation URL resolution

When **`AGENTS.md` → UI validation environments** includes **`Local dev port role`** (default `app`):

1. Read port from `feature/<slug>/AIDLC.md` (**App port**, etc.) or `.aidlc/dev.env`
2. Fallback: query `aidlc-ports.sqlite` for the slug/role
3. Build URL: `http://127.0.0.1:<port>`

If **`Local dev port role`** is absent, use literal **`Local dev URL`** from `AGENTS.md` (single-checkout fallback).

## Safety rules

- One slug per run
- Never overwrite another slug’s `ports.port` row
- Never allocate ports &lt; 1024 without human approval
- If host port is in use by a **different** slug’s process, increment slot — do not kill foreign processes
- Do not commit `.aidlc/dev.env`

## What this does not do

- Create git worktrees — see [git-workflow](../git-workflow/SKILL.md) § Create at Build
- Start the dev server — consumer `dev:aidlc` or equivalent
- Allocate CI runner ports — GitHub Actions uses workflow `env:` (see CONSUMER-SETUP)
- Remove registry rows — see [git-worktree-cleanup](../git-worktree-cleanup/SKILL.md) § Port registry cleanup

## Build / Review / Ship integration

| Phase | When to run |
|-------|-------------|
| **Build** | Before local build + tests (checklist step 1); use `dev:aidlc` with loaded env |
| **Review** | Before Frontend/UX Chrome DevTools pass when UI is in scope |
| **Ship** | Before Validate UI validation against local URL |

If ports already exist in `AIDLC.md` and `.aidlc/dev.env` is present, skip re-allocation unless the user requests refresh.
