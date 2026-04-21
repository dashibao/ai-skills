---
name: verify
description: Verify a code change does what it should by running lint, typecheck, and tests. Detects project type and runs appropriate verification commands.
---

# /verify — Verify Code Correctness

Run the project's verification suite to ensure code changes are correct. Automatically detects project type and runs lint, typecheck, and test commands.

## What This Skill Does

When you invoke `/verify`, Claude will:

1. **Detect project type** — Identify the language/framework (Node.js, Python, Go, Rust, etc.)
2. **Run lint** — Check code style and formatting
3. **Run typecheck** — Verify type correctness (if applicable)
4. **Run tests** — Execute the test suite

## Project Detection

The skill detects project type by checking for config files:

| Project Type | Detection Files | Commands |
|--------------|-----------------|----------|
| Node.js/JS | `package.json` | `npm run lint` / `npm run typecheck` / `npm test` |
| Bun | `bun.lockb` or `bunfig.toml` | `bun run lint` / `bun test` |
| Python | `pyproject.toml`, `setup.py`, `requirements.txt` | `ruff check` / `mypy` / `pytest` |
| Go | `go.mod` | `go vet` / `go test ./...` |
| Rust | `Cargo.toml` | `cargo clippy` / `cargo test` |
| Makefile | `Makefile` with lint/typecheck/test targets | `make lint && make typecheck && make test` |

## Execution Steps

### 1. Detect Project Type

Check for config files to determine the project's language/framework:

```
ls package.json pyproject.toml go.mod Cargo.toml Makefile bun.lockb
```

### 2. Determine Commands

Based on the project type, identify the appropriate commands:

- **Makefile projects**: Run `make lint && make typecheck && make test`
- **Node.js**: Check package.json for scripts, run `npm run lint`, `npm run typecheck` (if exists), `npm test`
- **Bun**: Run `bun run lint` (if exists), `bun test`
- **Python**: Run `ruff check .` or `flake8`, `mypy .` (if configured), `pytest`
- **Go**: Run `go vet ./...`, `go test ./...`
- **Rust**: Run `cargo clippy`, `cargo test`

### 3. Run Verification Commands

Execute each verification step sequentially. Stop and report if any step fails.

### 4. Report Results

Summarize the results:

```
Verification Results:
- Lint: [PASS/FAIL] (X issues)
- Typecheck: [PASS/FAIL/SKIP] (X errors)
- Tests: [PASS/FAIL] (X passed, Y failed)

[If all pass: Code verification complete - ready for commit]
[If any fail: List specific issues to fix]
```

## Usage

Invoke with `/verify`:

```
/verify                  # Run full verification suite
/verify --quick          # Run only lint and typecheck (skip tests)
/verify --fix            # Run lint with auto-fix where possible
```

## User Arguments

If the user provides arguments:

- `--quick` or `--fast`: Skip tests, run only lint and typecheck
- `--fix`: Pass `--fix` flag to linters that support it
- Specific file paths: Run verification on only those files

## Handling Failures

When verification fails:

1. **Lint failures**: Show the specific issues and suggest fixes
2. **Typecheck errors**: Explain the type mismatches
3. **Test failures**: Show which tests failed and their output

Offer to fix issues automatically if they're straightforward.

## Notes

- Run verification after making code changes before committing
- If `Makefile` exists with `lint`, `typecheck`, `test` targets, prefer those
- For CI/CD alignment, use the same commands the CI pipeline uses
- Some projects may not have typecheck (JS without TypeScript, Python without mypy)

## Examples

### Node.js Project

```
$ /verify
Detected Node.js project (package.json)
Running: npm run lint && npm run typecheck && npm test

> my-project@1.0.0 lint
> eslint src/

> my-project@1.0.0 typecheck
> tsc --noEmit

> my-project@1.0.0 test
> jest

All 3 steps passed. Verification complete.
```

### Python Project

```
$ /verify
Detected Python project (pyproject.toml)
Running: ruff check . && mypy . && pytest

ruff: 0 issues
mypy: Success: no issues found
pytest: 12 passed

All verification steps passed.
```

### Makefile Project

```
$ /verify
Detected Makefile with lint/typecheck/test targets
Running: make lint && make typecheck && make test

make lint: ✓
make typecheck: ✓
make test: ✓

All targets passed. Ready for commit.
```