---
name: using-superpowers
description: Use when starting any conversation - establishes how to find and use skills, requiring Skill tool invocation before ANY response including clarifying questions
---

<SUBAGENT-STOP>
If dispatched as subagent for specific task, skip this skill.
</SUBAGENT-STOP>

<EXTREMELY-IMPORTANT>
If 1% chance skill applies, MUST invoke skill.

IF SKILL APPLIES, NO CHOICE. MUST USE.

Not negotiable. Not optional. No rationalizing out.
</EXTREMELY-IMPORTANT>

## Instruction Priority

Superpowers skills override default system prompt, but **user instructions always win**:

1. **User explicit instructions** (CLAUDE.md, GEMINI.md, AGENTS.md, direct requests) — top priority
2. **Superpowers skills** — override default system where conflict
3. **Default system prompt** — lowest

If CLAUDE.md/GEMINI.md/AGENTS.md say "no TDD" and skill says "always TDD," follow user. User in control.

## How to Access Skills

**Claude Code:** Use `Skill` tool. Invoking loads content—follow directly. Never Read skill files.

**Copilot CLI:** Use `skill` tool. Auto-discovered from installed plugins. `skill` tool works same as Claude Code `Skill` tool.

**Gemini CLI:** Skills activate via `activate_skill` tool. Gemini loads metadata at session start, activates full content on demand.

**Other envs:** Check platform docs for skill loading.

## Platform Adaptation

Skills use Claude Code tool names. Non-CC: see `references/copilot-tools.md` (Copilot CLI), `references/codex-tools.md` (Codex) for equivalents. Gemini CLI gets mapping auto-loaded via GEMINI.md.

# Using Skills

## The Rule

**Invoke relevant/requested skills BEFORE any response or action.** Even 1% chance → invoke to check. Wrong skill → don't use.

```dot
digraph skill_flow {
    "User message received" [shape=doublecircle];
    "About to EnterPlanMode?" [shape=doublecircle];
    "Already brainstormed?" [shape=diamond];
    "Invoke brainstorming skill" [shape=box];
    "Might any skill apply?" [shape=diamond];
    "Invoke Skill tool" [shape=box];
    "Announce: 'Using [skill] to [purpose]'" [shape=box];
    "Has checklist?" [shape=diamond];
    "Create TodoWrite todo per item" [shape=box];
    "Follow skill exactly" [shape=box];
    "Respond (including clarifications)" [shape=doublecircle];

    "About to EnterPlanMode?" -> "Already brainstormed?";
    "Already brainstormed?" -> "Invoke brainstorming skill" [label="no"];
    "Already brainstormed?" -> "Might any skill apply?" [label="yes"];
    "Invoke brainstorming skill" -> "Might any skill apply?";

    "User message received" -> "Might any skill apply?";
    "Might any skill apply?" -> "Invoke Skill tool" [label="yes, even 1%"];
    "Might any skill apply?" -> "Respond (including clarifications)" [label="definitely not"];
    "Invoke Skill tool" -> "Announce: 'Using [skill] to [purpose]'";
    "Announce: 'Using [skill] to [purpose]'" -> "Has checklist?";
    "Has checklist?" -> "Create TodoWrite todo per item" [label="yes"];
    "Has checklist?" -> "Follow skill exactly" [label="no"];
    "Create TodoWrite todo per item" -> "Follow skill exactly";
}
```

## Red Flags

Thoughts = STOP, rationalizing:

| Thought | Reality |
|---------|---------|
| "Just simple question" | Questions = tasks. Check skills. |
| "Need more context first" | Skill check BEFORE clarifying. |
| "Let me explore codebase first" | Skills tell HOW to explore. Check first. |
| "Can check git/files quickly" | Files lack convo context. Check skills. |
| "Let me gather info first" | Skills tell HOW to gather. |
| "Doesn't need formal skill" | Skill exists → use it. |
| "I remember this skill" | Skills evolve. Read current. |
| "Doesn't count as task" | Action = task. Check skills. |
| "Skill overkill" | Simple → complex. Use it. |
| "Just do one thing first" | Check BEFORE anything. |
| "Feels productive" | Undisciplined action wastes time. Skills prevent. |
| "Know what that means" | Knowing ≠ using. Invoke. |

## Skill Priority

Multiple skills apply, order:

1. **Process skills first** (brainstorming, debugging) — determine HOW to approach
2. **Implementation skills second** (frontend-design, mcp-builder) — guide execution

"Build X" → brainstorming first, then impl skills.
"Fix bug" → debugging first, then domain skills.

## Skill Types

**Rigid** (TDD, debugging): Follow exact. No adapting away discipline.

**Flexible** (patterns): Adapt principles to context.

Skill itself tells which.

## User Instructions

Instructions = WHAT, not HOW. "Add X" / "Fix Y" ≠ skip workflows.