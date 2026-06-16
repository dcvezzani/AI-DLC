# Interactive UI validation (Chrome DevTools MCP)

**This is not the AIDLC Validate phase.** The **Validate phase** is Feature-level (`/ship` orchestrator): scorecard vs Product Spec, deploy readiness, tracker closure. **UI validation** is a **technique** for exercising UI acceptance criteria with tool evidence — used **inside** Review (Frontend/UX pass) and **inside** Validate phase when specs include UI success criteria.

---

## Mandatory tool

**`chrome-devtools`** MCP server — [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp).

Configure in the consumer repo’s `.cursor/mcp.json` (see [templates/mcp.json.example](templates/mcp.json.example)). Before first use, confirm the server is loaded and tools exist: `navigate_page`, `take_snapshot`, `fill_form`, `click`, `take_screenshot`, `wait_for`.

---

## Forbidden for interactive UI validation

| Tool | Allowed use |
|------|-------------|
| **`cursor-ide-browser` MCP** | **Never** for agent UI verification — unreliable in many Cloud Agent environments. |
| **Playwright** (smoke, demo, `e2e/`) | **CI and test authoring only** — not for “I checked the UI” claims in Review or Validate. |
| **`computerUse` subagent** | **Not** a substitute for Chrome DevTools MCP evidence. |

---

## Workflow

1. Confirm **`chrome-devtools`** MCP is ready.
2. **Resolve local URL** (see [Local URL resolution](#local-url-resolution-per-worktree)) — then **`navigate_page`** to that URL (or staging from `AGENTS.md`).
3. **`take_snapshot`** — read accessibility `uid`s for links, inputs, buttons.
4. **Sign in (when needed)** — use consumer-declared test credentials; `fill_form` + `click` submit; **`wait_for`** post-login indicator.
5. **Exercise acceptance criteria** — navigate surfaces from the Tech Spec or Product Spec; `click`, `fill`, `wait_for` as needed.
6. **Evidence** — **`take_screenshot`** (PNG). Attach paths in PR comments, `review-report.md`, or validate scorecard artifacts.
7. **Optional** — `list_console_messages`, `list_network_requests` for regressions.

### Honesty rule

Never report code review, inference, or Playwright CI logs as UI validation evidence. If login fails, MCP errors, or criteria were not exercised, record **Fail** or **Unverifiable** with the error text.

---

## Environments (consumer declares in `AGENTS.md`)

Each app repo fills a **`## UI validation environments`** table in **`AGENTS.md`** (template in [CONSUMER-SETUP.md](CONSUMER-SETUP.md)):

| Field | Example |
|-------|---------|
| **Local dev URL** | `http://127.0.0.1:8080` (single-checkout fallback) |
| **Local dev port role** | `app` (when using parallel worktrees — see below) |
| **Staging / pre-prod URL** | `https://app.staging.example.com` |
| **Test credential env vars** | Names only in `AGENTS.md`; values in Cloud Agent dashboard or local env — never commit secrets |

### Local URL resolution (per worktree)

When validating **local** UI in a **feature worktree**, resolve the URL in this order:

1. If **`AGENTS.md` → UI validation environments** includes **`Local dev port role`** (default `app`):
   - Read port from **`feature/<slug>/AIDLC.md`** (**App port**, **API port**, etc. matching the role)
   - Or load **`.aidlc/dev.env`** in the worktree (`AIDLC_APP_URL`, `AIDLC_API_URL`, …)
   - Or query **`../<repo>-worktrees/aidlc-ports.sqlite`** via [git-worktree-port-registry](../skills/git-worktree-port-registry/SKILL.md)
   - Build: `http://127.0.0.1:<port>`
2. If **`Local dev port role`** is absent, use literal **`Local dev URL`** from `AGENTS.md`.

Before UI validation, run **`git-worktree-port-registry <slug>`** when ports are missing from `AIDLC.md` or `.aidlc/dev.env`. Ensure the dev server is running on the allocated port (`dev:aidlc` or equivalent).

For **staging** validation, always use **`Staging / pre-prod URL`** from `AGENTS.md`.

---

## Unavailable MCP

If **`navigate_page`** fails with **“Missing X server”** (common in headless Cloud VMs):

1. Ensure a display exists (e.g. `Xvfb :1` when `DISPLAY=:1` has no server).
2. Retry **`navigate_page`** once the display is up.

If **`chrome-devtools` is not in the MCP catalog**:

- **Stop** interactive UI validation.
- Report: *Chrome DevTools MCP not loaded* — fix `.cursor/mcp.json` and team MCP settings; start a **new** agent session.
- Do **not** substitute Playwright or `cursor-ide-browser`.

---

## Who references this doc

| Phase / artifact | Usage |
|------------------|--------|
| **`/review`** § Frontend/UX | UI validation before merge |
| **`/ship`** (Validate phase) | UI success criteria after deploy/CI gates |
| **`frontend-web`**, **`testing`** skills | Cross-links for manual pre-PR checks |
