# Example: Server Project Verification

## Scenario

You've modified a backend server (Node.js/Express, Python/FastAPI, Go, etc.) and want to verify before deploying.

## Commands

```bash
/verify
```

## Expected Output (Node.js + Express)

```
Detected Node.js server project (package.json)
Running verification suite...

Step 1: Lint
> npm run lint
✓ 0 errors, 0 warnings

Step 2: Typecheck
> npm run typecheck
✓ No type errors

Step 3: Tests
> npm test

  Routes
    ✓ GET /api/users returns list
    ✓ POST /api/users creates user
    ✓ DELETE /api/users/:id removes user
  Middleware
    ✓ auth middleware validates tokens
    ✓ rate limiter blocks excess requests

  5 tests passed

✓ All verification steps passed
```

## Expected Output (Python + FastAPI)

```
Detected Python project (pyproject.toml)
Running verification suite...

Step 1: Lint (ruff)
> ruff check src/
✓ All checks passed

Step 2: Typecheck (mypy)
> mypy src/
✓ Success: no issues found in 12 source files

Step 3: Tests (pytest)
> pytest tests/ -v

  tests/api/test_users.py::test_get_users PASSED
  tests/api/test_users.py::test_create_user PASSED
  tests/api/test_auth.py::test_auth_middleware PASSED

  3 passed in 0.45s

✓ All verification steps passed
```

## Expected Output (Go)

```
Detected Go project (go.mod)
Running verification suite...

Step 1: Vet
> go vet ./...
✓ No issues found

Step 2: Tests
> go test ./... -v

  === RUN   TestGetUsers
  --- PASS: TestGetUsers (0.01s)
  === RUN   TestCreateUser
  --- PASS: TestCreateUser (0.02s)
  === RUN   TestAuthMiddleware
  --- PASS: TestAuthMiddleware (0.03s)

  PASS
  ok      github.com/myorg/server/api    0.123s

✓ All verification steps passed
```