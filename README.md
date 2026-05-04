# Superpowers

Superpowers = full software dev methodology for coding agents. Composable skills + initial instructions ensure agent use them.

## How it works

Start moment agent fire up. Sees you build something → *doesn't* jump to code. Steps back, asks what really trying do.

Once spec teased from convo, shows in chunks short enough digest.

After design signoff, agent builds impl plan clear enough for enthusiastic junior engineer with poor taste, no judgement, no project context, aversion to testing. Emphasize true red/green TDD, YAGNI (You Aren't Gonna Need It), DRY.

Next, you say "go" → launch *subagent-driven-development*. Agents work each task, inspect/review work, continue. Claude often work autonomous couple hours, no plan deviation.

More to it, but core. Skills auto-trigger → nothing special needed. Agent just have Superpowers.


## Sponsorship

Superpowers help you make money + so inclined → appreciate [sponsoring my opensource work](https://github.com/sponsors/obra).

Thanks!

- Jesse


## Installation

**Note:** Install differs by platform.

### Claude Code Official Marketplace

Superpowers on [official Claude plugin marketplace](https://claude.com/plugins/superpowers)

Install from Anthropic official marketplace:

```bash
/plugin install superpowers@claude-plugins-official
```

### Claude Code (Superpowers Marketplace)

Superpowers marketplace ships Superpowers + related plugins for Claude Code.

In Claude Code, register marketplace first:

```bash
/plugin marketplace add obra/superpowers-marketplace
```

Install plugin from this marketplace:

```bash
/plugin install superpowers@superpowers-marketplace
```

### OpenAI Codex CLI

- Open plugin search

```bash
/plugins
```

Search Superpowers

```bash
superpowers
```

Select `Install Plugin`

### OpenAI Codex App

- Codex app → click Plugins in sidebar.
- See `Superpowers` in Coding section.
- Click `+` next to Superpowers, follow prompts.


### Cursor (via Plugin Marketplace)

Cursor Agent chat, install from marketplace:

```text
/add-plugin superpowers
```

or search "superpowers" in plugin marketplace.

### OpenCode

Tell OpenCode:

```
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md
```

**Detailed docs:** [docs/README.opencode.md](docs/README.opencode.md)

### GitHub Copilot CLI

```bash
copilot plugin marketplace add obra/superpowers-marketplace
copilot plugin install superpowers@superpowers-marketplace
```

### Gemini CLI

```bash
gemini extensions install https://github.com/obra/superpowers
```

Update:

```bash
gemini extensions update superpowers
```

## The Basic Workflow

1. **brainstorming** - Activate before writing code. Refines rough ideas via questions, explores alternatives, presents design in sections for validation. Saves design doc.

2. **using-git-worktrees** - Activate after design approval. Creates isolated workspace on new branch, runs project setup, verifies clean test baseline.

3. **writing-plans** - Activate with approved design. Breaks work into bite-sized tasks (2-5 min each). Every task has exact file paths, complete code, verification steps.

4. **subagent-driven-development** or **executing-plans** - Activate with plan. Dispatches fresh subagent per task with two-stage review (spec compliance, then code quality), or executes in batches with human checkpoints.

5. **test-driven-development** - Activate during impl. Enforces RED-GREEN-REFACTOR: write failing test, watch fail, write minimal code, watch pass, commit. Deletes code written before tests.

6. **requesting-code-review** - Activate between tasks. Reviews against plan, reports issues by severity. Critical issues block progress.

7. **finishing-a-development-branch** - Activate when tasks complete. Verifies tests, presents options (merge/PR/keep/discard), cleans up worktree.

**Agent checks for relevant skills before any task.** Mandatory workflows, not suggestions.

## What's Inside

### Skills Library

**Testing**
- **test-driven-development** - RED-GREEN-REFACTOR cycle (includes testing anti-patterns reference)

**Debugging**
- **systematic-debugging** - 4-phase root cause process (includes root-cause-tracing, defense-in-depth, condition-based-waiting techniques)
- **verification-before-completion** - Ensure actually fixed

**Collaboration**
- **brainstorming** - Socratic design refinement
- **writing-plans** - Detailed impl plans
- **executing-plans** - Batch execution with checkpoints
- **dispatching-parallel-agents** - Concurrent subagent workflows
- **requesting-code-review** - Pre-review checklist
- **receiving-code-review** - Respond to feedback
- **using-git-worktrees** - Parallel dev branches
- **finishing-a-development-branch** - Merge/PR decision workflow
- **subagent-driven-development** - Fast iteration with two-stage review (spec compliance, then code quality)

**Meta**
- **writing-skills** - Create new skills following best practices (includes testing methodology)
- **using-superpowers** - Intro to skills system

## Philosophy

- **Test-Driven Development** - Write tests first, always
- **Systematic over ad-hoc** - Process over guessing
- **Complexity reduction** - Simplicity primary goal
- **Evidence over claims** - Verify before declaring success

Read [the original release announcement](https://blog.fsck.com/2025/10/09/superpowers/).

## Contributing

General contribution process below. Note: don't generally accept new skill contributions; any skill updates must work across all supported coding agents.

1. Fork repo
2. Switch to 'dev' branch
3. Create branch for work
4. Follow `writing-skills` skill for creating/testing new + modified skills
5. Submit PR, fill in PR template.

See `skills/writing-skills/SKILL.md` for full guide.

## Updating

Superpowers updates coding-agent dependent, often automatic.

## License

MIT License - see LICENSE file

## Community

Superpowers built by [Jesse Vincent](https://blog.fsck.com) + folks at [Prime Radiant](https://primeradiant.com).

- **Discord**: [Join us](https://discord.gg/35wsABTejz) for community support, questions, sharing what you build with Superpowers
- **Issues**: https://github.com/obra/superpowers/issues
- **Release announcements**: [Sign up](https://primeradiant.com/superpowers/) for new version notifications