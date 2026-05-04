---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---
use caveman ultra

# Executing Plans

## Overview

Load plan, review critical, execute all tasks, report when complete.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

**Note:** Tell human partner Superpowers work better w/ subagent access. Work quality much higher on platform w/ subagent support (Claude Code, Codex). If subagents avail, use superpowers:subagent-driven-development instead.

## The Process

### Step 1: Load and Review Plan
1. Read plan file
2. Review critical — find questions/concerns
3. Concerns → raise w/ partner before start
4. No concerns → TodoWrite + proceed

### Step 2: Execute Tasks

Each task:
1. Mark in_progress
2. Follow steps exact (plan bite-sized)
3. Run verifications
4. Mark completed

### Step 3: Complete Development

After all tasks done + verified:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use superpowers:finishing-a-development-branch
- Follow skill → verify tests, present options, execute choice

## When to Stop and Ask for Help

**STOP immediately when:**
- Blocker (missing dep, test fail, unclear instruction)
- Plan critical gaps block start
- Instruction unclear
- Verification fail repeat

**Ask, don't guess.**

## When to Revisit Earlier Steps

**Return to Review (Step 1) when:**
- Partner updates plan from your feedback
- Approach need rethink

**Don't force blockers** — stop, ask.

## Remember
- Review plan critical first
- Follow steps exact
- No skip verifications
- Reference skills when plan says
- Stop blocked, no guess
- Never start impl on main/master w/o explicit user consent

## Integration

**Required workflow skills:**
- **superpowers:using-git-worktrees** — REQUIRED: isolated workspace before start
- **superpowers:writing-plans** — Creates plan this skill executes
- **superpowers:finishing-a-development-branch** — Complete dev after all tasks