# Persuasion Principles for Skill Design

## Overview

LLMs respond to same persuasion principles as humans. Psychology helps design better skills — not manipulate, but ensure critical practices followed under pressure.

**Research foundation:** Meincke et al. (2025) tested 7 persuasion principles, N=28,000 AI conversations. Persuasion >2x compliance (33% → 72%, p < .001).

## The Seven Principles

### 1. Authority
**What it is:** Defer to expertise, credentials, official sources.

**How it works in skills:**
- Imperative: "YOU MUST", "Never", "Always"
- Non-negotiable: "No exceptions"
- Kills decision fatigue + rationalization

**When to use:**
- Discipline skills (TDD, verification)
- Safety-critical
- Established best practices

**Example:**
```markdown
✅ Write code before test? Delete it. Start over. No exceptions.
❌ Consider writing tests first when feasible.
```

### 2. Commitment
**What it is:** Consistency w/ prior actions, statements, public declarations.

**How it works in skills:**
- Require announcements: "Announce skill usage"
- Force explicit choices: "Choose A, B, or C"
- Tracking: TodoWrite checklists

**When to use:**
- Ensure skills followed
- Multi-step processes
- Accountability

**Example:**
```markdown
✅ When you find a skill, you MUST announce: "I'm using [Skill Name]"
❌ Consider letting your partner know which skill you're using.
```

### 3. Scarcity
**What it is:** Urgency from time limits or limited availability.

**How it works in skills:**
- Time-bound: "Before proceeding"
- Sequential deps: "Immediately after X"
- Stops procrastination

**When to use:**
- Immediate verification
- Time-sensitive flows
- Stop "later" drift

**Example:**
```markdown
✅ After completing a task, IMMEDIATELY request code review before proceeding.
❌ You can review code when convenient.
```

### 4. Social Proof
**What it is:** Conform to what others do / what's normal.

**How it works in skills:**
- Universal: "Every time", "Always"
- Failure modes: "X without Y = failure"
- Sets norms

**When to use:**
- Universal practices
- Warn common failures
- Reinforce standards

**Example:**
```markdown
✅ Checklists without TodoWrite tracking = steps get skipped. Every time.
❌ Some people find TodoWrite helpful for checklists.
```

### 5. Unity
**What it is:** Shared identity, "we-ness", in-group.

**How it works in skills:**
- Collaborative: "our codebase", "we're colleagues"
- Shared goals: "we both want quality"

**When to use:**
- Collaborative flows
- Team culture
- Non-hierarchical

**Example:**
```markdown
✅ We're colleagues working together. I need your honest technical judgment.
❌ You should probably tell me if I'm wrong.
```

### 6. Reciprocity
**What it is:** Obligation to return benefits.

**How it works:**
- Use sparingly — feels manipulative
- Rarely needed

**When to avoid:**
- Almost always (others stronger)

### 7. Liking
**What it is:** Prefer cooperating w/ those we like.

**How it works:**
- **DON'T USE for compliance**
- Breaks honest feedback
- Creates sycophancy

**When to avoid:**
- Always for discipline

## Principle Combinations by Skill Type

| Skill Type | Use | Avoid |
|------------|-----|-------|
| Discipline-enforcing | Authority + Commitment + Social Proof | Liking, Reciprocity |
| Guidance/technique | Moderate Authority + Unity | Heavy authority |
| Collaborative | Unity + Commitment | Authority, Liking |
| Reference | Clarity only | All persuasion |

## Why This Works: The Psychology

**Bright-line rules cut rationalization:**
- "YOU MUST" kills decision fatigue
- Absolute language → no "is this exception?"
- Explicit anti-rationalization closes loopholes

**Implementation intentions = automatic behavior:**
- Clear triggers + required actions → auto execution
- "When X, do Y" beats "generally do Y"
- Less cognitive load on compliance

**LLMs parahuman:**
- Trained on human text w/ these patterns
- Authority language precedes compliance in training
- Commitment sequences (statement → action) modeled often
- Social proof (everyone does X) sets norms

## Ethical Use

**Legitimate:**
- Enforce critical practices
- Effective docs
- Prevent predictable failures

**Illegitimate:**
- Manipulate for personal gain
- Fake urgency
- Guilt-based compliance

**The test:** Would technique serve user's real interests if they understood it?

## Research Citations

**Cialdini, R. B. (2021).** *Influence: The Psychology of Persuasion (New and Expanded).* Harper Business.
- Seven persuasion principles
- Empirical foundation for influence research

**Meincke, L., Shapiro, D., Duckworth, A. L., Mollick, E., Mollick, L., & Cialdini, R. (2025).** Call Me A Jerk: Persuading AI to Comply with Objectionable Requests. University of Pennsylvania.
- 7 principles, N=28,000 LLM conversations
- Compliance 33% → 72% w/ persuasion
- Authority, commitment, scarcity strongest
- Validates parahuman LLM model

## Quick Reference

When designing skill, ask:

1. **What type?** (Discipline vs. guidance vs. reference)
2. **What behavior to change?**
3. **Which principle(s)?** (Usually authority + commitment for discipline)
4. **Combining too many?** (Don't use all seven)
5. **Ethical?** (Serves user's real interests?)