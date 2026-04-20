# npc-bug-fix · 示例流程

> 本文件展示两个真实案例的完整执行过程，供 Claude Code 理解每一步的预期输入与输出。

---

## 案例 1 · NPAD-17159（Vue/JS · 数据上报缺字段）

### 1. 接收任务

**用户输入：**
```
https://www.teambition.com/task/69ae8deadd95777901fe003f
```

**任务信息：**
```
标题：【导入学员】导入学员数据统计--导入学员到小组，上报内容不全
执行者：梅天鹏
```

---

### 2. 检查任务

获取任务，确认执行者为梅天鹏 → 继续。

```javascript
const task = await teambition.getTask('69ae8deadd95777901fe003f');
// task.assignee === '梅天鹏' → 开始处理
```

---

### 2.5 同步代码

```bash
git pull --rebase   # 仅此处执行，后续步骤不再 pull
```

---

### 3. Bug 分析

**任务描述：**
```
【操作描述】学习小组点击导入用户操作，上报字段：
  - 导入时间 / 导入方式（新建导入 / 已有组导入）
  - 导入组名 / 导入组 CODE / 导入人数

【预期结果】上报完整数据
【实际结果】缺少 timeStamp 字段
```

**问题定位（3 个文件）：**
```
src/pages/classGroup/CreateClassGroup.vue
src/pages/classGroup/EditClassGroup.vue
src/pages/classGroup/components/ChooseStudent/ImportStudentModal.vue
```

---

### 4. Bug 修复

```javascript
// ❌ 修复前
sendAudioTalkEvent(EventAction.importStudentGroup, {
  p_r: false, evt_c: 200,
  evt_p: {
    importMethod: '新建导入',
    groupName: lastSaved?.groupName ?? '',
    groupCode: lastSaved?.groupCode ?? '',
    importCount,
  },
});

// ✅ 修复后（三个文件均添加 timeStamp）
sendAudioTalkEvent(EventAction.importStudentGroup, {
  p_r: false, evt_c: 200,
  evt_p: {
    importMethod: '新建导入',
    groupName: lastSaved?.groupName ?? '',
    groupCode: lastSaved?.groupCode ?? '',
    importCount,
    timeStamp: new Date().getTime(), // ✅ 补充缺失字段
  },
});
```

---

### 5. Code Review

调用 `code-review:review-local-changes` 技能审查改动（P0/P1 需修复后重新 review）。

**Test Case（本地验证，不提交）：**
```javascript
describe('Bug Fix Verification - NPAD-17159', () => {
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

**Review 结果：**
```
✅ Code Review 通过
- 0 test case fail / 0 warnings / 0 compile error
- 无新 bug 引入 / 无现有逻辑影响
```

---

### 5.5 编译验证

**本案例：Vue/JS 项目**

```bash
npm install
npm run lint
# 期望输出：No errors / No warnings
```

**如编译失败：**
```
❌ 编译错误
错误类型：[描述]   错误位置：[文件:行号]
→ 返回 Step 4 重新修复
```

---

### 6. App 内冒烟验证

**本案例：非 UI 数据上报 bug** → 至少触发关键链路一次，核对上报参数。

```
✅ 冒烟验证通过
- 导入学员操作可正常执行
- evt_p 中包含 timeStamp
- 未发现新增异常
```

> UI bug 必须在真实 App / 平台（Flutter / Vue / React / Electron / WebView）做 live smoke check。

---

### 7. 通知用户 Review【第一次确认】

```
修复完成：【导入学员】导入学员数据统计--导入学员到小组，上报内容不全
链接：https://www.teambition.com/task/69ae8deadd95777901fe003f

修复内容：
- CreateClassGroup.vue: 添加 timeStamp 参数
- EditClassGroup.vue: 添加 timeStamp 参数
- ImportStudentModal.vue: 添加 timeStamp 参数

Review + 编译验证：✅ 0 fail, 0 warning, 0 error
App 内冒烟验证：✅ 已完成

