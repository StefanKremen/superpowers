# Codex App Compatibility: Worktree and Finishing Skill Adaptation

Make superpowers skills work in Codex App sandboxed worktree env. No break Claude Code / Codex CLI.

**Ticket:** PRI-823

## Motivation

Codex App run agents in git worktrees it manage — detached HEAD, under `$CODEX_HOME/worktrees/`, Seatbelt sandbox block `git checkout -b`, `git push`, network. 3 superpowers skills assume free git: `using-git-worktrees` make manual worktrees w/ named branches, `finishing-a-development-branch` merge/push/PR by branch name, `subagent-driven-development` need both.

Codex CLI (open source terminal) NO conflict — no built-in worktree mgmt. Manual worktree fill isolation gap there. Problem only Codex App.

## Empirical Findings

Tested Codex App 2026-03-23:

| Operation | workspace-write sandbox | Full access sandbox |
|---|---|---|
| `git add` | Works | Works |
| `git commit` | Works | Works |
| `git checkout -b` | **Blocked** (can't write `.git/refs/heads/`) | Works |
| `git push` | **Blocked** (network + `.git/refs/remotes/`) | Works |
| `gh pr create` | **Blocked** (network) | Works |
| `git status/diff/log` | Works | Works |

More:
- `spawn_agent` subagents **share** parent thread fs (confirmed marker file test)
- "Create branch" button in App header regardless of start branch
- App native finishing flow: Create branch → Commit modal → Commit and push / Commit and create PR
- `network_access = true` config silent broken on macOS (issue #10390)

## Design: Read-Only Environment Detection

3 read-only git cmds detect env, no side effects:

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

2 signals:

- **IN_LINKED_WORKTREE:** `GIT_DIR != GIT_COMMON` — agent in worktree made by something else (Codex App, Claude Code Agent tool, prior skill run, user)
- **ON_DETACHED_HEAD:** `BRANCH` empty — no named branch

Why `git-dir != git-common-dir` not `show-toplevel`:
- Normal repo: both → same `.git`
- Linked worktree: `git-dir` = `.git/worktrees/<name>`, `git-common-dir` = `.git`
- Submodule: both equal — avoid false positive `show-toplevel` give
- `cd && pwd -P` handle relative-path (`git-common-dir` return `.git` relative in normal repos but absolute in worktrees) + symlinks (macOS `/tmp` → `/private/tmp`)

### Decision Matrix

| Linked Worktree? | Detached HEAD? | Environment | Action |
|---|---|---|---|
| No | No | Claude Code / Codex CLI / normal git | Full skill behavior (unchanged) |
| Yes | Yes | Codex App worktree (workspace-write) | Skip worktree creation; handoff payload at finish |
| Yes | No | Codex App (Full access) or manual worktree | Skip worktree creation; full finishing flow |
| No | Yes | Unusual (manual detached HEAD) | Create worktree normally; warn at finish |

## Changes

### 1. `using-git-worktrees/SKILL.md` — Add Step 0 (~12 lines)

New section between "Overview" and "Directory Selection Process":

**Step 0: Check if Already in an Isolated Workspace**

Run detection cmds. If `GIT_DIR != GIT_COMMON`, skip worktree creation. Instead:
1. Skip to "Run Project Setup" subsection under Creation Steps — `npm install` etc. idempotent, worth run for safety
2. Then "Verify Clean Baseline" — run tests
3. Report w/ branch state:
   - On branch: "Already in an isolated workspace at `<path>` on branch `<name>`. Tests passing. Ready to implement."
   - Detached HEAD: "Already in an isolated workspace at `<path>` (detached HEAD, externally managed). Tests passing. Note: branch creation needed at finish time. Ready to implement."

If `GIT_DIR == GIT_COMMON`, proceed full worktree creation flow (unchanged).

Safety verification (.gitignore check) skipped when Step 0 fires — irrelevant for externally-created worktrees.

Update Integration "Called by" entries. Change desc on each from context-specific text to: "Ensures isolated workspace (creates one or verifies existing)". E.g., `subagent-driven-development` entry: "REQUIRED: Set up isolated workspace before starting" → "REQUIRED: Ensures isolated workspace (creates one or verifies existing)".

**Sandbox fallback:** If `GIT_DIR == GIT_COMMON` and skill proceed to Creation Steps, but `git worktree add -b` fail w/ permission error (e.g., Seatbelt denial), treat as late-detected restricted env. Fall back to Step 0 "already in workspace" — skip creation, run setup + baseline tests in current dir, report.

After Step 0 report, STOP. No continue to Directory Selection or Creation Steps.

**Else unchanged:** Directory Selection, Safety Verification, Creation Steps, Project Setup, Baseline Tests, Quick Reference, Common Mistakes, Red Flags.

### 2. `finishing-a-development-branch/SKILL.md` — Add Step 1.5 + cleanup guard (~20 lines)

**Step 1.5: Detect Environment** (after Step 1 "Verify Tests", before Step 2 "Determine Base Branch")

Run detection cmds. 3 paths:

- **Path A** skip Steps 2 + 3 (no base branch / options needed).
- **Paths B + C** proceed Step 2 (Determine Base Branch) + Step 3 (Present Options) normal.

**Path A — Externally managed worktree + detached HEAD** (`GIT_DIR != GIT_COMMON` AND `BRANCH` empty):

First, ensure all work staged + committed (`git add` + `git commit`). Codex App finishing controls operate on committed work.

Then present to user (do NOT show 4-option menu):

```
Implementation complete. All tests passing.
Current HEAD: <full-commit-sha>

This workspace is externally managed (detached HEAD).
I cannot create branches, push, or open PRs from here.

⚠ These commits are on a detached HEAD. If you do not create a branch,
they may be lost when this workspace is cleaned up.

If your host application provides these controls:
- "Create branch" — to name a branch, then commit/push/PR
- "Hand off to local" — to move changes to your local checkout

Suggested branch name: <ticket-id/short-description>
Suggested commit message: <summary-of-work>
```

Branch name derive: use ticket ID if available (e.g., `pri-823/codex-compat`), else slugify first 5 words of plan title, else omit suggestion. Avoid sensitive content (vulnerability descriptions, customer names) in branch names.

Skip to Step 5 (cleanup no-op for externally managed worktrees).

**Path B — Externally managed worktree + named branch** (`GIT_DIR != GIT_COMMON` AND `BRANCH` exists):

Show 4-option menu normal. (Step 5 cleanup guard re-detect externally managed state independent.)

**Path C — Normal environment** (`GIT_DIR == GIT_COMMON`):

Show 4-option menu as today (unchanged).

**Step 5 cleanup guard:**

Re-run `GIT_DIR` vs `GIT_COMMON` detection at cleanup time (do not rely on prior skill output — finishing skill may run in different session). If `GIT_DIR != GIT_COMMON`, skip `git worktree remove` — host env owns workspace.

Else, check + remove as today. Note: existing Step 5 says "For Options 1, 2, 4" but Quick Reference table + Common Mistakes say "Options 1 & 4 only." New guard added before existing logic, no change which options trigger cleanup.

**Else unchanged:** Options 1-4 logic, Quick Reference, Common Mistakes, Red Flags.

### 3. `subagent-driven-development/SKILL.md` and `executing-plans/SKILL.md` — 1 line edit each

Both skills have identical Integration section line. Change from:
```
- superpowers:using-git-worktrees - REQUIRED: Set up isolated workspace before starting
```
To:
```
- superpowers:using-git-worktrees - REQUIRED: Ensures isolated workspace (creates one or verifies existing)
```

**Else unchanged:** Dispatch/review loop, prompt templates, model selection, status handling, red flags.

### 4. `codex-tools.md` — Add environment detection docs (~15 lines)

2 new sections at end:

**Environment Detection:**

```markdown
## Environment Detection

Skills that create worktrees or finish branches should detect their
environment with read-only git commands before proceeding:

\```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
\```

- `GIT_DIR != GIT_COMMON` → already in a linked worktree (skip creation)
- `BRANCH` empty → detached HEAD (cannot branch/push/PR from sandbox)

See `using-git-worktrees` Step 0 and `finishing-a-development-branch`
Step 1.5 for how each skill uses these signals.
```

**Codex App Finishing:**

```markdown
## Codex App Finishing

When the sandbox blocks branch/push operations (detached HEAD in an
externally managed worktree), the agent commits all work and informs
the user to use the App's native controls:

- **"Create branch"** — names the branch, then commit/push/PR via App UI
- **"Hand off to local"** — transfers work to the user's local checkout

The agent can still run tests, stage files, and output suggested branch
names, commit messages, and PR descriptions for the user to copy.
```

## What Does NOT Change

- `implementer-prompt.md`, `spec-reviewer-prompt.md`, `code-quality-reviewer-prompt.md` — subagent prompts untouched
- `executing-plans/SKILL.md` — only 1-line Integration desc changes (same as `subagent-driven-development`); runtime behavior unchanged
- `dispatching-parallel-agents/SKILL.md` — no worktree / finishing ops
- `.codex/INSTALL.md` — install process unchanged
- 4-option finishing menu — preserved exact for Claude Code + Codex CLI
- Full worktree creation flow — preserved exact for non-worktree envs
- Subagent dispatch/review/iterate loop — unchanged (fs sharing confirmed)

## Scope Summary

| File | Change |
|---|---|
| `skills/using-git-worktrees/SKILL.md` | +12 lines (Step 0) |
| `skills/finishing-a-development-branch/SKILL.md` | +20 lines (Step 1.5 + cleanup guard) |
| `skills/subagent-driven-development/SKILL.md` | 1 line edit |
| `skills/executing-plans/SKILL.md` | 1 line edit |
| `skills/using-superpowers/references/codex-tools.md` | +15 lines |

~50 lines added/changed across 5 files. Zero new files. Zero breaking changes.

## Future Considerations

If 3rd skill need same detection pattern, extract into shared `references/environment-detection.md` (Approach B). Not now — only 2 skills use.

## Test Plan

### Automated (run in Claude Code after implementation)

1. Normal repo detection — assert IN_LINKED_WORKTREE=false
2. Linked worktree detection — `git worktree add` test worktree, assert IN_LINKED_WORKTREE=true
3. Detached HEAD detection — `git checkout --detach`, assert ON_DETACHED_HEAD=true
4. Finishing skill handoff output — verify handoff message (not 4-option menu) in restricted env
5. **Step 5 cleanup guard** — make linked worktree (`git worktree add /tmp/test-cleanup -b test-cleanup`), `cd` in, run Step 5 cleanup detection (`GIT_DIR` vs `GIT_COMMON`), assert NOT call `git worktree remove`. Then `cd` back main repo, run same detection, assert WOULD call `git worktree remove`. Clean up test worktree after.

### Manual Codex App Tests (5 tests)

1. Detection in Worktree thread (workspace-write) — verify GIT_DIR != GIT_COMMON, empty branch
2. Detection in Worktree thread (Full access) — same detection, different sandbox behavior
3. Finishing skill handoff format — verify agent emit handoff payload, not 4-option menu
4. Full lifecycle — detection → commit → finishing detection → correct behavior → cleanup
5. **Sandbox fallback in Local thread** — Start Codex App **Local thread** (workspace-write sandbox). Prompt: "Use the superpowers skill `using-git-worktrees` to set up an isolated workspace for implementing a small change." Pre-check: `git checkout -b test-sandbox-check` should fail w/ `Operation not permitted`. Expected: skill detect `GIT_DIR == GIT_COMMON` (normal repo), attempt `git worktree add -b`, hit Seatbelt denial, fall back to Step 0 "already in workspace" — run setup, baseline tests, report ready from current dir. Pass: agent recover graceful, no cryptic errors. Fail: agent print raw Seatbelt error, retry, or give up w/ confusing output.

### Regression

- Existing Claude Code skill-triggering tests still pass
- Existing subagent-driven-development integration tests still pass
- Normal Claude Code session: full worktree creation + 4-option finishing still works