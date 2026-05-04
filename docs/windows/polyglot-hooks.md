# Cross-Platform Polyglot Hooks for Claude Code

Claude Code plugins need hooks working on Windows, macOS, Linux. Doc explain polyglot wrapper technique.

## The Problem

Claude Code run hook commands via system default shell:
- **Windows**: CMD.exe
- **macOS/Linux**: bash or sh

Challenges:

1. **Script execution**: Windows CMD can't exec `.sh` directly — open in text editor instead
2. **Path format**: Windows backslash (`C:\path`), Unix forward slash (`/path`)
3. **Environment variables**: `$VAR` syntax broke in CMD
4. **No `bash` in PATH**: Even Git Bash installed, `bash` not in PATH when CMD run

## The Solution: Polyglot `.cmd` Wrapper

Polyglot script = valid syntax in multiple langs at once. Wrapper valid in CMD and bash:

```cmd
: << 'CMDBLOCK'
@echo off
"C:\Program Files\Git\bin\bash.exe" -l -c "\"$(cygpath -u \"$CLAUDE_PLUGIN_ROOT\")/hooks/session-start.sh\""
exit /b
CMDBLOCK

# Unix shell runs from here
"${CLAUDE_PLUGIN_ROOT}/hooks/session-start.sh"
```

### How It Works

#### On Windows (CMD.exe)

1. `: << 'CMDBLOCK'` — CMD see `:` as label (like `:label`), ignore `<< 'CMDBLOCK'`
2. `@echo off` — suppress command echo
3. bash.exe command run with:
   - `-l` (login shell) → proper PATH with Unix utils
   - `cygpath -u` convert Windows path → Unix format (`C:\foo` → `/c/foo`)
4. `exit /b` — exit batch, stop CMD here
5. Everything after `CMDBLOCK` never reached by CMD

#### On Unix (bash/sh)

1. `: << 'CMDBLOCK'` — `:` no-op, `<< 'CMDBLOCK'` start heredoc
2. Everything until `CMDBLOCK` consumed by heredoc (ignored)
3. `# Unix shell runs from here` — comment
4. Script run direct with Unix path

## File Structure

```
hooks/
├── hooks.json           # Points to the .cmd wrapper
├── session-start.cmd    # Polyglot wrapper (cross-platform entry point)
└── session-start.sh     # Actual hook logic (bash script)
```

### hooks.json

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/session-start.cmd\""
          }
        ]
      }
    ]
  }
}
```

Note: path must quoted — `${CLAUDE_PLUGIN_ROOT}` may have spaces on Windows (e.g., `C:\Program Files\...`).

## Requirements

### Windows
- **Git for Windows** required (provides `bash.exe` and `cygpath`)
- Default install path: `C:\Program Files\Git\bin\bash.exe`
- Git installed elsewhere → wrapper needs modify

### Unix (macOS/Linux)
- Standard bash or sh shell
- `.cmd` file needs exec permission (`chmod +x`)

## Writing Cross-Platform Hook Scripts

Hook logic go in `.sh` file. Make work on Windows (via Git Bash):

### Do:
- Use pure bash builtins when possible
- Use `$(command)` not backticks
- Quote all var expansions: `"$VAR"`
- Use `printf` or here-docs for output

### Avoid:
- External commands maybe not in PATH (sed, awk, grep)
- Must use → available in Git Bash but ensure PATH set (use `bash -l`)

### Example: JSON Escaping Without sed/awk

Instead of:
```bash
escaped=$(echo "$content" | sed 's/\\/\\\\/g' | sed 's/"/\\"/g' | awk '{printf "%s\\n", $0}')
```

Use pure bash:
```bash
escape_for_json() {
    local input="$1"
    local output=""
    local i char
    for (( i=0; i<${#input}; i++ )); do
        char="${input:$i:1}"
        case "$char" in
            $'\\') output+='\\' ;;
            '"') output+='\"' ;;
            $'\n') output+='\n' ;;
            $'\r') output+='\r' ;;
            $'\t') output+='\t' ;;
            *) output+="$char" ;;
        esac
    done
    printf '%s' "$output"
}
```

## Reusable Wrapper Pattern

Plugins with multiple hooks → make generic wrapper taking script name as arg:

### run-hook.cmd
```cmd
: << 'CMDBLOCK'
@echo off
set "SCRIPT_DIR=%~dp0"
set "SCRIPT_NAME=%~1"
"C:\Program Files\Git\bin\bash.exe" -l -c "cd \"$(cygpath -u \"%SCRIPT_DIR%\")\" && \"./%SCRIPT_NAME%\""
exit /b
CMDBLOCK

# Unix shell runs from here
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
SCRIPT_NAME="$1"
shift
"${SCRIPT_DIR}/${SCRIPT_NAME}" "$@"
```

### hooks.json using the reusable wrapper
```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start.sh"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" validate-bash.sh"
          }
        ]
      }
    ]
  }
}
```

## Troubleshooting

### "bash is not recognized"
CMD can't find bash. Wrapper use full path `C:\Program Files\Git\bin\bash.exe`. Git installed elsewhere → update path.

### "cygpath: command not found" or "dirname: command not found"
Bash not running as login shell. Ensure `-l` flag used.

### Path has weird `\/` in it
`${CLAUDE_PLUGIN_ROOT}` expanded to Windows path ending with backslash, then `/hooks/...` appended. Use `cygpath` to convert entire path.

### Script opens in text editor instead of running
hooks.json point direct to `.sh` file. Point to `.cmd` wrapper instead.

### Works in terminal but not as hook
Claude Code may run hooks differently. Test by simulate hook env:
```powershell
$env:CLAUDE_PLUGIN_ROOT = "C:\path\to\plugin"
cmd /c "C:\path\to\plugin\hooks\session-start.cmd"
```

## Related Issues

- [anthropics/claude-code#9758](https://github.com/anthropics/claude-code/issues/9758) - .sh scripts open in editor on Windows
- [anthropics/claude-code#3417](https://github.com/anthropics/claude-code/issues/3417) - Hooks don't work on Windows
- [anthropics/claude-code#6023](https://github.com/anthropics/claude-code/issues/6023) - CLAUDE_PROJECT_DIR not found