请确认是否提交代码。
```

**用户回复：** `确认`

---

### 8. 提交代码

```bash
# 仅添加本次修复文件（禁止 git add . 或 git add -A）
git add src/pages/classGroup/CreateClassGroup.vue
git add src/pages/classGroup/EditClassGroup.vue
git add src/pages/classGroup/components/ChooseStudent/ImportStudentModal.vue

git commit -m "fix: 补充导入学员上报 timeStamp 参数

- CreateClassGroup.vue: 添加 timeStamp 到 importStudentGroup 事件
- EditClassGroup.vue: 添加 timeStamp 到 importStudentGroup 事件
- ImportStudentModal.vue: 添加 timeStamp 到 importStudentSchedule 事件

基于 commit de16cfc7beea209f81aa1f9c6df471125a1b135c 的数据统计功能补充"

# ⚠️ 不要直接 push
```

---

### 9. 通知用户确认 Push【第二次确认】

```
代码已 commit (分支：feature-task-voice，commit: 1c8c6891e)
确认 push 到远端吗？
```

**用户回复：** `确认`

```bash
git push origin feature-task-voice
```

---

### 10. 更新 Teambition

**状态流转：** `开发中 → RESOLVED`

**必填信息：**
- 解决结果：`完成`
- 影响范围：`[受影响端/页面]`

**进展备注：**
```
已根据 commit de16cfc7beea209f81aa1f9c6df471125a1b135c 为三个文件添加 timeStamp 参数，
代码已提交推送到 feature-task-voice 分支。
```

**完成输出：**
```
代码已提交并 push 到 feature-task-voice 分支 (commit: 1c8c6891e)
Teambition 状态已更新为 RESOLVED ✅
```

---

## 案例 2 · NPAD-17443（Flutter · Windows 窗口无法关闭）

### 1. 接收任务

| 字段 | 值 |
|------|----|
| 任务 ID | NPAD-17443 |
| 标题 | 【历史问题】榜单编辑页面在电脑下方任务栏点击关闭，无法关闭 |
| 类型 | Flutter 桌面应用 |
| 执行者 | 梅天鹏 |

---

### 2. 检查任务 & 同步代码

执行者为梅天鹏 → 继续。

```bash
git pull --rebase
```

---

### 3. Bug 分析

**问题描述：** 榜单编辑页面在 Windows 任务栏点击 X，无法关闭。

**对比分析：**
- ✅ 其他页面可正常关闭（均已重写 `onWindowClose()`）
- ❌ `NPCRankingListPage` 未重写 `onWindowClose()`

**根本原因：** Windows 任务栏窗口预览的 X 按钮触发原生关闭消息，该页面未处理此事件。

---

### 4. Bug 修复

**文件：** `lib/business/ranking_list/views/page/npc_ranking_list_page.dart`

```dart
/// 修复 NPAD-17443：处理窗口原生关闭事件
/// Windows 任务栏窗口预览的 X 按钮会触发此方法
@override
void onWindowClose() {
  NPCWindowManager.instance.currentWindow().close();
  super.onWindowClose();
}
```

---

### 5. Code Review

**检查清单：**
- ✅ 修复针对 Teambition 描述的问题
- ✅ 与其他正常页面实现保持一致
- ✅ 不引入新 bug / 不影响现有逻辑

**编译验证：Flutter 项目**

```bash
flutter pub get && flutter analyze
# 期望：No issues found
```

**Review 结果：**
```
✅ 通过
- 添加 onWindowClose() 方法重写
- 调用 NPCWindowManager.close() 正确处理 setPreventClose
- 仅影响榜单编辑页面，不影响其他关闭方式
```

---

### 6. App 内冒烟验证

**本案例：UI 交互 bug** → 必须在真实 Windows 环境中点击任务栏 X 按钮验证。

```
✅ 冒烟验证通过
- 任务栏预览点击 X 可正常关闭榜单编辑页面
- 其他页面关闭功能不受影响
```

---

### 7. 通知用户 Review【第一次确认】

```
修复完成：NPAD-17443 - 榜单编辑页面无法通过 Windows 任务栏预览关闭

