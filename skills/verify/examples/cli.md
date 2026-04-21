# Example: CLI Project Verification

## Scenario

You've made changes to a TypeScript CLI project and want to verify everything still works.

## Commands

```bash
/verify
```

## Expected Output

```
Detected TypeScript CLI project (package.json)
Running verification suite...

Step 1: Lint
> npm run lint
> eslint src/ --ext .ts,.tsx
✓ No linting errors

Step 2: Typecheck
> npm run typecheck
> tsc --noEmit
✓ Typecheck passed

Step 3: Tests
> npm test
> jest --coverage

Test Suites: 5 passed, 5 total
Tests:       23 passed, 23 total
Coverage:    89.5%

✓ All verification steps passed
```

## If Tests Fail

```
Step 3: Tests
> npm test

FAIL src/utils/parser.test.ts
  ● parseInput › should handle empty strings

    Expected: ""
    Received: null

    at parser.test.ts:45:12

✗ 1 test failed

Fix suggestion: The parser.test.ts test expects empty string handling.
Check src/utils/parser.ts line 23.
```