# Code Quality Reviewer Prompt Template

Use template when dispatch code quality reviewer subagent.

**Purpose:** Verify impl well-built (clean, tested, maintainable)

**Only dispatch after spec compliance review passes.**

```
Task tool (superpowers:code-reviewer):
  Use template at requesting-code-review/code-reviewer.md

  WHAT_WAS_IMPLEMENTED: [from implementer's report]
  PLAN_OR_REQUIREMENTS: Task N from [plan-file]
  BASE_SHA: [commit before task]
  HEAD_SHA: [current commit]
  DESCRIPTION: [task summary]
```

**Beyond standard code quality, reviewer check:**
- Each file → one clear responsibility, well-defined interface?
- Units decomposed → understood + tested independently?
- Impl follow file structure from plan?
- Change create new files already large, or grow existing files big? (Skip pre-existing sizes — focus what this change added.)

**Code reviewer returns:** Strengths, Issues (Critical/Important/Minor), Assessment