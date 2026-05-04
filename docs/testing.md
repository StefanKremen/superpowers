# Testing Superpowers Skills

Doc describe how test Superpowers skills, esp. integration tests for complex skills like `subagent-driven-development`.

## Overview

Test skills with subagents/workflows/complex interactions → run real Claude Code sessions headless, verify via session transcripts.

## Test Structure

```
tests/
├── claude-code/
│   ├── test-helpers.sh                    # Shared test utilities
│   ├── test-subagent-driven-development-integration.sh
│   ├── analyze-token-usage.py             # Token analysis tool
│   └── run-skill-tests.sh                 # Test runner (if exists)
```

## Running Tests

### Integration Tests

Integration tests run real Claude Code sessions w/ real skills:

```bash
# Run the subagent-driven-development integration test
cd tests/claude-code
./test-subagent-driven-development-integration.sh
```

**Note:** Integration tests take 10-30 min — run real impl plans w/ multiple subagents.

### Requirements

- Must run from **superpowers plugin directory** (not temp dirs)
- Claude Code installed, available as `claude` cmd
- Local dev marketplace enabled: `"superpowers@superpowers-dev": true` in `~/.claude/settings.json`

## Integration Test: subagent-driven-development

### What It Tests

Test verify `subagent-driven-development` skill correct:

1. **Plan Loading**: Read plan once at start
2. **Full Task Text**: Give complete task descriptions to subagents (no file-read)
3. **Self-Review**: Subagents self-review before report
4. **Review Order**: Spec compliance review before code quality review
5. **Review Loops**: Use review loops when issues found
6. **Independent Verification**: Spec reviewer read code independently, no trust implementer reports

### How It Works

1. **Setup**: Make temp Node.js project w/ minimal impl plan
2. **Execution**: Run Claude Code headless w/ skill
3. **Verification**: Parse session transcript (`.jsonl` file) verify:
   - Skill tool invoked
   - Subagents dispatched (Task tool)
   - TodoWrite used for tracking
   - Impl files created
   - Tests pass
   - Git commits show proper workflow
4. **Token Analysis**: Show token usage breakdown per subagent

### Test Output

```
========================================
 Integration Test: subagent-driven-development
========================================

Test project: /tmp/tmp.xyz123

=== Verification Tests ===

Test 1: Skill tool invoked...
  [PASS] subagent-driven-development skill was invoked

Test 2: Subagents dispatched...
  [PASS] 7 subagents dispatched

Test 3: Task tracking...
  [PASS] TodoWrite used 5 time(s)

Test 6: Implementation verification...
  [PASS] src/math.js created
  [PASS] add function exists
  [PASS] multiply function exists
  [PASS] test/math.test.js created
  [PASS] Tests pass

Test 7: Git commit history...
  [PASS] Multiple commits created (3 total)

Test 8: No extra features added...
  [PASS] No extra features added

=========================================
 Token Usage Analysis
=========================================

Usage Breakdown:
----------------------------------------------------------------------------------------------------
Agent           Description                          Msgs      Input     Output      Cache     Cost
----------------------------------------------------------------------------------------------------
main            Main session (coordinator)             34         27      3,996  1,213,703 $   4.09
3380c209        implementing Task 1: Create Add Function     1          2        787     24,989 $   0.09
34b00fde        implementing Task 2: Create Multiply Function     1          4        644     25,114 $   0.09
3801a732        reviewing whether an implementation matches...   1          5        703     25,742 $   0.09
4c142934        doing a final code review...                    1          6        854     25,319 $   0.09
5f017a42        a code reviewer. Review Task 2...               1          6        504     22,949 $   0.08
a6b7fbe4        a code reviewer. Review Task 1...               1          6        515     22,534 $   0.08
f15837c0        reviewing whether an implementation matches...   1          6        416     22,485 $   0.07
----------------------------------------------------------------------------------------------------

TOTALS:
  Total messages:         41
  Input tokens:           62
  Output tokens:          8,419
  Cache creation tokens:  132,742
  Cache read tokens:      1,382,835

  Total input (incl cache): 1,515,639
  Total tokens:             1,524,058

  Estimated cost: $4.67
  (at $3/$15 per M tokens for input/output)

========================================
 Test Summary
========================================

STATUS: PASSED
```

## Token Analysis Tool

### Usage

Analyze token usage from any Claude Code session:

```bash
python3 tests/claude-code/analyze-token-usage.py ~/.claude/projects/<project-dir>/<session-id>.jsonl
```

### Finding Session Files

Session transcripts stored in `~/.claude/projects/` w/ working dir path encoded:

