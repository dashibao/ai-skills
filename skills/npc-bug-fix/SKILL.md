---
name: npc-bug-fix
description: Use when the user provides a Teambition task URL and asks to fix a bug. Runs the complete end-to-end workflow — task ingestion, root-cause confirmation, code fix, review loop, compile check, smoke test, git commit/push with dual user approval, and Teambition status update. Default assignee is 梅天鹏 (meitianpeng@xdf.cn); prompts user when task belongs to someone else.
---

# npc-bug-fix

End-to-end bug-fix workflow for Teambition tasks. Ingests a task URL, confirms root cause with the user, iterates through fix → review → compile → smoke test, then commits and closes the task — with **two mandatory user confirmations** before any git operation.

| When | Read |
|------|------|
| Step 2 — fetching task activities or Step 13 — updating status | `teambition-api.md` |
| Step 6 — running code review | `review-checklist.md` |
| Unfamiliar with the full workflow or need a concrete example | `sample-flow.md` |

---

## Workflow

### 1. Receive Task

User provides a Teambition URL: `https://www.teambition.com/task/<taskId>`

### 2. Read Task ⚠️ Both sub-steps are mandatory — skipping either causes wrong fixes

Open the task in Playwright and complete both:

**A. Read the full bug description**  
Title, description, expected vs actual behaviour, and **every attachment** (images, screenshots, videos). Attachments are the primary source of requirements and reproduction evidence.

**B. Fetch task activities**
```
GET /api/tasks/{taskId}/activities
```
Scan for comments containing fix clues (e.g. "参数不对", "参考上一个bug"), linked tasks, executor changes. See `references/teambition-api.md`.

**Check executor:**
- Executor == 梅天鹏 → proceed
- Executor ≠ 梅天鹏 → ask: "此任务在 [executor] 名下，是否继续修复？" and wait for confirmation

### 3. Sync Code

```bash
git status        # Must be clean — if dirty, ask user to handle it first
git pull --rebase # Run once here; do NOT pull again later in the workflow
```

### 4. Root-Cause Analysis — Confirm Before Coding

1. Classify bug type: UI anomaly · functional error · data-reporting · performance
2. Trace affected files, functions, and call paths
3. **Present root-cause analysis to the user and wait for confirmation**
   - Confirmed → proceed to Step 5
   - Rejected / unclear → revise and re-present
   - Info missing → list specific questions and wait for answers

> Do not write any fix code until root-cause is confirmed.

### 5. Fix

Write the minimum change that resolves the confirmed root cause. Do not touch unrelated code.

### 6. Code Review

Call `code-review:review-local-changes`. It dispatches multiple review agents and aggregates feedback.

Write local test cases to verify the fix (see `references/review-checklist.md` for format and pass criteria). **Do not commit test cases.**

| Outcome | Action |
|---------|--------|
| P0 / P1 issues | Return to Step 5, fix, re-review |
| P2 and below only | Adopt at discretion, continue |
| 0 fail · 0 warning · 0 error | Proceed |

> If review loops exceed 5 iterations, pause and escalate to the user for manual guidance.

### 7. Compile Check

| Project type | Command |
|--------------|---------|
| Flutter | `flutter pub get && flutter analyze` |
| Node / Vue / React | `npm install && npm run lint` |
| Other | Use the project's configured lint/build command |

Target: **0 errors, 0 warnings.** Any failure → return to Step 5.

### 8. Smoke Test

| Bug type | Requirement |
|----------|-------------|
| UI bug | Live UI smoke check in the real runtime (Flutter / Vue / React / Electron / WebView). Record outcome. |
| Non-UI bug | Trigger the key operation path at least once; confirm no regression. |

Failure → return to Step 5. If blocked, document the blocker explicitly.

### 9. ✋ First User Confirmation — Commit Approval

```
修复完成：[任务标题]
链接：[Teambition URL]

问题原因：[简要说明]
修复方案：[简要说明]
修改文件：
  - [file 1]
  - [file 2]

Code Review：✅ 0 fail, 0 warning, 0 error
Smoke Test： ✅ 已完成

请确认是否提交代码。
```

Wait for explicit user confirmation before proceeding.

### 10. Commit

```bash
# Stage only the files changed for this fix
git add <file1> <file2>   # Never: git add . / git add -A

git commit -m "fix: <concise description>

- [file 1]: what changed and why
- [file 2]: what changed and why

Refs: <Teambition task ID>"
```

### 11. ✋ Second User Confirmation — Push Approval

```
代码已 commit
分支：[branch]  commit：[short hash]

确认 push 到远端吗？
```

Wait for explicit user confirmation before pushing.

### 12. Push

```bash
git push origin <branch>
```

### 13. Update Teambition

1. Transition status: `开发中 → RESOLVED`
2. Set resolution field: `完成`
3. Fill **影响范围** (e.g. "PC 端榜单编辑页面")
4. Post progress note using the template in `references/teambition-api.md`

### 14. Done ✅

Verify: code pushed · Teambition RESOLVED · compile clean · smoke test passed.

---

## Critical Rules

| # | Rule |
|---|------|
| 1 | Read **all** attachments and activities in Step 2. Skipping them causes wrong-direction fixes. |
| 2 | `git pull --rebase` runs once in Step 3 only. |
| 3 | Root-cause must be confirmed by the user before writing any code. |
| 4 | `git add` only fix-related files. `git add .` and `git add -A` are forbidden. |
| 5 | Test cases: local verification only — never commit. |
| 6 | Two explicit user confirmations required: before commit (Step 9) and before push (Step 11). |
| 7 | Compile must be clean (0 errors, 0 warnings) before Step 9. |
| 8 | Review loops > 5 → escalate to user. |
| 9 | Always verify actual API response field names against the real response; do not assume (e.g. `createName` vs `creatorName`). |

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Teambition login fails | Re-authenticate via DingTalk QR or enterprise SSO |
| Code review hangs | Check `code-review:review-local-changes` skill config |
| `git push` fails | Run `git pull --rebase` first |
| Test case fails | Return to Step 5 |
| Smoke test blocked | Document blocker, notify user, do not proceed silently |

---

## File Structure

```
npc-bug-fix/
├── SKILL.md              ← This file — core workflow and critical rules
├── teambition-api.md     ← Auth, API endpoints, status transitions, progress-note template
├── review-checklist.md   ← Test-case format, pass criteria, severity guidance
├── sample-flow.md        ← Two complete worked examples (NPAD-17159, NPAD-17443)
└── LICENSE.txt
```
