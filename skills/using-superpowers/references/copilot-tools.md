# Copilot CLI Tool Mapping

Skills use Claude Code tool names. Encounter in skill → use platform equivalent:

| Skill references | Copilot CLI equivalent |
|-----------------|----------------------|
| `Read` (file reading) | `view` |
| `Write` (file creation) | `create` |
| `Edit` (file editing) | `edit` |
| `Bash` (run commands) | `bash` |
| `Grep` (search file content) | `grep` |
| `Glob` (search files by name) | `glob` |
| `Skill` tool (invoke a skill) | `skill` |
| `WebFetch` | `web_fetch` |
| `Task` tool (dispatch subagent) | `task` (see [Agent types](#agent-types)) |
| Multiple `Task` calls (parallel) | Multiple `task` calls |
| Task status/output | `read_agent`, `list_agents` |
| `TodoWrite` (task tracking) | `sql` w/ built-in `todos` table |
| `WebSearch` | None — use `web_fetch` w/ search engine URL |
| `EnterPlanMode` / `ExitPlanMode` | None — stay in main session |

## Agent types

Copilot CLI `task` tool takes `agent_type` param:

| Claude Code agent | Copilot CLI equivalent |
|-------------------|----------------------|
| `general-purpose` | `"general-purpose"` |
| `Explore` | `"explore"` |
| Named plugin agents (e.g. `superpowers:code-reviewer`) | Auto-discovered from installed plugins |

## Async shell sessions

Copilot CLI supports persistent async shell sessions. No Claude Code equivalent:

| Tool | Purpose |
|------|---------|
| `bash` with `async: true` | Start long-running cmd in background |
| `write_bash` | Send input → running async session |
| `read_bash` | Read output from async session |
| `stop_bash` | Kill async session |
| `list_bash` | List active shell sessions |

## Additional Copilot CLI tools

| Tool | Purpose |
|------|---------|
| `store_memory` | Persist codebase facts for future sessions |
| `report_intent` | Update UI status line w/ current intent |
| `sql` | Query session SQLite DB (todos, metadata) |
| `fetch_copilot_cli_documentation` | Look up Copilot CLI docs |
| GitHub MCP tools (`github-mcp-server-*`) | Native GitHub API (issues, PRs, code search) |