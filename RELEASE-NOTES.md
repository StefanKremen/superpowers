# Superpowers Release Notes

## v5.0.7 (2026-03-31)

### GitHub Copilot CLI Support

- **SessionStart context injection** — Copilot CLI v1.0.11 added `additionalContext` support in sessionStart hook output. Session-start hook now detect `COPILOT_CLI` env var, emit SDK-standard `{ "additionalContext": "..." }` format → Copilot CLI users get full superpowers bootstrap at session start. (Original fix by @culinablaz in PR #910)
- **Tool mapping** — added `references/copilot-tools.md` with full Claude Code → Copilot CLI tool equivalence table
- **Skill and README updates** — added Copilot CLI to `using-superpowers` skill platform instructions + README install section

### OpenCode Fixes

- **Skills path consistency** — bootstrap text no longer advertise misleading `configDir/skills/superpowers/` path mismatching runtime path. Agent must use native `skill` tool, not navigate by path. Tests now use consistent paths from single source of truth. (#847, #916)
- **Bootstrap as user message** — moved bootstrap injection from `experimental.chat.system.transform` → `experimental.chat.messages.transform`, prepend to first user message instead of system message. Avoids token bloat from system messages repeated every turn (#750), fixes compat w/ Qwen + other models breaking on multiple system messages (#894).

## v5.0.6 (2026-03-24)

### Inline Self-Review Replaces Subagent Review Loops

Subagent review loop (fresh agent reviews plans/specs) doubled exec time (~25 min overhead) w/o measurably improving plan quality. Regression test 5 versions × 5 trials → identical quality scores regardless of review loop.

- **brainstorming** — replaced Spec Review Loop (subagent dispatch + 3-iter cap) → inline Spec Self-Review checklist: placeholder scan, internal consistency, scope check, ambiguity check
- **writing-plans** — replaced Plan Review Loop (subagent dispatch + 3-iter cap) → inline Self-Review checklist: spec coverage, placeholder scan, type consistency
- **writing-plans** — added explicit "No Placeholders" section defining plan failures (TBD, vague descriptions, undefined refs, "similar to Task N")
- Self-review catch 3-5 real bugs/run in ~30s vs ~25 min, comparable defect rate to subagent approach

### Brainstorm Server

- **Session directory restructured** — brainstorm server session dir now contain two peer subdirs: `content/` (HTML files served to browser) + `state/` (events, server-info, pid, log). Previously server state + user interaction data stored alongside served content → accessible over HTTP. `screen_dir` + `state_dir` paths both in server-started JSON. (Reported by 吉田仁)

### Bug Fixes

- **Owner-PID lifecycle fixes** — brainstorm server owner-PID monitoring had two bugs causing false shutdowns within 60s: (1) EPERM from cross-user PIDs (Tailscale SSH, etc.) treated as "process dead", (2) on WSL grandparent PID resolves to short-lived subprocess exiting before first lifecycle check. Fixed: treat EPERM as "alive" + validate owner PID at startup — if already dead, monitoring disabled, server relies on 30-min idle timeout. Also remove Windows/MSYS2-specific carve-out from `start-server.sh` since server now handles generically. (#879)
- **writing-skills** — corrected false claim SKILL.md frontmatter supports "only two fields"; now says "two required fields" + links agentskills.io spec for all supported fields (PR #882 by @arittr)

### Codex App Compatibility

- **codex-tools** — added named agent dispatch mapping documenting how to translate Claude Code named agent types → Codex `spawn_agent` w/ worker roles (PR #647 by @arittr)
- **codex-tools** — added env detection + Codex App finishing sections for worktree-aware skills (by @arittr)
- **Design spec** — added Codex App compat design spec (PRI-823) covering read-only env detection, worktree-safe skill behavior, sandbox fallback patterns (by @arittr)

## v5.0.5 (2026-03-17)

### Bug Fixes

- **Brainstorm server ESM fix** — renamed `server.js` → `server.cjs` so brainstorming server starts on Node.js 22+ where root `package.json` `"type": "module"` caused `require()` fail. (PR #784 by @sarbojitrana, fixes #774, #780, #783)
- **Brainstorm owner-PID on Windows** — skip PID lifecycle monitoring on Windows/MSYS2 where PID namespace invisible to Node.js → prevent server self-terminating after 60s. (#770, docs from PR #768 by @lucasyhzlu-debug)
- **stop-server.sh reliability** — verify server process actually died before reporting success. SIGTERM + 2s wait + SIGKILL fallback. (#723)

### Changed

- **Execution handoff** — restore user choice between subagent-driven + inline execution after plan writing. Subagent-driven recommended but no longer mandatory.

## v5.0.4 (2026-03-16)

### Review Loop Refinements

Cuts token usage + speeds spec/plan reviews by removing unnecessary review passes + tightening reviewer focus.

- **Single whole-plan review** — plan reviewer now reviews complete plan in one pass instead of chunk-by-chunk. Removed all chunk concepts (`## Chunk N:` headings, 1000-line chunk limits, per-chunk dispatch).
- **Raised bar for blocking issues** — both spec + plan reviewer prompts now include "Calibration" section: only flag issues causing real problems during impl. Minor wording, stylistic prefs, formatting quibbles must not block approval.
- **Reduced max review iterations** — 5 → 3 for both spec + plan review loops. If reviewer calibrated correctly, 3 rounds plenty.
- **Streamlined reviewer checklists** — spec reviewer trimmed 7 → 5 categories; plan reviewer 7 → 4. Removed formatting checks (task syntax, chunk size) → favor substance (buildability, spec alignment).

### OpenCode

- **One-line plugin install** — OpenCode plugin now auto-registers skills dir via `config` hook. No symlinks or `skills.paths` config needed. Install = adding one line to `opencode.json`. (PR #753)
- **Added `package.json`** so OpenCode can install superpowers as npm package from git.

### Bug Fixes

- **Verify server actually stopped** — `stop-server.sh` now confirms process dead before reporting success. SIGTERM + 2s wait + SIGKILL fallback. Reports failure if process survives. (PR #751)
- **Generic agent language** — brainstorm companion waiting page now says "the agent" instead of "Claude".

## v5.0.3 (2026-03-15)

### Cursor Support

- **Cursor hooks** — added `hooks/hooks-cursor.json` w/ Cursor camelCase format (`sessionStart`, `version: 1`), updated `.cursor-plugin/plugin.json` to reference. Fixed platform detection in `session-start` to check `CURSOR_PLUGIN_ROOT` first (Cursor may also set `CLAUDE_PLUGIN_ROOT`). (Based on PR #709)

### Bug Fixes

- **Stop firing SessionStart hook on `--resume`** — startup hook was re-injecting context on resumed sessions which already have context in conversation history. Hook now fires only on `startup`, `clear`, `compact`.
- **Bash 5.3+ hook hang** — replaced heredoc (`cat <<EOF`) with `printf` in `hooks/session-start`. Fixes indefinite hang on macOS w/ Homebrew bash 5.3+ caused by bash regression w/ large variable expansion in heredocs. (#572, #571)
- **POSIX-safe hook script** — replaced `${BASH_SOURCE[0]:-$0}` with `$0` in `hooks/session-start`. Fixes "Bad substitution" error on Ubuntu/Debian where `/bin/sh` is dash. (#553)
- **Portable shebangs** — replaced `#!/bin/bash` with `#!/usr/bin/env bash` in all shell scripts. Fixes exec on NixOS, FreeBSD, macOS w/ Homebrew bash where `/bin/bash` outdated/missing. (#700)
- **Brainstorm server on Windows** — auto-detect Windows/Git Bash (`OSTYPE=msys*`, `MSYSTEM`), switch to foreground mode → fixes silent server failure caused by `nohup`/`disown` process reaping. (#737)
- **Codex docs fix** — replaced deprecated `collab` flag w/ `multi_agent` in Codex docs. (PR #749)

## v5.0.2 (2026-03-11)

### Zero-Dependency Brainstorm Server

**Removed all vendored node_modules — server.js now fully self-contained**

- Replaced Express/Chokidar/WebSocket deps w/ zero-dep Node.js server using built-in `http`, `fs`, `crypto` modules
- Removed ~1,200 lines of vendored `node_modules/`, `package.json`, `package-lock.json`
- Custom WebSocket protocol impl (RFC 6455 framing, ping/pong, proper close handshake)
- Native `fs.watch()` file watching replaces Chokidar
- Full test suite: HTTP serving, WebSocket protocol, file watching, integration tests

### Brainstorm Server Reliability

- **Auto-exit after 30 min idle** — server shuts down when no clients connected → prevent orphaned processes
- **Owner process tracking** — server monitors parent harness PID, exits when owning session dies
- **Liveness check** — skill verifies server responsive before reusing existing instance
- **Encoding fix** — proper `<meta charset="utf-8">` on served HTML pages

### Subagent Context Isolation

- All delegation skills (brainstorming, dispatching-parallel-agents, requesting-code-review, subagent-driven-development, writing-plans) now include context isolation principle
- Subagents receive only context they need → prevent context window pollution

## v5.0.1 (2026-03-10)

### Agentskills Compliance

**Brainstorm-server moved into skill directory**

- Moved `lib/brainstorm-server/` → `skills/brainstorming/scripts/` per [agentskills.io](https://agentskills.io) spec
- All `${CLAUDE_PLUGIN_ROOT}/lib/brainstorm-server/` refs replaced w/ relative `scripts/` paths
- Skills now fully portable across platforms — no platform-specific env vars needed to locate scripts
- `lib/` dir removed (was last remaining content)

### New Features

**Gemini CLI extension**

- Native Gemini CLI extension support via `gemini-extension.json` + `GEMINI.md` at repo root
- `GEMINI.md` @imports `using-superpowers` skill + tool mapping table at session start
- Gemini CLI tool mapping ref (`skills/using-superpowers/references/gemini-tools.md`) — translates Claude Code tool names (Read, Write, Edit, Bash, etc.) → Gemini CLI equivalents (read_file, write_file, replace, etc.)
- Documents Gemini CLI limits: no subagent support, skills fall back to `executing-plans`
- Extension root at repo root for cross-platform compat (avoids Windows symlink issues)
- Install instructions in README

### Improvements

**Multi-platform brainstorm server launch**

- Per-platform launch instructions in visual-companion.md: Claude Code (default mode), Codex (auto-foreground via `CODEX_CI`), Gemini CLI (`--foreground` w/ `is_background`), fallback for other envs
- Server now writes startup JSON to `$SCREEN_DIR/.server-info` → agents can find URL + port even when stdout hidden by background exec

**Brainstorm server dependencies bundled**

- `node_modules` vendored into repo → brainstorm server works immediately on fresh plugin installs w/o `npm` at runtime
- Removed `fsevents` from bundled deps (macOS-only native binary; chokidar falls back gracefully w/o it)
- Fallback auto-install via `npm install` if `node_modules` missing

**OpenCode tool mapping fix**

- `TodoWrite` → `todowrite` (was incorrectly mapped to `update_plan`); verified against OpenCode source

### Bug Fixes

**Windows/Linux: single quotes break SessionStart hook** (#577, #529, #644, PR #585)

- Single quotes around `${CLAUDE_PLUGIN_ROOT}` in hooks.json fail on Windows (cmd.exe doesn't recognize single quotes as path delimiters) + Linux (single quotes prevent variable expansion)
- Fix: replaced single quotes w/ escaped double quotes — works across macOS bash, Windows cmd.exe, Windows Git Bash, Linux, w/ + w/o spaces in paths
- Verified on Windows 11 (NT 10.0.26200.0) w/ Claude Code 2.1.72 + Git for Windows

**Brainstorming spec review loop skipped** (#677)

- Spec review loop (dispatch spec-document-reviewer subagent, iterate until approved) existed in prose "After the Design" section but missing from checklist + process flow diagram
- Since agents follow diagram + checklist more reliably than prose, spec review step skipped entirely
- Added step 7 (spec review loop) to checklist + corresponding nodes to dot graph
- Tested w/ `claude --plugin-dir` + `claude-session-driver`: worker now correctly dispatches reviewer

**Cursor install command** (PR #676)

- Fixed Cursor install command in README: `/plugin-add` → `/add-plugin` (confirmed via Cursor 2.5 release announcement)

**User review gate in brainstorming** (#565)

- Added explicit user review step between spec completion + writing-plans handoff
- User must approve spec before impl planning begins
- Checklist, process flow, prose updated w/ new gate

**Session-start hook emits context only once per platform**

- Hook now detects whether running in Claude Code or another platform
- Emits `hookSpecificOutput` for Claude Code, `additional_context` for others → prevent double context injection

**Linting fix in token analysis script**

- `except:` → `except Exception:` in `tests/claude-code/analyze-token-usage.py`

### Maintenance

**Removed dead code**

- Deleted `lib/skills-core.js` + its test (`tests/opencode/test-skills-core.js`) — unused since February 2026
- Removed skills-core existence check from `tests/opencode/test-plugin-loading.sh`

### Community

- @karuturi — Claude Code official marketplace install instructions (PR #610)
- @mvanhorn — session-start hook dual-emit fix, OpenCode tool mapping fix
- @daniel-graham — linting fix for bare except
- PR #585 author — Windows/Linux hooks quoting fix

---

## v5.0.0 (2026-03-09)

### Breaking Changes

**Specs and plans directory restructured**

- Specs (brainstorming output) now save to `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`
- Plans (writing-plans output) now save to `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- User prefs for spec/plan locations override these defaults
- All internal skill refs, test files, example paths updated to match
- Migration: move existing files from `docs/plans/` → new locations if desired

**Subagent-driven development mandatory on capable harnesses**

Writing-plans no longer offers choice between subagent-driven + executing-plans. On harnesses w/ subagent support (Claude Code, Codex), subagent-driven-development required. Executing-plans reserved for harnesses w/o subagent capability, now tells user Superpowers works better on subagent-capable platform.

**Executing-plans no longer batches**

Removed "execute 3 tasks then stop for review" pattern. Plans now execute continuously, stop only for blockers.

**Slash commands deprecated**

`/brainstorm`, `/write-plan`, `/execute-plan` now show deprecation notices pointing users to corresponding skills. Commands removed in next major release.

### New Features

**Visual brainstorming companion**

Optional browser-based companion for brainstorming sessions. When topic benefits from visuals, brainstorming skill offers to show mockups, diagrams, comparisons + other content in browser window alongside terminal conversation.

- `lib/brainstorm-server/` — WebSocket server w/ browser helper lib, session mgmt scripts, dark/light themed frame template ("Superpowers Brainstorming" w/ GitHub link)
- `skills/brainstorming/visual-companion.md` — Progressive disclosure guide for server workflow, screen authoring, feedback collection
- Brainstorming skill adds visual companion decision point to process flow: after exploring project context, skill evaluates whether upcoming questions involve visual content + offers companion in own message
- Per-question decision: even after accepting, each question evaluated for whether browser/terminal more appropriate
- Integration tests in `tests/brainstorm-server/`

**Document review system**

Automated review loops for spec + plan documents using subagent dispatch:

- `skills/brainstorming/spec-document-reviewer-prompt.md` — Reviewer checks completeness, consistency, architecture, YAGNI
- `skills/writing-plans/plan-document-reviewer-prompt.md` — Reviewer checks spec alignment, task decomposition, file structure, file size
- Brainstorming dispatches spec reviewer after writing design doc
- Writing-plans includes chunk-based plan review loop after each section
- Review loops repeat until approved or escalate after 5 iterations
- End-to-end tests in `tests/claude-code/test-document-review-system.sh`
- Design spec + impl plan in `docs/superpowers/`

**Architecture guidance across skill pipeline**

Design-for-isolation + file-size-awareness guidance added to brainstorming, writing-plans, subagent-driven-development:

- **Brainstorming** — New sections: "Design for isolation and clarity" (clear boundaries, well-defined interfaces, independently testable units) + "Working in existing codebases" (follow existing patterns, targeted improvements only)
- **Writing-plans** — New "File Structure" section: map files + responsibilities before defining tasks. New "Scope Check" backstop: catch multi-subsystem specs that should have been decomposed during brainstorming
- **SDD implementer** — New "Code Organization" section (follow plan file structure, report concerns about growing files) + "When You're in Over Your Head" escalation guidance
- **SDD code quality reviewer** — Now checks architecture, unit decomposition, plan conformance, file growth
- **Spec/plan reviewers** — Architecture + file size added to review criteria
- **Scope assessment** — Brainstorming now assesses whether project too large for single spec. Multi-subsystem requests flagged early + decomposed into sub-projects, each w/ own spec → plan → impl cycle

**Subagent-driven development improvements**

- **Model selection** — Guidance for choosing model capability by task type: cheap models for mechanical impl, standard for integration, capable for architecture + review
- **Implementer status protocol** — Subagents now report DONE, DONE_WITH_CONCERNS, BLOCKED, NEEDS_CONTEXT. Controller handles each status: re-dispatch w/ more context, upgrade model capability, break tasks apart, escalate to human

### Improvements

**Instruction priority hierarchy**

Added explicit priority ordering to using-superpowers:

1. User explicit instructions (CLAUDE.md, AGENTS.md, direct requests) — highest priority
2. Superpowers skills — override default system behavior
3. Default system prompt — lowest priority

If CLAUDE.md or AGENTS.md says "don't use TDD" + skill says "always use TDD," user instructions win.

**SUBAGENT-STOP gate**

Added `<SUBAGENT-STOP>` block to using-superpowers. Subagents dispatched for specific tasks now skip skill instead of activating 1% rule + invoking full skill workflows.

**Multi-platform improvements**

- Codex tool mapping moved to progressive disclosure ref file (`references/codex-tools.md`)
- Platform Adaptation pointer added → non-Claude-Code platforms can find tool equivalents
- Plan headers now address "agentic workers" instead of "Claude" specifically
- Collab feature requirement documented in `docs/README.codex.md`

**Writing-plans template updates**

- Plan steps now use checkbox syntax (`- [ ] **Step N:**`) for progress tracking
- Plan header references both subagent-driven-development + executing-plans w/ platform-aware routing

---

## v4.3.1 (2026-02-21)

### Added

**Cursor support**

Superpowers now works w/ Cursor plugin system. Includes `.cursor-plugin/plugin.json` manifest + Cursor-specific install instructions in README. SessionStart hook output now includes `additional_context` field alongside existing `hookSpecificOutput.additionalContext` for Cursor hook compat.

### Fixed

**Windows: Restored polyglot wrapper for reliable hook execution (#518, #504, #491, #487, #466, #440)**

Claude Code `.sh` auto-detection on Windows was prepending `bash` to hook command → broke exec. Fix:

- Renamed `session-start.sh` → `session-start` (extensionless) so auto-detection doesn't interfere
- Restored `run-hook.cmd` polyglot wrapper w/ multi-location bash discovery (standard Git for Windows paths, then PATH fallback)
- Exits silently if no bash found rather than erroring
- On Unix, wrapper runs script directly via `exec bash`
- Uses POSIX-safe `dirname "$0"` path resolution (works on dash/sh, not just bash)

Fixes SessionStart failures on Windows w/ spaces in paths, missing WSL, `set -euo pipefail` fragility on MSYS, backslash mangling.

## v4.3.0 (2026-02-12)

Fix should dramatically improve superpowers skills compliance + reduce chances of Claude entering native plan mode unintentionally.

### Changed

**Brainstorming skill now enforces workflow instead of describing it**

Models were skipping design phase + jumping straight to impl skills like frontend-design, or collapsing entire brainstorming process into single text block. Skill now uses hard gates, mandatory checklist, graphviz process flow to enforce compliance:

- `<HARD-GATE>`: no impl skills, code, or scaffolding until design presented + user approves
- Explicit checklist (6 items) must be created as tasks + completed in order
- Graphviz process flow w/ `writing-plans` as only valid terminal state
- Anti-pattern callout for "this is too simple to need a design" — exact rationalization models use to skip process
- Design section sizing based on section complexity, not project complexity

**Using-superpowers workflow graph intercepts EnterPlanMode**

Added `EnterPlanMode` intercept to skill flow graph. When model about to enter Claude native plan mode, checks whether brainstorming happened + routes through brainstorming skill instead. Plan mode never entered.

### Fixed

**SessionStart hook now runs synchronously**

Changed `async: true` → `async: false` in hooks.json. When async, hook could fail to complete before model first turn → using-superpowers instructions weren't in context for first message.

## v4.2.0 (2026-02-05)

### Breaking Changes

**Codex: Replaced bootstrap CLI with native skill discovery**

`superpowers-codex` bootstrap CLI, Windows `.cmd` wrapper, related bootstrap content file removed. Codex now uses native skill discovery via `~/.agents/skills/superpowers/` symlink → old `use_skill`/`find_skills` CLI tools no longer needed.

Install now = clone + symlink (documented in INSTALL.md). No Node.js dep required. Old `~/.codex/skills/` path deprecated.

### Fixes

**Windows: Fixed Claude Code 2.1.x hook execution (#331)**

Claude Code 2.1.x changed how hooks execute on Windows: now auto-detects `.sh` files in commands + prepends `bash`. Broke polyglot wrapper pattern because `bash "run-hook.cmd" session-start.sh` tries to execute `.cmd` file as bash script.

Fix: hooks.json now calls session-start.sh directly. Claude Code 2.1.x handles bash invocation automatically. Also added .gitattributes to enforce LF line endings for shell scripts (fixes CRLF issues on Windows checkout).

**Windows: SessionStart hook runs async to prevent terminal freeze (#404, #413, #414, #419)**

Synchronous SessionStart hook blocked TUI from entering raw mode on Windows → froze all keyboard input. Running hook async prevents freeze while still injecting superpowers context.

**Windows: Fixed O(n^2) `escape_for_json` performance**

Char-by-char loop using `${input:$i:1}` was O(n^2) in bash due to substring copy overhead. On Windows Git Bash took 60+ seconds. Replaced w/ bash parameter substitution (`${s//old/new}`) which runs each pattern as single C-level pass — 7x faster on macOS, dramatically faster on Windows.

**Codex: Fixed Windows/PowerShell invocation (#285, #243)**

- Windows doesn't respect shebangs → directly invoking extensionless `superpowers-codex` script triggered "Open with" dialog. All invocations now prefixed w/ `node`.
- Fixed `~/` path expansion on Windows — PowerShell doesn't expand `~` when passed as arg to `node`. Changed to `$HOME` which expands correctly in both bash + PowerShell.

**Codex: Fixed path resolution in installer**

Used `fileURLToPath()` instead of manual URL pathname parsing → correctly handle paths w/ spaces + special chars on all platforms.

**Codex: Fixed stale skills path in writing-skills**

Updated `~/.codex/skills/` ref (deprecated) → `~/.agents/skills/` for native discovery.

### Improvements

**Worktree isolation now required before implementation**

Added `using-git-worktrees` as required skill for both `subagent-driven-development` + `executing-plans`. Impl workflows now explicitly require setting up isolated worktree before starting work → prevent accidental work directly on main.

**Main branch protection softened to require explicit consent**

Instead of prohibiting main branch work entirely, skills now allow it w/ explicit user consent. More flexible while still ensuring users aware of implications.

**Simplified installation verification**

Removed `/help` command check + specific slash command list from verification steps. Skills primarily invoked by describing what you want to do, not running specific commands.

**Codex: Clarified subagent tool mapping in bootstrap**

Improved docs of how Codex tools map to Claude Code equivalents for subagent workflows.

### Tests

- Added worktree requirement test for subagent-driven-development
- Added main branch red flag warning test
- Fixed case sensitivity in skill recognition test assertions

---

## v4.1.1 (2026-01-23)

### Fixes

**OpenCode: Standardized on `plugins/` directory per official docs (#343)**

OpenCode official docs use `~/.config/opencode/plugins/` (plural). Our docs previously used `plugin/` (singular). While OpenCode accepts both forms, we standardized on official convention to avoid confusion.

Changes:
- Renamed `.opencode/plugin/` → `.opencode/plugins/` in repo structure
- Updated all install docs (INSTALL.md, README.opencode.md) across all platforms
- Updated test scripts to match

**OpenCode: Fixed symlink instructions (#339, #342)**

- Added explicit `rm` before `ln -s` (fixes "file already exists" errors on reinstall)
- Added missing skills symlink step that was absent from INSTALL.md
- Updated from deprecated `use_skill`/`find_skills` → native `skill` tool refs

---

## v4.1.0 (2026-01-23)

### Breaking Changes

**OpenCode: Switched to native skills system**

Superpowers for OpenCode now uses OpenCode native `skill` tool instead of custom `use_skill`/`find_skills` tools. Cleaner integration that works w/ OpenCode built-in skill discovery.

**Migration required:** Skills must be symlinked to `~/.config/opencode/skills/superpowers/` (see updated install docs).

### Fixes

**OpenCode: Fixed agent reset on session start (#226)**

Previous bootstrap injection method using `session.prompt({ noReply: true })` caused OpenCode to reset selected agent to "build" on first message. Now uses `experimental.chat.system.transform` hook which modifies system prompt directly w/o side effects.

**OpenCode: Fixed Windows installation (#232)**

- Removed dep on `skills-core.js` (eliminates broken relative imports when file copied instead of symlinked)
- Added comprehensive Windows install docs for cmd.exe, PowerShell, Git Bash
- Documented proper symlink vs junction usage for each platform

**Claude Code: Fixed Windows hook execution for Claude Code 2.1.x**

Claude Code 2.1.x changed how hooks execute on Windows: now auto-detects `.sh` files in commands + prepends `bash `. Broke polyglot wrapper pattern because `bash "run-hook.cmd" session-start.sh` tries to execute .cmd file as bash script.

Fix: hooks.json now calls session-start.sh directly. Claude Code 2.1.x handles bash invocation automatically. Also added .gitattributes to enforce LF line endings for shell scripts (fixes CRLF issues on Windows checkout).

---

## v4.0.3 (2025-12-26)

### Improvements

**Strengthened using-superpowers skill for explicit skill requests**

Addressed failure mode where Claude would skip invoking skill even when user explicitly requested by name (e.g., "subagent-driven-development, please"). Claude would think "I know what that means" + start working directly instead of loading skill.

Changes:
- Updated "The Rule" to say "Invoke relevant or requested skills" instead of "Check for skills" — emphasizing active invocation over passive checking
- Added "BEFORE any response or action" — original wording only mentioned "response" but Claude would sometimes take action w/o responding first
- Added reassurance that invoking wrong skill is okay — reduces hesitation
- Added new red flag: "I know what that means" → Knowing concept ≠ using skill

**Added explicit skill request tests**

New test suite in `tests/explicit-skill-requests/` verifies Claude correctly invokes skills when users request by name. Includes single-turn + multi-turn test scenarios.

## v4.0.2 (2025-12-23)

### Fixes

**Slash commands now user-only**

Added `disable-model-invocation: true` to all three slash commands (`/brainstorm`, `/execute-plan`, `/write-plan`). Claude can no longer invoke these commands via Skill tool — restricted to manual user invocation only.

Underlying skills (`superpowers:brainstorming`, `superpowers:executing-plans`, `superpowers:writing-plans`) remain available for Claude to invoke autonomously. Change prevents confusion when Claude would invoke command that just redirects to skill anyway.

## v4.0.1 (2025-12-23)

### Fixes

**Clarified how to access skills in Claude Code**

Fixed confusing pattern where Claude would invoke skill via Skill tool, then try to Read skill file separately. `using-superpowers` skill now explicitly states Skill tool loads skill content directly — no need to read files.

- Added "How to Access Skills" section to `using-superpowers`
- Changed "read the skill" → "invoke the skill" in instructions
- Updated slash commands to use fully qualified skill names (e.g., `superpowers:brainstorming`)

**Added GitHub thread reply guidance to receiving-code-review** (h/t @ralphbean)

Added note about replying to inline review comments in original thread rather than as top-level PR comments.

**Added automation-over-documentation guidance to writing-skills** (h/t @EthanJStark)

Added guidance that mechanical constraints should be automated, not documented — save skills for judgment calls.

## v4.0.0 (2025-12-17)

### New Features

**Two-stage code review in subagent-driven-development**

Subagent workflows now use two separate review stages after each task:

1. **Spec compliance review** - Skeptical reviewer verifies impl matches spec exactly. Catches missing requirements AND over-building. Won't trust implementer report — reads actual code.

2. **Code quality review** - Only runs after spec compliance passes. Reviews for clean code, test coverage, maintainability.

Catches common failure mode where code well-written but doesn't match what was requested. Reviews are loops, not one-shot: if reviewer finds issues, implementer fixes them, then reviewer checks again.

Other subagent workflow improvements:
- Controller provides full task text to workers (not file refs)
- Workers can ask clarifying questions before AND during work
- Self-review checklist before reporting completion
- Plan read once at start, extracted to TodoWrite

New prompt templates in `skills/subagent-driven-development/`:
- `implementer-prompt.md` - Includes self-review checklist, encourages questions
- `spec-reviewer-prompt.md` - Skeptical verification against requirements
- `code-quality-reviewer-prompt.md` - Standard code review

**Debugging techniques consolidated with tools**

`systematic-debugging` now bundles supporting techniques + tools:
- `root-cause-tracing.md` - Trace bugs backward through call stack
- `defense-in-depth.md` - Add validation at multiple layers
- `condition-based-waiting.md` - Replace arbitrary timeouts w/ condition polling
- `find-polluter.sh` - Bisection script to find which test creates pollution
- `condition-based-waiting-example.ts` - Complete impl from real debugging session

**Testing anti-patterns reference**

`test-driven-development` now includes `testing-anti-patterns.md` covering:
- Testing mock behavior instead of real behavior
- Adding test-only methods to production classes
- Mocking w/o understanding deps
- Incomplete mocks that hide structural assumptions

**Skill test infrastructure**

Three new test frameworks for validating skill behavior:

`tests/skill-triggering/` - Validates skills trigger from naive prompts w/o explicit naming. Tests 6 skills to ensure descriptions alone sufficient.

`tests/claude-code/` - Integration tests using `claude -p` for headless testing. Verifies skill usage via session transcript (JSONL) analysis. Includes `analyze-token-usage.py` for cost tracking.

`tests/subagent-driven-dev/` - End-to-end workflow validation w/ two complete test projects:
- `go-fractals/` - CLI tool w/ Sierpinski/Mandelbrot (10 tasks)
- `svelte-todo/` - CRUD app w/ localStorage + Playwright (12 tasks)

### Major Changes

**DOT flowcharts as executable specifications**

Rewrote key skills using DOT/GraphViz flowcharts as authoritative process definition. Prose becomes supporting content.

**The Description Trap** (documented in `writing-skills`): Discovered skill descriptions override flowchart content when descriptions contain workflow summaries. Claude follows short description instead of reading detailed flowchart. Fix: descriptions must be trigger-only ("Use when X") w/ no process details.

**Skill priority in using-superpowers**

When multiple skills apply, process skills (brainstorming, debugging) now explicitly come before impl skills. "Build X" triggers brainstorming first, then domain skills.

**brainstorming trigger strengthened**

Description changed to imperative: "You MUST use this before any creative work—creating features, building components, adding functionality, or modifying behavior."

### Breaking Changes

**Skill consolidation** - Six standalone skills merged:
- `root-cause-tracing`, `defense-in-depth`, `condition-based-waiting` → bundled in `systematic-debugging/`
- `testing-skills-with-subagents` → bundled in `writing-skills/`
- `testing-anti-patterns` → bundled in `test-driven-development/`
- `sharing-skills` removed (obsolete)

### Other Improvements

- **render-graphs.js** - Tool to extract DOT diagrams from skills + render to SVG
- **Rationalizations table** in using-superpowers - Scannable format including new entries: "I need more context first", "Let me explore first", "This feels productive"
- **docs/testing.md** - Guide to testing skills w/ Claude Code integration tests

---

## v3.6.2 (2025-12-03)

### Fixed

- **Linux Compatibility**: Fixed polyglot hook wrapper (`run-hook.cmd`) to use POSIX-compliant syntax
  - Replaced bash-specific `${BASH_SOURCE[0]:-$0}` w/ standard `$0` on line 16
  - Resolves "Bad substitution" error on Ubuntu/Debian systems where `/bin/sh` is dash
  - Fixes #141

---

## v3.5.1 (2025-11-24)

### Changed

- **OpenCode Bootstrap Refactor**: Switched from `chat.message` hook → `session.created` event for bootstrap injection
  - Bootstrap now injects at session creation via `session.prompt()` w/ `noReply: true`
  - Explicitly tells model using-superpowers already loaded → prevent redundant skill loading
  - Consolidated bootstrap content generation into shared `getBootstrapContent()` helper
  - Cleaner single-impl approach (removed fallback pattern)

---

## v3.5.0 (2025-11-23)

### Added

- **OpenCode Support**: Native JavaScript plugin for OpenCode.ai
  - Custom tools: `use_skill` + `find_skills`
  - Message insertion pattern for skill persistence across context compaction
  - Automatic context injection via chat.message hook
  - Auto re-injection on session.compacted events
  - Three-tier skill priority: project > personal > superpowers
  - Project-local skills support (`.opencode/skills/`)
  - Shared core module (`lib/skills-core.js`) for code reuse w/ Codex
  - Automated test suite w/ proper isolation (`tests/opencode/`)
  - Platform-specific docs (`docs/README.opencode.md`, `docs/README.codex.md`)

### Changed

- **Refactored Codex Implementation**: Now uses shared `lib/skills-core.js` ES module
  - Eliminates code duplication between Codex + OpenCode
  - Single source of truth for skill discovery + parsing
  - Codex successfully loads ES modules via Node.js interop

- **Improved Documentation**: Rewrote README to explain problem/solution clearly
  - Removed duplicate sections + conflicting info
  - Added complete workflow description (brainstorm → plan → execute → finish)
  - Simplified platform install instructions
  - Emphasized skill-checking protocol over auto activation claims

---

## v3.4.1 (2025-10-31)

### Improvements

- Optimized superpowers bootstrap to eliminate redundant skill exec. `using-superpowers` skill content now provided directly in session context, w/ clear guidance to use Skill tool only for other skills. Reduces overhead + prevents confusing loop where agents would execute `using-superpowers` manually despite already having content from session start.

## v3.4.0 (2025-10-30)

### Improvements

- Simplified `brainstorming` skill → return to original conversational vision. Removed heavyweight 6-phase process w/ formal checklists in favor of natural dialogue: ask questions one at a time, then present design in 200-300 word sections w/ validation. Keeps docs + impl handoff features.

## v3.3.1 (2025-10-28)

### Improvements

- Updated `brainstorming` skill to require autonomous recon before questioning, encourage recommendation-driven decisions, prevent agents from delegating prioritization back to humans.
- Applied writing clarity improvements to `brainstorming` skill following Strunk's "Elements of Style" principles (omitted needless words, converted negative → positive form, improved parallel construction).

### Bug Fixes

- Clarified `writing-skills` guidance → points to correct agent-specific personal skill directories (`~/.claude/skills` for Claude Code, `~/.codex/skills` for Codex).

## v3.3.0 (2025-10-28)

### New Features

**Experimental Codex Support**
- Added unified `superpowers-codex` script w/ bootstrap/use-skill/find-skills commands
- Cross-platform Node.js impl (works on Windows, macOS, Linux)
- Namespaced skills: `superpowers:skill-name` for superpowers skills, `skill-name` for personal
- Personal skills override superpowers skills when names match
- Clean skill display: shows name/description w/o raw frontmatter
- Helpful context: shows supporting files dir for each skill
- Tool mapping for Codex: TodoWrite→update_plan, subagents→manual fallback, etc.
- Bootstrap integration w/ minimal AGENTS.md for automatic startup
- Complete install guide + bootstrap instructions specific to Codex

**Key differences from Claude Code integration:**
- Single unified script instead of separate tools
- Tool substitution system for Codex-specific equivalents
- Simplified subagent handling (manual work instead of delegation)
- Updated terminology: "Superpowers skills" instead of "Core skills"

### Files Added
- `.codex/INSTALL.md` - Install guide for Codex users
- `.codex/superpowers-bootstrap.md` - Bootstrap instructions w/ Codex adaptations
- `.codex/superpowers-codex` - Unified Node.js executable w/ all functionality

**Note:** Codex support experimental. Integration provides core superpowers functionality but may require refinement based on user feedback.

## v3.2.3 (2025-10-23)

### Improvements

**Updated using-superpowers skill to use Skill tool instead of Read tool**
- Changed skill invocation instructions from Read tool → Skill tool
- Updated description: "using Read tool" → "using Skill tool"
- Updated step 3: "Use the Read tool" → "Use the Skill tool to read and run"
- Updated rationalization list: "Read the current version" → "Run the current version"

Skill tool = proper mechanism for invoking skills in Claude Code. Update corrects bootstrap instructions to guide agents toward correct tool.

### Files Changed
- Updated: `skills/using-superpowers/SKILL.md` - Changed tool refs from Read → Skill

## v3.2.2 (2025-10-21)

### Improvements

**Strengthened using-superpowers skill against agent rationalization**
- Added EXTREMELY-IMPORTANT block w/ absolute language about mandatory skill checking
  - "If even 1% chance a skill applies, you MUST read it"
  - "You do not have a choice. You cannot rationalize your way out."
- Added MANDATORY FIRST RESPONSE PROTOCOL checklist
  - 5-step process agents must complete before any response
  - Explicit "responding without this = failure" consequence
- Added Common Rationalizations section w/ 8 specific evasion patterns
  - "This is just a simple question" → WRONG
  - "I can check files quickly" → WRONG
  - "Let me gather information first" → WRONG
  - Plus 5 more common patterns observed in agent behavior

Changes address observed agent behavior where they rationalize around skill usage despite clear instructions. Forceful language + pre-emptive counter-args aim to make non-compliance harder.

### Files Changed
- Updated: `skills/using-superpowers/SKILL.md` - Added three layers of enforcement to prevent skill-skipping rationalization

## v3.2.1 (2025-10-20)

### New Features

**Code reviewer agent now included in plugin**
- Added `superpowers:code-reviewer` agent to plugin `agents/` dir
- Agent provides systematic code review against plans + coding standards
- Previously required users to have personal agent config
- All skill refs updated to use namespaced `superpowers:code-reviewer`
- Fixes #55

### Files Changed
- New: `agents/code-reviewer.md` - Agent definition w/ review checklist + output format
- Updated: `skills/requesting-code-review/SKILL.md` - References to `superpowers:code-reviewer`
- Updated: `skills/subagent-driven-development/SKILL.md` - References to `superpowers:code-reviewer`

## v3.2.0 (2025-10-18)

### New Features

**Design documentation in brainstorming workflow**
- Added Phase 4: Design Documentation to brainstorming skill
- Design docs now written to `docs/plans/YYYY-MM-DD-<topic>-design.md` before impl
- Restores functionality from original brainstorming command lost during skill conversion
- Docs written before worktree setup + impl planning
- Tested w/ subagent → verify compliance under time pressure

### Breaking Changes

**Skill reference namespace standardization**
- All internal skill refs now use `superpowers:` namespace prefix
- Updated format: `superpowers:test-driven-development` (previously just `test-driven-development`)
- Affects all REQUIRED SUB-SKILL, RECOMMENDED SUB-SKILL, REQUIRED BACKGROUND refs
- Aligns w/ how skills invoked using Skill tool
- Files updated: brainstorming, executing-plans, subagent-driven-development, systematic-debugging, testing-skills-with-subagents, writing-plans, writing-skills

### Improvements

**Design vs implementation plan naming**
- Design docs use `-design.md` suffix → prevent filename collisions
- Impl plans continue using existing `YYYY-MM-DD-<feature-name>.md` format
- Both stored in `docs/plans/` dir w/ clear naming distinction

## v3.1.1 (2025-10-17)

### Bug Fixes

- **Fixed command syntax in README** (#44) - Updated all command refs to use correct namespaced syntax (`/superpowers:brainstorm` instead of `/brainstorm`). Plugin-provided commands automatically namespaced by Claude Code → avoid conflicts between plugins.

## v3.1.0 (2025-10-17)

### Breaking Changes

**Skill names standardized to lowercase**
- All skill frontmatter `name:` fields now use lowercase kebab-case matching dir names
- Examples: `brainstorming`, `test-driven-development`, `using-git-worktrees`
- All skill announcements + cross-refs updated to lowercase format
- Ensures consistent naming across dir names, frontmatter, docs

### New Features

**Enhanced brainstorming skill**
- Added Quick Reference table showing phases, activities, tool usage
- Added copyable workflow checklist for tracking progress
- Added decision flowchart for when to revisit earlier phases
- Added comprehensive AskUserQuestion tool guidance w/ concrete examples
- Added "Question Patterns" section explaining when to use structured vs open-ended questions
- Restructured Key Principles as scannable table

**Anthropic best practices integration**
- Added `skills/writing-skills/anthropic-best-practices.md` - Official Anthropic skill authoring guide
- Referenced in writing-skills SKILL.md for comprehensive guidance
- Provides patterns for progressive disclosure, workflows, evaluation

### Improvements

**Skill cross-reference clarity**
- All skill refs now use explicit requirement markers:
  - `**REQUIRED BACKGROUND:**` - Prerequisites you must understand
  - `**REQUIRED SUB-SKILL:**` - Skills that must be used in workflow
  - `**Complementary skills:**` - Optional but helpful related skills
- Removed old path format (`skills/collaboration/X` → just `X`)
- Updated Integration sections w/ categorized relationships (Required vs Complementary)
- Updated cross-reference docs w/ best practices

**Alignment w/ Anthropic best practices**
- Fixed description grammar + voice (fully third-person)
- Added Quick Reference tables for scanning
- Added workflow checklists Claude can copy + track
- Appropriate use of flowcharts for non-obvious decision points
- Improved scannable table formats
- All skills well under 500-line recommendation

### Bug Fixes

- **Re-added missing command redirects** - Restored `commands/brainstorm.md` + `commands/write-plan.md` accidentally removed in v3.0 migration
- Fixed `defense-in-depth` name mismatch (was `Defense-in-Depth-Validation`)
- Fixed `receiving-code-review` name mismatch (was `Code-Review-Reception`)
- Fixed `commands/brainstorm.md` reference to correct skill name
- Removed refs to non-existent related skills

### Documentation

**writing-skills improvements**
- Updated cross-referencing guidance w/ explicit requirement markers
- Added ref to Anthropic official best practices
- Improved examples showing proper skill ref format

## v3.0.1 (2025-10-16)

### Changes

We now use Anthropic first-party skills system!

## v2.0.2 (2025-10-12)

### Bug Fixes

- **Fixed false warning when local skills repo is ahead of upstream** - Init script was incorrectly warning "New skills available from upstream" when local repo had commits ahead of upstream. Logic now correctly distinguishes three git states: local behind (should update), local ahead (no warning), diverged (should warn).

## v2.0.1 (2025-10-12)

### Bug Fixes

- **Fixed session-start hook execution in plugin context** (#8, PR #9) - Hook was failing silently w/ "Plugin hook error" preventing skills context from loading. Fixed by:
  - Using `${BASH_SOURCE[0]:-$0}` fallback when BASH_SOURCE unbound in Claude Code execution context
  - Adding `|| true` to handle empty grep results gracefully when filtering status flags

---

# Superpowers v2.0.0 Release Notes

## Overview

Superpowers v2.0 makes skills more accessible, maintainable, community-driven through major architectural shift.

Headline change = **skills repository separation**: all skills, scripts, docs moved from plugin → dedicated repo ([obra/superpowers-skills](https://github.com/obra/superpowers-skills)). Transforms superpowers from monolithic plugin → lightweight shim managing local clone of skills repo. Skills auto-update on session start. Users fork + contribute improvements via standard git workflows. Skills lib versions independently from plugin.

Beyond infra, this release adds nine new skills focused on problem-solving, research, architecture. Rewrote core **using-skills** docs w/ imperative tone + clearer structure → easier for Claude to understand when + how to use skills. **find-skills** now outputs paths you can paste directly into Read tool → eliminate friction in skills discovery workflow.

Users experience seamless operation: plugin handles cloning, forking, updating automatically. Contributors find new architecture makes improving + sharing skills trivial. Release lays foundation for skills to evolve rapidly as community resource.

## Breaking Changes

### Skills Repository Separation

**Biggest change:** Skills no longer live in plugin. Moved to separate repo at [obra/superpowers-skills](https://github.com/obra/superpowers-skills).

**What this means:**

- **First install:** Plugin auto-clones skills to `~/.config/superpowers/skills/`
- **Forking:** During setup, offered option to fork skills repo (if `gh` installed)
- **Updates:** Skills auto-update on session start (fast-forward when possible)
- **Contributing:** Work on branches, commit locally, submit PRs to upstream
- **No more shadowing:** Old two-tier system (personal/core) replaced w/ single-repo branch workflow

**Migration:**

If existing install:
1. Old `~/.config/superpowers/.git` backed up to `~/.config/superpowers/.git.bak`
2. Old skills backed up to `~/.config/superpowers/skills.bak`
3. Fresh clone of obra/superpowers-skills created at `~/.config/superpowers/skills/`

### Removed Features

- **Personal superpowers overlay system** - Replaced w/ git branch workflow
- **setup-personal-superpowers hook** - Replaced by initialize-skills.sh

## New Features

### Skills Repository Infrastructure

**Automatic Clone & Setup** (`lib/initialize-skills.sh`)
- Clones obra/superpowers-skills on first run
- Offers fork creation if GitHub CLI installed
- Sets up upstream/origin remotes correctly
- Handles migration from old install

**Auto-Update**
- Fetches from tracking remote on every session start
- Auto-merges w/ fast-forward when possible
- Notifies when manual sync needed (branch diverged)
- Uses pulling-updates-from-skills-repository skill for manual sync

### New Skills

**Problem-Solving Skills** (`skills/problem-solving/`)
- **collision-zone-thinking** - Force unrelated concepts together for emergent insights
- **inversion-exercise** - Flip assumptions to reveal hidden constraints
- **meta-pattern-recognition** - Spot universal principles across domains
- **scale-game** - Test at extremes to expose fundamental truths
- **simplification-cascades** - Find insights that eliminate multiple components
- **when-stuck** - Dispatch to right problem-solving technique

**Research Skills** (`skills/research/`)
- **tracing-knowledge-lineages** - Understand how ideas evolved over time

**Architecture Skills** (`skills/architecture/`)
- **preserving-productive-tensions** - Keep multiple valid approaches instead of forcing premature resolution

### Skills Improvements

**using-skills (formerly getting-started)**
- Renamed from getting-started → using-skills
- Complete rewrite w/ imperative tone (v4.0.0)
- Front-loaded critical rules
- Added "Why" explanations for all workflows
- Always includes /SKILL.md suffix in refs
- Clearer distinction between rigid rules + flexible patterns

**writing-skills**
- Cross-referencing guidance moved from using-skills
- Added token efficiency section (word count targets)
- Improved CSO (Claude Search Optimization) guidance

**sharing-skills**
- Updated for new branch-and-PR workflow (v2.0.0)
- Removed personal/core split refs

**pulling-updates-from-skills-repository** (new)
- Complete workflow for syncing w/ upstream
- Replaces old "updating-skills" skill

### Tools Improvements

**find-skills**
- Now outputs full paths w/ /SKILL.md suffix
- Makes paths directly usable w/ Read tool
- Updated help text

**skill-run**
- Moved from scripts/ → skills/using-skills/
- Improved docs

### Plugin Infrastructure

**Session Start Hook**
- Now loads from skills repo location
- Shows full skills list at session start
- Prints skills location info
- Shows update status (updated successfully / behind upstream)
- Moved "skills behind" warning to end of output

**Environment Variables**
- `SUPERPOWERS_SKILLS_ROOT` set to `~/.config/superpowers/skills`
- Used consistently throughout all paths

## Bug Fixes

- Fixed duplicate upstream remote addition when forking
- Fixed find-skills double "skills/" prefix in output
- Removed obsolete setup-personal-superpowers call from session-start
- Fixed path refs throughout hooks + commands

## Documentation

### README
- Updated for new skills repo architecture
- Prominent link to superpowers-skills repo
- Updated auto-update description
- Fixed skill names + refs
- Updated Meta skills list

### Testing Documentation
- Added comprehensive testing checklist (`docs/TESTING-CHECKLIST.md`)
- Created local marketplace config for testing
- Documented manual testing scenarios

## Technical Details

### File Changes

**Added:**
- `lib/initialize-skills.sh` - Skills repo initialization + auto-update
- `docs/TESTING-CHECKLIST.md` - Manual testing scenarios
- `.claude-plugin/marketplace.json` - Local testing config

**Removed:**
- `skills/` directory (82 files) - Now in obra/superpowers-skills
- `scripts/` directory - Now in obra/superpowers-skills/skills/using-skills/
- `hooks/setup-personal-superpowers.sh` - Obsolete

**Modified:**
- `hooks/session-start.sh` - Use skills from ~/.config/superpowers/skills
- `commands/brainstorm.md` - Updated paths to SUPERPOWERS_SKILLS_ROOT
- `commands/write-plan.md` - Updated paths to SUPERPOWERS_SKILLS_ROOT
- `commands/execute-plan.md` - Updated paths to SUPERPOWERS_SKILLS_ROOT
- `README.md` - Complete rewrite for new architecture

### Commit History

Release includes:
- 20+ commits for skills repo separation
- PR #1: Amplifier-inspired problem-solving + research skills
- PR #2: Personal superpowers overlay system (later replaced)
- Multiple skill refinements + doc improvements

## Upgrade Instructions

### Fresh Install

```bash
# In Claude Code
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

Plugin handles everything automatically.

### Upgrading from v1.x

1. **Backup personal skills** (if any):
   ```bash
   cp -r ~/.config/superpowers/skills ~/superpowers-skills-backup
   ```

2. **Update plugin:**
   ```bash
   /plugin update superpowers
   ```

3. **On next session start:**
   - Old install backed up automatically
   - Fresh skills repo cloned
   - If GitHub CLI installed, offered option to fork

4. **Migrate personal skills** (if any):
   - Create branch in local skills repo
   - Copy personal skills from backup
   - Commit + push to fork
   - Consider contributing back via PR

## What's Next

### For Users

- Explore new problem-solving skills
- Try branch-based workflow for skill improvements
- Contribute skills back to community

### For Contributors

- Skills repo now at https://github.com/obra/superpowers-skills
- Fork → Branch → PR workflow
- See skills/meta/writing-skills/SKILL.md for TDD approach to docs

## Known Issues

None at this time.

## Credits

- Problem-solving skills inspired by Amplifier patterns
- Community contributions + feedback
- Extensive testing + iteration on skill effectiveness

---

**Full Changelog:** https://github.com/obra/superpowers/compare/dd013f6...main
**Skills Repository:** https://github.com/obra/superpowers-skills
**Issues:** https://github.com/obra/superpowers/issues