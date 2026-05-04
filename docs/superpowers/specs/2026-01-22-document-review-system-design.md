# Document Review System Design

## Overview

Add two review stages to superpowers workflow:

1. **Spec Document Review** - After brainstorming, before writing-plans
2. **Plan Document Review** - After writing-plans, before implementation

Both follow iterative loop pattern from impl reviews.

## Spec Document Reviewer

**Purpose:** Verify spec complete, consistent, ready for impl planning.

**Location:** `skills/brainstorming/spec-document-reviewer-prompt.md`

**What it checks for:**

| Category | What to Look For |
|----------|------------------|
| Completeness | TODOs, placeholders, "TBD", incomplete sections |
| Coverage | Missing error handling, edge cases, integration points |
| Consistency | Internal contradictions, conflicting requirements |
| Clarity | Ambiguous requirements |
| YAGNI | Unrequested features, over-engineering |

**Output format:**
```
## Spec Review

**Status:** Approved | Issues Found

**Issues (if any):**
- [Section X]: [issue] - [why it matters]

**Recommendations (advisory):**
- [suggestions that don't block approval]
```

**Review loop:** Issues → brainstorming agent fix → re-review → repeat til approved.

**Dispatch mechanism:** Task tool, `subagent_type: general-purpose`. Reviewer prompt template = full prompt. Brainstorming skill controller dispatches reviewer.

## Plan Document Reviewer

**Purpose:** Verify plan complete, matches spec, proper task decomposition.

**Location:** `skills/writing-plans/plan-document-reviewer-prompt.md`

**What it checks for:**

| Category | What to Look For |
|----------|------------------|
| Completeness | TODOs, placeholders, incomplete tasks |
| Spec Alignment | Plan covers spec requirements, no scope creep |
| Task Decomposition | Tasks atomic, clear boundaries |
| Task Syntax | Checkbox syntax on tasks and steps |
| Chunk Size | Each chunk under 1000 lines |

**Chunk definition:** Chunk = logical task grouping in plan doc, delimited by `## Chunk N: <name>` headings. Writing-plans skill creates boundaries by logical phases (e.g. "Foundation", "Core Features", "Integration"). Each chunk self-contained → reviewable independently.

**Spec alignment verification:** Reviewer gets both:
1. Plan doc (or current chunk)
2. Path to spec doc for reference

Reviewer reads both → compares requirements coverage.

**Output format:** Same as spec reviewer, scoped to current chunk.

**Review process (chunk-by-chunk):**
1. Writing-plans creates chunk N
2. Controller dispatches plan-document-reviewer w/ chunk N content + spec path
3. Reviewer reads chunk + spec, returns verdict
4. Issues → writing-plans fix chunk N, goto 2
5. Approved → chunk N+1
6. Repeat til all chunks approved

**Dispatch mechanism:** Same as spec reviewer — Task tool, `subagent_type: general-purpose`.

## Updated Workflow

```
brainstorming -> spec -> SPEC REVIEW LOOP -> writing-plans -> plan -> PLAN REVIEW LOOP -> implementation
```

**Spec Review Loop:**
1. Spec complete
2. Dispatch reviewer
3. Issues → fix → goto 2
4. Approved → proceed

**Plan Review Loop:**
1. Chunk N complete
2. Dispatch reviewer for chunk N
3. Issues → fix → goto 2
4. Approved → next chunk or impl

## Markdown Task Syntax

Tasks + steps use checkbox syntax:

```markdown
- [ ] ### Task 1: Name

- [ ] **Step 1:** Description
  - File: path
  - Command: cmd
```

## Error Handling

**Review loop termination:**
- No hard iter limit — loop til reviewer approves
- Loop > 5 iter → controller surface to human for guidance
- Human choice: continue, approve w/ known issues, abort

**Disagreement handling:**
- Reviewers advisory — flag issues, don't block
- Agent thinks feedback wrong → explain why in fix
- Disagreement persist > 3 iter on same issue → surface to human

**Malformed reviewer output:**
- Controller validate reviewer output has required fields (Status, Issues if applicable)
- Malformed → re-dispatch w/ note about expected format
- After 2 malformed responses → surface to human

## Files to Change

**New files:**
- `skills/brainstorming/spec-document-reviewer-prompt.md`
- `skills/writing-plans/plan-document-reviewer-prompt.md`

**Modified files:**
- `skills/brainstorming/SKILL.md` - add review loop after spec written
- `skills/writing-plans/SKILL.md` - add chunk-by-chunk review loop, update task syntax examples