```bash
# Example for /Users/jesse/Documents/GitHub/superpowers/superpowers
SESSION_DIR="$HOME/.claude/projects/-Users-jesse-Documents-GitHub-superpowers-superpowers"

# Find recent sessions
ls -lt "$SESSION_DIR"/*.jsonl | head -5
```

### What It Shows

- **Main session usage**: Token usage by coordinator (you or main Claude instance)
- **Per-subagent breakdown**: Each Task invocation w/:
  - Agent ID
  - Description (from prompt)
  - Msg count
  - Input/output tokens
  - Cache usage
  - Est. cost
- **Totals**: Overall token usage + cost estimate

### Understanding the Output

- **High cache reads**: Good → prompt caching working
- **High input tokens on main**: Expected → coordinator has full context
- **Similar costs per subagent**: Expected → each get similar task complexity
- **Cost per task**: Typical $0.05-$0.15 per subagent, varies by task

## Troubleshooting

### Skills Not Loading

**Problem**: Skill not found in headless tests

**Solutions**:
1. Run FROM superpowers dir: `cd /path/to/superpowers && tests/...`
2. Check `~/.claude/settings.json` has `"superpowers@superpowers-dev": true` in `enabledPlugins`
3. Verify skill exists in `skills/` dir

### Permission Errors

**Problem**: Claude blocked from writing files / accessing dirs

**Solutions**:
1. Use `--permission-mode bypassPermissions` flag
2. Use `--add-dir /path/to/temp/dir` for test dir access
3. Check file perms on test dirs

### Test Timeouts

**Problem**: Test too long, times out

**Solutions**:
1. Bump timeout: `timeout 1800 claude ...` (30 min)
2. Check infinite loops in skill logic
3. Review subagent task complexity

### Session File Not Found

**Problem**: No session transcript after test run

**Solutions**:
1. Check correct project dir in `~/.claude/projects/`
2. Use `find ~/.claude/projects -name "*.jsonl" -mmin -60` for recent sessions
3. Verify test actually ran (errors in test output?)

## Writing New Integration Tests

### Template

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
source "$SCRIPT_DIR/test-helpers.sh"

# Create test project
TEST_PROJECT=$(create_test_project)
trap "cleanup_test_project $TEST_PROJECT" EXIT

# Set up test files...
cd "$TEST_PROJECT"

# Run Claude with skill
PROMPT="Your test prompt here"
cd "$SCRIPT_DIR/../.." && timeout 1800 claude -p "$PROMPT" \
  --allowed-tools=all \
  --add-dir "$TEST_PROJECT" \
  --permission-mode bypassPermissions \
  2>&1 | tee output.txt

# Find and analyze session
WORKING_DIR_ESCAPED=$(echo "$SCRIPT_DIR/../.." | sed 's/\\//-/g' | sed 's/^-//')
SESSION_DIR="$HOME/.claude/projects/$WORKING_DIR_ESCAPED"
SESSION_FILE=$(find "$SESSION_DIR" -name "*.jsonl" -type f -mmin -60 | sort -r | head -1)

# Verify behavior by parsing session transcript
if grep -q '"name":"Skill".*"skill":"your-skill-name"' "$SESSION_FILE"; then
    echo "[PASS] Skill was invoked"
fi

# Show token analysis
python3 "$SCRIPT_DIR/analyze-token-usage.py" "$SESSION_FILE"
```

### Best Practices

1. **Always cleanup**: Use trap → cleanup temp dirs
2. **Parse transcripts**: No grep user-facing output → parse `.jsonl` session file
3. **Grant permissions**: Use `--permission-mode bypassPermissions` + `--add-dir`
4. **Run from plugin dir**: Skills load only from superpowers dir
5. **Show token usage**: Always include token analysis for cost visibility
6. **Test real behavior**: Verify actual files created, tests pass, commits made

## Session Transcript Format

Session transcripts = JSONL (JSON Lines) files. Each line = JSON obj for message/tool result.

### Key Fields

```json
{
  "type": "assistant",
  "message": {
    "content": [...],
    "usage": {
      "input_tokens": 27,
      "output_tokens": 3996,
      "cache_read_input_tokens": 1213703
    }
  }
}
```

### Tool Results

```json
{
  "type": "user",
  "toolUseResult": {
    "agentId": "3380c209",
    "usage": {
      "input_tokens": 2,
      "output_tokens": 787,
      "cache_read_input_tokens": 24989
    },
    "prompt": "You are implementing Task 1...",
    "content": [{"type": "text", "text": "..."}]
  }
}
```

`agentId` field link to subagent sessions, `usage` field hold token usage for that subagent invocation.