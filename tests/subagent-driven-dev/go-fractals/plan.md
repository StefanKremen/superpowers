# Go Fractals CLI - Implementation Plan

Execute plan using `superpowers:subagent-driven-development` skill.

## Context

CLI tool generate ASCII fractals. See `design.md` for full spec.

## Tasks

### Task 1: Project Setup

Create Go module + dir structure.

**Do:**
- Init `go.mod` with module `github.com/superpowers-test/fractals`
- Create dirs: `cmd/fractals/`, `internal/sierpinski/`, `internal/mandelbrot/`, `internal/cli/`
- Minimal `cmd/fractals/main.go` prints "fractals cli"
- Add `github.com/spf13/cobra` dep

**Verify:**
- `go build ./cmd/fractals` ok
- `./fractals` prints "fractals cli"

---

### Task 2: CLI Framework with Help

Cobra root cmd + help output.

**Do:**
- Create `internal/cli/root.go` with root cmd
- Help text show subcmds
- Wire root cmd into `main.go`

**Verify:**
- `./fractals --help` show usage with "sierpinski" + "mandelbrot" listed
- `./fractals` (no args) show help

---

### Task 3: Sierpinski Algorithm

Impl Sierpinski triangle gen.

**Do:**
- Create `internal/sierpinski/sierpinski.go`
- Impl `Generate(size, depth int, char rune) []string` → triangle lines
- Recursive midpoint subdivision
- Create `internal/sierpinski/sierpinski_test.go` tests:
  - Small triangle (size=4, depth=2) match expected
  - Size=1 → single char
  - Depth=0 → filled triangle

**Verify:**
- `go test ./internal/sierpinski/...` pass

---

### Task 4: Sierpinski CLI Integration

Wire algo → CLI subcmd.

**Do:**
- Create `internal/cli/sierpinski.go` with `sierpinski` subcmd
- Flags: `--size` (default 32), `--depth` (default 5), `--char` (default '*')
- Call `sierpinski.Generate()` → stdout

**Verify:**
- `./fractals sierpinski` → triangle
- `./fractals sierpinski --size 16 --depth 3` → smaller triangle
- `./fractals sierpinski --help` show flag docs

---

### Task 5: Mandelbrot Algorithm

Impl Mandelbrot set ASCII renderer.

**Do:**
- Create `internal/mandelbrot/mandelbrot.go`
- Impl `Render(width, height, maxIter int, char string) []string`
- Map complex plane (-2.5 to 1.0 real, -1.0 to 1.0 imag) → output dims
- Map iter count → char gradient " .:-=+*#%@" (or single char if given)
- Create `internal/mandelbrot/mandelbrot_test.go` tests:
  - Output dims match requested width/height
  - Point inside set (0,0) → max-iter char
  - Point outside set (2,0) → low-iter char

**Verify:**
- `go test ./internal/mandelbrot/...` pass

---

### Task 6: Mandelbrot CLI Integration

Wire algo → CLI subcmd.

**Do:**
- Create `internal/cli/mandelbrot.go` with `mandelbrot` subcmd
- Flags: `--width` (default 80), `--height` (default 24), `--iterations` (default 100), `--char` (default "")
- Call `mandelbrot.Render()` → stdout

**Verify:**
- `./fractals mandelbrot` → recognizable Mandelbrot set
- `./fractals mandelbrot --width 40 --height 12` → smaller
- `./fractals mandelbrot --help` show flag docs

---

### Task 7: Character Set Configuration

`--char` flag work consistent across both cmds.

**Do:**
- Verify Sierpinski `--char` pass char to algo
- Mandelbrot `--char` use single char instead of gradient
- Tests for custom char output

**Verify:**
- `./fractals sierpinski --char '#'` use '#'
- `./fractals mandelbrot --char '.'` use '.' for all filled points
- Tests pass

---

### Task 8: Input Validation and Error Handling

Validate invalid inputs.

**Do:**
- Sierpinski: size > 0, depth >= 0
- Mandelbrot: width/height > 0, iterations > 0
- Clear error msgs for invalid input
- Tests for error cases

**Verify:**
- `./fractals sierpinski --size 0` → error, non-zero exit
- `./fractals mandelbrot --width -1` → error, non-zero exit
- Error msgs clear + helpful

---

### Task 9: Integration Tests

Add integration tests invoke CLI.

**Do:**
- Create `cmd/fractals/main_test.go` or `test/integration_test.go`
- Test full CLI invocation both cmds
- Verify output format + exit codes
- Error cases → non-zero exit

**Verify:**
- `go test ./...` pass all incl integration

---

### Task 10: README

Document usage + examples.

**Do:**
- Create `README.md` with:
  - Project description
  - Install: `go install ./cmd/fractals`
  - Usage examples both cmds
  - Example output (small samples)

**Verify:**
- README accurate
- Examples work