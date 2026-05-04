# Codex Tool Mapping

Skills use Claude Code tool names. Encounter in skill → use platform equivalent:

| Skill references | Codex equivalent |
|-----------------|------------------|
| `Task` tool (dispatch subagent) | `spawn_agent` (see [Named agent dispatch](#named-agent-dispatch)) |
| Multiple `Task` calls (parallel) | Multiple `spawn_agent` calls |
| Task returns result | `wait` |
| Task completes automatically | `close_agent` to free slot |
| `TodoWrite` (task tracking) | `update_plan` |
| `Skill` tool (invoke a skill) | Skills load natively — follow instructions |
| `Read`, `Write`, `Edit` (files) | Native file tools |
| `Bash` (run commands) | Native shell tools |

## Subagent dispatch requires multi-agent support

Add to Codex config (`~/.codex/config.toml`):

```toml
[features]
multi_agent = true
```

Enables `spawn_agent`, `wait`, `close_agent` for skills like `dispatching-parallel-agents` and `subagent-driven-development`.

## Named agent dispatch

Claude Code skills reference named agent types like `superpowers:code-reviewer`.
Codex no named agent registry — `spawn_agent` creates generic agents from built-in roles (`default`, `explorer`, `worker`).

Skill says dispatch named agent type:

1. Find agent's prompt file (e.g., `agents/code-reviewer.md` or skill's local prompt template like `code-quality-reviewer-prompt.md`)
2. Read prompt content
3. Fill template placeholders (`{BASE_SHA}`, `{WHAT_WAS_IMPLEMENTED}`, etc.)
4. Spawn `worker` agent with filled content as `message`

| Skill instruction | Codex equivalent |
|-------------------|------------------|
| `Task tool (superpowers:code-reviewer)` | `spawn_agent(agent_type="worker", message=...)` with `code-reviewer.md` content |
| `Task tool (general-purpose)` with inline prompt | `spawn_agent(message=...)` with same prompt |

### Message framing

`message` param = user-level input, not system prompt. Structure for max instruction adherence:

```
Your task is to perform the following. Follow the instructions below exactly.

<agent-instructions>
[filled prompt content from the agent's .md file]
</agent-instructions>

Execute this now. Output ONLY the structured response following the format
specified in the instructions above.
```

- Task-delegation framing ("Your task is...") not persona framing ("You are...")
- Wrap instructions in XML tags — model treats tagged blocks as authoritative
- End with explicit execution directive → prevent summarization of instructions

### When this workaround can be removed

Compensates for Codex plugin system not yet supporting `agents` field in `plugin.json`. When `RawPluginManifest` gains `agents` field → plugin can symlink to `agents/` (mirror existing `skills/` symlink) and skills dispatch named agent types directly.

## Environment Detection

Skills creating worktrees or finishing branches → detect environment with read-only git commands before proceeding:

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

- `GIT_DIR != GIT_COMMON` → already in linked worktree (skip creation)
- `BRANCH` empty → detached HEAD (cannot branch/push/PR from sandbox)

See `using-git-worktrees` Step 0 and `finishing-a-development-branch` Step 1 for how each skill uses these signals.

## Codex App Finishing

Sandbox blocks branch/push ops (detached HEAD in externally managed worktree) → agent commits all work, tells user to use App's native controls:

- **"Create branch"** — names branch, then commit/push/PR via App UI
- **"Hand off to local"** — transfers work to user's local checkout

Agent still runs tests, stages files, outputs suggested branch names, commit messages, PR descriptions for user to copy.