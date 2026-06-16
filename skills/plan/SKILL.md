---
name: plan
description: AIDLC Plan phase — Product Spec under feature/<slug>/ on a feature branch; creates worktree + AIDLC.md when consumer AGENTS.md documents git worktrees; conversation-first, human approval. Not for quick bugfixes.
type: skill
aidlc_phases: [plan]
tags: [aidlc, orchestrator, plan, product-spec, specs, worktree]
requires: [git-workflow]
author: Melissa Benua
created_at: 2026-04-12
updated_at: 2026-06-16
---

# /plan — Plan (Product Spec)

You are the **phase orchestrator** for AIDLC **Plan** (Product Spec). Ground truth is **`docs/AIDLC.md`** in the **consumer workspace** (e.g. [alexa-recipe-app](https://github.com/dcvezzani/alexa-recipe-app) `docs/AIDLC.md`).

**Design (Tech Spec)** is a **separate** skill: **`/design`** ([skills/design/SKILL.md](../design/SKILL.md)) so a different person can own it after Product approval.

**Library skills and agents** — [docs/SKILLS.md](../../docs/SKILLS.md); resolve from your install or `.claude/skills/<bundle>/`. Apply **`git-workflow`** ([skills/git-workflow/SKILL.md](../git-workflow/SKILL.md)) for branches and worktree lifecycle.

## Before you start

1. Resolve **feature slug** from `$ARGUMENTS` or ask: kebab-case, stable for the life of the feature.
2. **Consumer worktree pre-flight** — read **`AGENTS.md` → Git worktrees (AIDLC)** and [CONSUMER-SETUP.md](../../docs/CONSUMER-SETUP.md) § Git worktrees. When the consumer documents worktrees (or `feature/<slug>/AIDLC.md` already has **Worktree** + **Branch** rows), **do not** create spec files as untracked dirt on the parent checkout (`main`). All Plan work runs on branch **`feature/<slug>`** in the feature worktree:
   - From the **parent repo checkout** on the integration branch (`main`, or parent epic branch per issue / `AIDLC.md`): `git fetch origin && git rebase origin/<base>`.
   - **Path:** expand **`AGENTS.md` → Path pattern** with `<slug>` (default `../<repo>-worktrees/<slug>`). **Branch:** `feature/<slug>` unless `AGENTS.md` defines another pattern.
   - If no worktree exists yet: `git worktree add <path> -b <branch> <base>` (e.g. `origin/main`).
   - **Work in that worktree** for the rest of Plan (open or switch the IDE workspace to the worktree root when needed).
   - Copy **[`feature/_template/AIDLC.md`](../../feature/_template/AIDLC.md)** → `feature/<slug>/AIDLC.md` with absolute **Worktree** and **Branch**; fill **Work item** after the parent issue exists.
   - **Commit** on the feature branch when the human asks — never on `main`. **Push** before using **`blob/<branch>/…`** spec URLs in GitHub issue bodies.
   - When **`AGENTS.md` has no worktree block:** use a feature branch per **`git-workflow`** in a single checkout; worktree creation stays optional until **`/build`**.
3. Ensure `feature/<slug>/` exists (in the feature worktree or feature branch). If empty, copy **[`product-spec-template.md`](../spec-management/templates/product-spec-template.md)** → `product-spec.md` (and optionally seed **`tech-spec-template.md`** → `tech-spec.md` so `/design` has a file to fill — or let `/design` create it; see [design skill](../design/SKILL.md)).
4. **Parent work item:** Read **`AGENTS.md` → Issue tracker (AIDLC)** if the repo documents it — that is the source of truth for which system (GitHub, Linear, Jira, …) holds the Feature. Create or link the **parent** item there:
   - Include the folder path as plain text: `` `feature/<slug>/` ``.
   - Link **Product Spec** per tracker rules. For **`github-issues`**, use a full **`blob/<branch>/…`** URL — **never** a relative markdown link like `(feature/<slug>/product-spec.md)` ([GITHUB-ISSUE-SPEC-LINKS.md](https://github.com/dcvezzani/AI-DLC/blob/main/docs/GITHUB-ISSUE-SPEC-LINKS.md)).
   - Child issues under a parent epic: same rule; use templates in [work-tracking](../work-tracking/SKILL.md) § AIDLC GitHub issue templates.
   If **`AGENTS.md` has no tracker section**, ask which system to use or follow existing queue docs (e.g. `docs/github-queue.md`). For **GitHub Projects (classic)** automation, see [GITHUB-AIDLC-PROJECT.md](https://github.com/dcvezzani/AI-DLC/blob/main/docs/GITHUB-AIDLC-PROJECT.md). For choosing a tracker or filling `AGENTS.md`, see [ISSUE-TRACKER-PORTABILITY.md](https://github.com/dcvezzani/AI-DLC/blob/main/docs/ISSUE-TRACKER-PORTABILITY.md) and **`agent-issue-tracker-setup`**.

## Orchestration — Product Spec (`product-spec.md`)

1. Load **`spec-management`** ([skills/spec-management/SKILL.md](../spec-management/SKILL.md)).
2. Use **`agent-product-manager`** behavior ([skills/agents/agent-product-manager/SKILL.md](../agents/agent-product-manager/SKILL.md)) for a structured draft: problem, outcomes, success criteria, out-of-scope, constraints — per AIDLC Plan in `docs/AIDLC.md`.
3. **Conversation first (required):** **Ask in chat** before treating the spec as ready. Do **not** use a long “open questions” block in the doc instead of talking to the human. Record **resolved** decisions briefly (e.g. **Decisions** subsection) after they answer.
4. Run **`agent-grounding-reviewer`** on the **repo** — blocking vs advisory; don’t rewrite the whole spec silently ([skills/agents/agent-grounding-reviewer/SKILL.md](../agents/agent-grounding-reviewer/SKILL.md)).
5. **Stop for human approval** of the Product Spec.
6. **No** technical implementation, architecture, or API design here — that belongs in **`/design`**.

## Outputs

- `feature/<slug>/product-spec.md`
- `feature/<slug>/AIDLC.md` (when consumer uses git worktrees — Branch + Worktree metadata for downstream phases)

## Handoff to Design

- After approval, the **same or another** person runs **`/design`** for `tech-spec.md` and review passes. **Do not** block on Tech Spec in this run unless the user explicitly asked for both in one session (prefer splitting for separate owners).

## Rules

- Follow AIDLC **orchestration rhythm** in `docs/AIDLC.md` (*Development: Orchestration Model*). User input = **chat**, not only markdown edits.
- Don’t paste large chunks of AIDLC into the spec; **link** where useful.
- **Never commit directly to `main`** — see **`git-workflow`** § Branch Protection.
