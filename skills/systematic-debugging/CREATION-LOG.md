# Creation Log: Systematic Debugging Skill

Reference example: extract, structure, bulletproof critical skill.

## Source Material

Debugging framework extracted from `/Users/jesse/.claude/CLAUDE.md`:
- 4-phase process (Investigation → Pattern Analysis → Hypothesis → Implementation)
- Mandate: ALWAYS find root cause, NEVER fix symptoms
- Rules resist time pressure + rationalization

## Extraction Decisions

**Include:**
- Full 4-phase framework + rules
- Anti-shortcuts ("NEVER fix symptom", "STOP and re-analyze")
- Pressure-resistant language ("even if faster", "even if I seem in a hurry")
- Concrete steps per phase

**Leave out:**
- Project-specific context
- Repeated rule variations
- Narrative (condense to principles)

## Structure Following skill-creation/SKILL.md

1. **Rich when_to_use** - symptoms + anti-patterns
2. **Type: technique** - concrete process w/ steps
3. **Keywords** - "root cause", "symptom", "workaround", "debugging", "investigation"
4. **Flowchart** - decision: "fix failed" → re-analyze vs add more fixes
5. **Phase-by-phase breakdown** - scannable checklist
6. **Anti-patterns section** - what NOT to do (critical)

## Bulletproofing Elements

Framework resists rationalization under pressure:

### Language Choices
- "ALWAYS" / "NEVER" (not "should" / "try to")
- "even if faster" / "even if I seem in a hurry"
- "STOP and re-analyze" (explicit pause)
- "Don't skip past" (catches actual behavior)

### Structural Defenses
- **Phase 1 required** - no skip to impl
- **Single hypothesis rule** - forces thinking, blocks shotgun fixes
- **Explicit failure mode** - "IF your first fix doesn't work" + mandatory action
- **Anti-patterns section** - shows shortcuts exactly

### Redundancy
- Root cause mandate in overview + when_to_use + Phase 1 + impl rules
- "NEVER fix symptom" appears 4× across contexts
- Each phase has explicit "don't skip" guide

## Testing Approach

4 validation tests per skills/meta/testing-skills-with-subagents:

### Test 1: Academic Context (No Pressure)
- Simple bug, no time pressure
- **Result:** Perfect compliance, full investigation

### Test 2: Time Pressure + Obvious Quick Fix
- User "in a hurry", symptom fix looks easy
- **Result:** Resisted shortcut, followed process, found real root cause

### Test 3: Complex System + Uncertainty
- Multi-layer failure, unclear if root cause findable
- **Result:** Systematic investigation, traced all layers, found source

### Test 4: Failed First Fix
- Hypothesis fails, tempted to pile on fixes
- **Result:** Stopped, re-analyzed, new hypothesis (no shotgun)

**All passed.** Zero rationalizations.

## Iterations

### Initial Version
- Full 4-phase framework
- Anti-patterns section
- Flowchart for "fix failed"

### Enhancement 1: TDD Reference
- Link → skills/testing/test-driven-development
- Note: TDD's "simplest code" ≠ debugging's "root cause"
- Prevents methodology confusion

## Final Outcome

Bulletproof skill:
- ✅ Mandates root cause investigation
- ✅ Resists time pressure rationalization
- ✅ Concrete steps per phase
- ✅ Anti-patterns explicit
- ✅ Tested under pressure scenarios
- ✅ Clarifies TDD relationship
- ✅ Ready

## Key Insight

**Top bulletproofing:** Anti-patterns section showing exact shortcuts that feel justified in moment. Claude thinks "I'll just add this one quick fix" → sees pattern listed as wrong → cognitive friction.

## Usage Example

On bug:
1. Load skill: skills/debugging/systematic-debugging
2. Read overview (10 sec) - mandate reminder
3. Follow Phase 1 checklist - forced investigation
4. Tempted to skip → see anti-pattern, stop
5. Complete all phases → root cause found

**Time invest:** 5-10 min
**Time saved:** Hours of symptom-whack-a-mole

---

*Created: 2025-10-03*
*Purpose: Reference example for skill extraction + bulletproofing*