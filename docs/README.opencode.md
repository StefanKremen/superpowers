# Superpowers for OpenCode

Guide for Superpowers with [OpenCode.ai](https://opencode.ai).

## Installation

Add superpowers to `plugin` array in `opencode.json` (global or project):

```json
{
  "plugin": ["superpowers@git+https://github.com/obra/superpowers.git"]
}
```

Restart OpenCode. Plugin auto-install via Bun, register all skills.

Verify: ask "Tell me about your superpowers"

### Migrating from the old symlink-based install

Old install used `git clone` + symlinks. Remove old setup:

```bash
# Remove old symlinks
rm -f ~/.config/opencode/plugins/superpowers.js
rm -rf ~/.config/opencode/skills/superpowers

# Optionally remove the cloned repo
rm -rf ~/.config/opencode/superpowers

# Remove skills.paths from opencode.json if you added one for superpowers
```

Then install steps above.

## Usage

### Finding Skills

OpenCode native `skill` tool list skills:

```
use skill tool to list skills
```

### Loading a Skill

```
use skill tool to load superpowers/brainstorming
```

### Personal Skills

Make own skills in `~/.config/opencode/skills/`:

```bash
mkdir -p ~/.config/opencode/skills/my-skill
```

Create `~/.config/opencode/skills/my-skill/SKILL.md`:

```markdown
---
name: my-skill
description: Use when [condition] - [what it does]
---

# My Skill

[Your skill content here]
```

### Project Skills

Project-specific skills in `.opencode/skills/` inside project.

**Skill Priority:** Project > Personal > Superpowers

## Updating

Superpowers auto-update on OpenCode restart. Plugin re-install from git each launch.

Pin version: use branch or tag:

```json
{
  "plugin": ["superpowers@git+https://github.com/obra/superpowers.git#v5.0.3"]
}
```

## How It Works

Plugin do two things:

1. **Inject bootstrap context** via `experimental.chat.system.transform` hook → superpowers awareness every conversation.
2. **Register skills dir** via `config` hook → OpenCode discover superpowers skills, no symlinks or manual config.

### Tool Mapping

Claude Code skills auto-adapt for OpenCode:

- `TodoWrite` → `todowrite`
- `Task` with subagents → OpenCode `@mention` system
- `Skill` tool → OpenCode native `skill` tool
- File ops → Native OpenCode tools

## Troubleshooting

### Plugin not loading

1. Check OpenCode logs: `opencode run --print-logs "hello" 2>&1 | grep -i superpowers`
2. Verify plugin line in `opencode.json` correct
3. Run recent OpenCode version

### Skills not found

1. Use OpenCode `skill` tool to list skills
2. Check plugin loads (see above)
3. Each skill need `SKILL.md` with valid YAML frontmatter

### Bootstrap not appearing

1. Check OpenCode version support `experimental.chat.system.transform` hook
2. Restart OpenCode after config change

## Getting Help

- Report issues: https://github.com/obra/superpowers/issues
- Main docs: https://github.com/obra/superpowers
- OpenCode docs: https://opencode.ai/docs/