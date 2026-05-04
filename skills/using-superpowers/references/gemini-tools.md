# Gemini CLI Tool Mapping

Skills use Claude Code tool names. Encounter in skill → use platform equivalent:

| Skill references | Gemini CLI equivalent |
|-----------------|----------------------|
| `Read` (file reading) | `read_file` |
| `Write` (file creation) | `write_file` |
| `Edit` (file editing) | `replace` |
| `Bash` (run commands) | `run_shell_command` |
| `Grep` (search file content) | `grep_search` |
| `Glob` (search files by name) | `glob` |
| `TodoWrite` (task tracking) | `write_todos` |
| `Skill` tool (invoke a skill) | `activate_skill` |
| `WebSearch` | `google_web_search` |
| `WebFetch` | `web_fetch` |
| `Task` tool (dispatch subagent) | No equivalent — Gemini CLI no subagents |

## No subagent support

Gemini CLI no equivalent to Claude Code `Task` tool. Skills relying on subagent dispatch (`subagent-driven-development`, `dispatching-parallel-agents`) fall back to single-session via `executing-plans`.

## Additional Gemini CLI tools

Available in Gemini CLI, no Claude Code equivalent:

| Tool | Purpose |
|------|---------|
| `list_directory` | List files + subdirs |
| `save_memory` | Persist facts to GEMINI.md across sessions |
| `ask_user` | Request structured input from user |
| `tracker_create_task` | Rich task mgmt (create, update, list, visualize) |
| `enter_plan_mode` / `exit_plan_mode` | Switch to read-only research mode before changes |