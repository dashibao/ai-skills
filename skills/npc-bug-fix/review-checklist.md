# Review Checklist

Reference for Step 6 of the npc-bug-fix workflow. Covers test-case format and pass criteria.

---

## Hard Rules (all must hold)

| Category | Rule |
|----------|------|
| **Must not** | Introduce new bugs |
| **Must not** | Change logic unrelated to the bug |
| **Must not** | Delete existing test cases |
| **Must not** | Modify files not involved in the fix |
| **Must** | Fix the exact problem described in Teambition |
| **Must** | Write local test cases to verify the fix |
| **Must** | Pass all pre-existing test cases |
| **Must** | Complete smoke check (live UI for UI bugs; key-path run for non-UI bugs) |

---

## Writing Test Cases

Base each test on the Teambition bug description. Test the specific symptom that was broken.

```javascript
// Example — bug: "导入时间 timeStamp 未上报"
describe('Bug Fix Verification — NPAD-17159', () => {
  it('should include timeStamp in importStudentGroup evt_p', () => {
    const result = sendAudioTalkEvent('importStudentGroup', {
      evt_p: {
        importMethod: '新建导入',
        timeStamp: new Date().getTime(),
      },
    });
    expect(result.evt_p.timeStamp).toBeDefined();
  });
});
```

> Test cases are for **local verification only** — never commit them.

---

## Pass Criteria

All of the following must be true before proceeding to Step 7:

- [ ] 0 test case failures
- [ ] 0 compile errors
- [ ] 0 lint warnings
- [ ] All new test cases pass
- [ ] All pre-existing test cases pass

---

## Code Quality Checklist

Run after the automated review. Flag anything that would block a human reviewer:

- [ ] Code style consistent with surrounding code
- [ ] Variable names are clear and descriptive
- [ ] No leftover `console.log` or debug statements
- [ ] No hardcoded secrets or credentials
- [ ] No obvious performance regressions (unguarded loops, memory leaks)
- [ ] Comments added where logic is non-obvious

---

## Handling Review Feedback

| Severity | Action |
|----------|--------|
| P0 — crash / data loss / security | Fix immediately, re-review before continuing |
| P1 — functional regression | Fix immediately, re-review before continuing |
| P2 — code quality / style | Adopt at discretion; document rationale if skipped |
| P3 — suggestions | Optional |

> After 5 fix-and-review cycles without passing, escalate to the user for manual guidance.
