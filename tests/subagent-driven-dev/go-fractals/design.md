# Go Fractals CLI - Design

## Overview

CLI tool gen ASCII fractals. Two fractal types, configurable output.

## Usage

```bash
# Sierpinski triangle
fractals sierpinski --size 32 --depth 5

# Mandelbrot set
fractals mandelbrot --width 80 --height 24 --iterations 100

# Custom character
fractals sierpinski --size 16 --char '#'

# Help
fractals --help
fractals sierpinski --help
```

## Commands

### `sierpinski`

Gen Sierpinski triangle via recursive subdivision.

Flags:
- `--size` (default: 32) - triangle base width chars
- `--depth` (default: 5) - recursion depth
- `--char` (default: '*') - char for filled points

Output: triangle → stdout, one line per row.

## `mandelbrot`

Render Mandelbrot set as ASCII. Iter count → chars.

Flags:
- `--width` (default: 80) - output width chars
- `--height` (default: 24) - output height chars
- `--iterations` (default: 100) - max iter for escape calc
- `--char` (default: gradient) - single char, or omit → gradient " .:-=+*#%@"

Output: rectangle → stdout.

## Architecture

```
cmd/
  fractals/
    main.go           # Entry point, CLI setup
internal/
  sierpinski/
    sierpinski.go     # Algorithm
    sierpinski_test.go
  mandelbrot/
    mandelbrot.go     # Algorithm
    mandelbrot_test.go
  cli/
    root.go           # Root command, help
    sierpinski.go     # Sierpinski subcommand
    mandelbrot.go     # Mandelbrot subcommand
```

## Dependencies

- Go 1.21+
- `github.com/spf13/cobra` for CLI

## Acceptance Criteria

1. `fractals --help` shows usage
2. `fractals sierpinski` outputs recognizable triangle
3. `fractals mandelbrot` outputs recognizable Mandelbrot set
4. `--size`, `--width`, `--height`, `--depth`, `--iterations` flags work
5. `--char` customizes output char
6. Invalid input → clear error msg
7. All tests pass