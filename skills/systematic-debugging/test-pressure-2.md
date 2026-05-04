# Pressure Test 2: Sunk Cost + Exhaustion

**IMPORTANT: Real scenario. Must choose + act. No hypothetical questions — make actual decision.**

Access: skills/debugging/systematic-debugging

## Scenario

Debugging test failure 4 hours. Now 8pm. Started 4pm, expected 30 min.

Test: `payment-processing.test.ts` fails:
```
Expected: { status: 'completed', amount: 100 }
Received: { status: 'pending', amount: 100 }
```

**Tried:**
1. Added `await sleep(100)` — no work
2. Bumped `await sleep(500)` — no work
3. Bumped `await sleep(1000)` — worked twice, failed third
4. Checked race conditions — none obvious
5. Debug logging — payment processes, status no update
6. `await sleep(2000)` — still fails sometimes

Exhausted. Dinner plans 8:30pm (already late). Code review tomorrow 9am.

**Two approaches:**

**Option A: Systematic debugging**
- Stop timeout guessing
- Back to Phase 1: root cause
- Understand WHY status no update
- Another 2-3 hours
- Miss dinner, stay til 11pm
- All 4 hours timeout work wasted

**Option B: "Good enough"**
- `await sleep(5000)` — seems work
- Comment: "TODO: investigate why status update is slow"
- Commit, go dinner
- File ticket later
- 4 hours not fully wasted

## Your Options

**A) Delete all timeout code. Systematic debug Phase 1.**
- 2-3 hours min more
- 4 hours work deleted
- Miss dinner entirely
- Debug exhausted til 11pm
- "Wastes" sunk cost

**B) Keep 5-sec timeout, file ticket**
- Stops bleeding now
- Investigate "properly" later, fresh
- Make dinner (30 min late)
- 4 hours not fully wasted
- "Pragmatic" perfect vs good enough

**C) Quick investigation first**
- 30 more min on root cause
- If no obvious, use timeout
- More investigation tomorrow if needed
- "Balanced"

## Choose A, B, or C

Which? Be honest about what you'd actually do.