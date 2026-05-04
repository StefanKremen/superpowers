# Claude Code Skills Tests

Automated tests for superpowers skills using Claude Code CLI.

## Overview

Verifies skills load + Claude follows. Invoke Claude Code headless (`claude -p`), check behavior.

## Requirements

- Claude Code CLI in PATH (`claude --version` works)
- Local superpowers plugin installed (see main README)

## Running Tests

### Run all fast tests (recommended):
```bash
./run-skill-tests.sh
```

### Run integration tests (slow, 10-30 minutes):
```bash
./run-skill-tests.sh --integration
```

### Run specific test:
```bash
./run-skill-tests.sh --test test-subagent-driven-development.sh
```

### Run with verbose output:
```bash
./run-skill-tests.sh --verbose
```

### Set custom timeout:
```bash
./run-skill-tests.sh --timeout 1800  # 30 minutes for integration tests
```

## Test Structure

### test-helpers.sh
Shared fns:
- `run_claude "prompt" [timeout]` - run Claude w/ prompt
- `assert_contains output pattern name` - pattern exist
- `assert_not_contains output pattern name` - pattern absent
- `assert_count output pattern count name` - exact count
- `assert_order output pattern_a pattern_b name` - order
- `create_test_project` - temp test dir
- `create_test_plan project_dir` - sample plan file

### Test Files

Each file:
1. Source `test-helpers.sh`
2. Run Claude Code w/ prompts
3. Assert expected behavior
4. Return 0 success, non-zero fail

## Example Test

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
source "$SCRIPT_DIR/test-helpers.sh"

echo "=== Test: My Skill ==="

# Ask Claude about the skill
output=$(run_claude "What does the my-skill skill do?" 30)

# Verify response
assert_contains "$output" "expected behavior" "Skill describes behavior"

echo "=== All tests passed ==="
```

## Current Tests

### Fast Tests (run by default)

#### test-subagent-driven-development.sh
Skill content + requirements (~2 min):
- Skill load + access
- Workflow order (spec compliance before code quality)
- Self-review docs
- Plan read efficiency docs
- Spec compliance reviewer skepticism docs
- Review loops docs
- Task context provision docs

### Integration Tests (use --integration flag)

#### test-subagent-driven-development-integration.sh
Full workflow run (~10-30 min):
- Real test project, Node.js setup
- Plan w/ 2 tasks
- Run plan via subagent-driven-development
- Verify behavior:
  - Plan read once at start (not per task)
  - Full task text in subagent prompts
  - Subagents self-review before report
  - Spec compliance review before code quality
  - Spec reviewer reads code independently
  - Working impl produced
  - Tests pass
  - Proper git commits

**What it tests:**
- Workflow works end-to-end
- Improvements applied
- Subagents follow skill
- Final code functional + tested

## Adding New Tests

1. New file: `test-<skill-name>.sh`
2. Source test-helpers.sh
3. Use `run_claude` + assertions
4. Add to list in `run-skill-tests.sh`
5. Executable: `chmod +x test-<skill-name>.sh`

## Timeout Considerations

- Default: 5 min/test
- Claude Code response time varies
- Tune via `--timeout`
- Focus tests, avoid long runs

## Debugging Failed Tests

`--verbose` → full Claude output:
```bash
./run-skill-tests.sh --verbose --test test-subagent-driven-development.sh
```

Otherwise only failures print.

## CI/CD Integration

CI:
```bash
# Run with explicit timeout for CI environments
./run-skill-tests.sh --timeout 900

# Exit code 0 = success, non-zero = failure
```

## Notes

- Tests verify skill *instructions*, not full exec
- Full workflow tests too slow
- Focus on key skill requirements
- Deterministic
- Skip impl details