问题原因：NPCRankingListPage 未重写 onWindowClose()，无法处理原生关闭消息
修复方案：重写 onWindowClose()，调用 NPCWindowManager.close()
修改文件：lib/business/ranking_list/views/page/npc_ranking_list_page.dart (+9 行)

Review + 编译验证通过，请确认是否提交代码。
```

**用户回复：** `确认`

---

### 8. 提交代码

```bash
git add lib/business/ranking_list/views/page/npc_ranking_list_page.dart

git commit -m "fix: 修复榜单编辑页面无法通过 Windows 任务栏预览关闭的问题 (NPAD-17443)

问题原因：榜单页面未重写 onWindowClose()，无法处理 Windows 原生关闭消息
修复方案：重写 onWindowClose()，调用 NPCWindowManager.close() 正确处理 setPreventClose"

# ⚠️ 不要直接 push
```

---

### 9. 通知用户确认 Push【第二次确认】

```
代码已 commit (分支：develop-3.5.0)
确认 push 到远端吗？
```

**用户回复：** `确认`

```bash
git push origin develop-3.5.0
```

---

### 10. 更新 Teambition

**状态流转：** `待开发 → RESOLVED`

**必填信息：**
- 解决结果：`完成`
- 影响范围：`Windows 端榜单编辑页面`

**进展备注：**
```
已修复榜单编辑页面无法通过 Windows 任务栏预览关闭的问题。

问题原因：NPCRankingListPage 未重写 onWindowClose() 方法
修复方案：重写 onWindowClose()，调用 NPCWindowManager.close() 处理原生关闭消息
修改文件：lib/business/ranking_list/views/page/npc_ranking_list_page.dart

代码已提交并推送到 develop-3.5.0 分支。
```

---

## 流程图

```
用户提供任务链接
       │
       ▼
  检查执行者
  ┌────┴─────┐
非梅天鹏   梅天鹏
  │           │
询问用户    git pull --rebase
  │           │
  └────┬──────┘
       ▼
  读取 Bug 描述
       │
  有歧义？──是──► 向用户提问 ──► 得到答复
       │ 否                          │
       ◄─────────────────────────────┘
       │
       ▼
  定位问题代码 → 编写修复方案
       │
       ▼
  Code Review + 编译验证
       │
  通过？──否──► 重新修复 ──┐
       │ 是               │
       ◄───────────────────┘
       │
       ▼
  App 内冒烟验证
       │
       ▼
  【第一次确认】通知用户 Review
       │
  用户确认
       │
       ▼
  git add <修复文件> && git commit
       │
       ▼
  【第二次确认】通知用户确认 Push
       │
  用户确认
       │
       ▼
  git push
       │
       ▼
  更新 Teambition（状态 + 进展备注）
       │
       ▼
     完成 🎉
```

---

## 两次确认模板

**第一次（修复完成，请求 commit 授权）：**
```
修复完成：[任务标题]
链接：[Teambition URL]

问题原因：[简要说明]
修复方案：[简要说明]
修改文件：[文件路径列表]

Review + 编译验证：✅ 0 fail, 0 warning, 0 error
App 内冒烟验证：✅ 已完成

请确认是否提交代码。
```

**第二次（已 commit，请求 push 授权）：**
```
代码已 commit (分支：[branch]，commit: [hash])
确认 push 到远端吗？
```

---

## Teambition 状态更新说明

**正常路径：** `待开发 → 开发中 → RESOLVED`

**变更为 RESOLVED 时，弹窗必填：**
- **解决结果：** 完成
- **影响范围：** 填写受影响的端/页面（如 `PC 端榜单编辑页面`）

**进展备注模板：**
```
已修复 [问题描述]。

问题原因：[简要说明]
修复方案：[简要说明]
修改文件：[文件路径]

代码已提交并推送到 [branch] 分支。
```
