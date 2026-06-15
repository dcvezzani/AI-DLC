# Changelog

All notable changes to the AI-DLC skills and docs library.

## [Unreleased]

### Added

- **`skills/git-worktree-cleanup/SKILL.md`** — standalone skill: remove worktree + prune branch by feature slug
- **`feature/_template/AIDLC.md`** — Branch + Worktree metadata for parallel Build and Learn cleanup
- **Git worktree cleanup in `/learn`** — `skills/learn/SKILL.md`, `skills/agents/agent-learn/SKILL.md`, `skills/git-workflow/SKILL.md` § Worktree lifecycle
- **`docs/CONSUMER-SETUP.md`** § Git worktrees (AIDLC) — optional `AGENTS.md` block and conventions

### Changed

- **`docs/templates/AIDLC.md`** — Learn responsibilities and success criteria include worktree/branch hygiene
- **`skills/learn/SKILL.md`** — orchestration step 5; `requires: [git-worktree-cleanup]`
- **`skills/git-workflow/SKILL.md`** — remove/discover delegated to `git-worktree-cleanup`

## [1.0.0] — 2026-06-09

### Added

- **`docs/templates/AIDLC.md`** — canonical process template for consumer repos
- **`docs/CONSUMER-SETUP.md`** — submodule, skill overrides, dual-tracker pattern
- **`docs/INTERACTIVE-UI-VALIDATION.md`** — Chrome DevTools MCP UI validation (distinct from Validate phase)
- **`docs/templates/mcp.json.example`** — sample `.cursor/mcp.json` for Chrome DevTools MCP
- **`skills/learn/SKILL.md`** — Learn orchestrator (after Validate PASS)
- **`docs/GITHUB-AIDLC-QUEUE.md`** — Projects v2 headless queue setup
- **Queue workflow templates** — `aidlc-launch-from-board`, `aidlc-pr-merged`, `aidlc-pr-opened-review`, `aidlc-ship-after-deploy`, `aidlc-issue-comment-launch`, `aidlc-project-phase-reconcile` under `docs/templates/github-workflows/`
- **`.github/actions/aidlc-launch/action.yml`** — org/user Projects v2, Ship CI context inputs

### Changed

- **`skills/ship/SKILL.md`** — Validate phase only; Learn handoff; UI validation by reference
- **`skills/review/SKILL.md`** — Frontend/UX pass requires Chrome DevTools MCP per INTERACTIVE-UI-VALIDATION
- **`skills/build/SKILL.md`** — consumer specialist dispatch section
- **`skills/frontend-web`**, **`skills/testing`** — cross-links to UI validation doc
- **`docs/ISSUE-TRACKER-PORTABILITY.md`** — dual-tracker template, PR ticket gate rows
- **`docs/GITHUB-AIDLC-PROJECT.md`** — automation tiers; Tier A points to queue doc + templates
