# npc-bug-fix — Example Flows

Two real cases demonstrating the complete workflow. Use these to understand the expected input and output at each step.

---

## Case 1 · NPAD-17159 — Vue/JS · Missing data-reporting field

**Task:** [https://www.teambition.com/task/69ae8deadd95777901fe003f](https://www.teambition.com/task/69ae8deadd95777901fe003f)  
**Title:** 【导入学员】导入学员数据统计--导入学员到小组，上报内容不全  
**Assignee:** 梅天鹏 → proceed automatically

### Step 2 — Read Task

**Bug description:**  
学习小组点击"导入用户"时，`importStudentGroup` 事件的 `evt_p` 缺少 `timeStamp` 字段。

| | |
|--|--|
| Expected | `evt_p` contains `importMethod`, `groupName`, `groupCode`, `importCount`, **`timeStamp`** |
| Actual | `timeStamp` missing |

Activities scan: no additional clues in comments.

### Step 3 — Sync Code

```bash
git pull --rebase
```

### Step 4 — Root-Cause Analysis

**Presented to user:**

> 三个文件调用 `sendAudioTalkEvent(EventAction.importStudentGroup, ...)` 时均未传 `timeStamp`：
> - `CreateClassGroup.vue`
> - `EditClassGroup.vue`
> - `ImportStudentModal.vue`
>
> 修复方案：在三处 `evt_p` 中补充 `timeStamp: new Date().getTime()`。

**User:** 确认

### Step 5 — Fix

```javascript
// Before (all three files)
sendAudioTalkEvent(EventAction.importStudentGroup, {
  p_r: false, evt_c: 200,
  evt_p: {
    importMethod: '新建导入',
    groupName: lastSaved?.groupName ?? '',
    groupCode: lastSaved?.groupCode ?? '',
    importCount,
  },
});

// After
sendAudioTalkEvent(EventAction.importStudentGroup, {
  p_r: false, evt_c: 200,
  evt_p: {
    importMethod: '新建导入',
    groupName: lastSaved?.groupName ?? '',
    groupCode: lastSaved?.groupCode ?? '',
    importCount,
    timeStamp: new Date().getTime(), // ← added
  },
});
```

### Step 6 — Code Review

```javascript
// Local test cases (not committed)
describe('Bug Fix Verification — NPAD-17159', () => {
  it('CreateClassGroup.vue should include timeStamp', () => {
    const content = fs.readFileSync('CreateClassGroup.vue', 'utf-8');
    expect(content).toContain('timeStamp: new Date().getTime()');
  });
  it('EditClassGroup.vue should include timeStamp', () => {
    expect(fs.readFileSync('EditClassGroup.vue', 'utf-8'))
      .toContain('timeStamp: new Date().getTime()');
  });
  it('ImportStudentModal.vue should include timeStamp', () => {
    expect(fs.readFileSync('ImportStudentModal.vue', 'utf-8'))
      .toContain('timeStamp: new Date().getTime()');
  });
});
```

Result: ✅ 0 fail · 0 warning · 0 error · no new bugs · no unrelated changes

### Step 7 — Compile Check

```bash
npm install && npm run lint
# Output: No errors / No warnings ✅
```

### Step 8 — Smoke Test

Non-UI bug (data reporting) → trigger import flow once, verify `evt_p.timeStamp` present in event payload.

Result: ✅ timeStamp appears in upload event

### Step 9 — First User Confirmation

```
修复完成：【导入学员】导入学员数据统计--导入学员到小组，上报内容不全
链接：https://www.teambition.com/task/69ae8deadd95777901fe003f

问题原因：三处 sendAudioTalkEvent 调用缺少 timeStamp 参数
修复方案：在 evt_p 中补充 timeStamp: new Date().getTime()
修改文件：
  - src/pages/classGroup/CreateClassGroup.vue
  - src/pages/classGroup/EditClassGroup.vue
  - src/pages/classGroup/components/ChooseStudent/ImportStudentModal.vue

Code Review：✅ 0 fail, 0 warning, 0 error
Smoke Test： ✅ 已完成

请确认是否提交代码。
```

**User:** 确认

### Step 10 — Commit

```bash
git add src/pages/classGroup/CreateClassGroup.vue \
        src/pages/classGroup/EditClassGroup.vue \
        src/pages/classGroup/components/ChooseStudent/ImportStudentModal.vue

git commit -m "fix: 补充导入学员上报 timeStamp 参数

- CreateClassGroup.vue: 添加 timeStamp 到 importStudentGroup 事件
- EditClassGroup.vue: 添加 timeStamp 到 importStudentGroup 事件
- ImportStudentModal.vue: 添加 timeStamp 到 importStudentSchedule 事件

Refs: NPAD-17159"
```

### Step 11 — Second User Confirmation

```
代码已 commit
分支：feature-task-voice  commit：1c8c6891e

确认 push 到远端吗？
```

**User:** 确认

### Steps 12–14

```bash
git push origin feature-task-voice
```

Teambition: `开发中 → RESOLVED` · 解决结果：完成 · 影响范围：Vue 端导入学员页

Progress note:
```
已为三个文件补充 timeStamp 参数，代码已提交并推送到 feature-task-voice 分支。
```

---

## Case 2 · NPAD-17443 — Flutter · Windows window cannot be closed

**Task:** NPAD-17443  
**Title:** 【历史问题】榜单编辑页面在电脑下方任务栏点击关闭，无法关闭  
**Assignee:** 梅天鹏 → proceed automatically

### Step 2 — Read Task

**Bug description:** On Windows, clicking the X button in the taskbar thumbnail preview does not close the ranking-list edit page.

Activities scan: no relevant comments.

### Step 3 — Sync Code

```bash
git pull --rebase
```

### Step 4 — Root-Cause Analysis

**Presented to user:**

> 其他页面均已重写 `onWindowClose()` 以处理 Windows 原生关闭消息，但 `NPCRankingListPage` 缺少该重写。  
> Windows 任务栏预览的 X 按钮触发的是原生 `WM_CLOSE` 消息，未处理则无响应。  
> 修复方案：在 `NPCRankingListPage` 中重写 `onWindowClose()`，调用 `NPCWindowManager.instance.currentWindow().close()`。

**User:** 确认

### Step 5 — Fix

**File:** `lib/business/ranking_list/views/page/npc_ranking_list_page.dart`

```dart
/// Fix NPAD-17443: handle native window-close event triggered by
/// the Windows taskbar thumbnail preview X button.
@override
void onWindowClose() {
  NPCWindowManager.instance.currentWindow().close();
  super.onWindowClose();
}
```

### Step 6 — Code Review

Review checklist:
- ✅ Matches the pattern used by all other pages that handle `onWindowClose`
- ✅ No new bugs introduced
- ✅ No unrelated logic changed

Result: ✅ 0 fail · 0 warning · 0 error

### Step 7 — Compile Check

```bash
flutter pub get && flutter analyze
# Output: No issues found ✅
```

### Step 8 — Smoke Test

UI bug → run on real Windows machine, click taskbar thumbnail X button on ranking-list edit page.

Result: ✅ Page closes correctly; other pages unaffected

### Step 9 — First User Confirmation

```
修复完成：NPAD-17443 - 榜单编辑页面无法通过 Windows 任务栏预览关闭

问题原因：NPCRankingListPage 未重写 onWindowClose()，无法处理 Windows 原生关闭消息
修复方案：重写 onWindowClose()，调用 NPCWindowManager.close()
修改文件：
  - lib/business/ranking_list/views/page/npc_ranking_list_page.dart (+9 行)

Code Review：✅ 0 fail, 0 warning, 0 error
Smoke Test： ✅ Windows 任务栏预览点击 X 可正常关闭

请确认是否提交代码。
```

**User:** 确认

### Step 10 — Commit

```bash
git add lib/business/ranking_list/views/page/npc_ranking_list_page.dart

git commit -m "fix: 修复榜单编辑页面无法通过 Windows 任务栏预览关闭的问题

- npc_ranking_list_page.dart: 重写 onWindowClose()，调用 NPCWindowManager.close()

Refs: NPAD-17443"
```

### Step 11 — Second User Confirmation

```
代码已 commit
分支：develop-3.5.0  commit：a3f91b2

确认 push 到远端吗？
```

**User:** 确认

### Steps 12–14

```bash
git push origin develop-3.5.0
```

Teambition: `待开发 → RESOLVED` · 解决结果：完成 · 影响范围：Windows 端榜单编辑页面

Progress note:
```
已修复榜单编辑页面无法通过 Windows 任务栏预览关闭的问题。

问题原因：NPCRankingListPage 未重写 onWindowClose() 方法
修复方案：重写 onWindowClose()，调用 NPCWindowManager.close() 处理原生关闭消息
修改文件：lib/business/ranking_list/views/page/npc_ranking_list_page.dart

代码已提交并推送到 develop-3.5.0 分支。
```

---

## Workflow at a Glance

```
User provides Teambition URL
         │
         ▼
  Read task (desc + all attachments + activities)
         │
  Executor == 梅天鹏?
  ├── No  → Ask user to confirm
  └── Yes → git pull --rebase
         │
         ▼
  Root-cause analysis → present to user → wait for confirmation
         │
         ▼
  Fix → Code Review → Compile Check
         │
  Issues found? → Yes → return to Fix
         │ No
         ▼
  Smoke Test
         │
         ▼
  ✋ First confirmation: "请确认是否提交代码"
         │
         ▼
  git add <fix files only> && git commit
         │
         ▼
  ✋ Second confirmation: "确认 push 到远端吗？"
         │
         ▼
  git push → Update Teambition → Done ✅
```
