# npc-bug-fix 示例流程

## 实际案例：NPAD-17159

### 1. 接收任务

用户提供 Teambition 任务链接：
```
https://www.teambition.com/task/69ae8deadd95777901fe003f
任务：【导入学员】导入学员数据统计--导入学员到小组，上报内容不全
执行者：梅天鹏
```

### 2. 检查任务

```javascript
// 自动打开 Teambition 任务
const task = await teambiton.getTask('69ae8deadd95777901fe003f');

// 检查执行者（assignee 是当前负责人，即 Teambition 中的"执行者"）
if (task.assignee === '梅天鹏') {
  console.log('开始处理 bug...');
  startBugFix(task);
} else {
  // 执行者不是梅天鹏，询问用户是否继续
  console.log(`任务执行者是 ${task.assignee}，询问用户是否继续修复`);
}
```

### 3. Bug 分析

**任务描述：**
```
【操作描述】：
学习小组点击导入用户操作
- 导入时间
- 导入方式
  - 新建导入
  - 已有组导入
- 导入组名
- 导入组 CODE
- 导入人数

【预期结果】：上报完整数据
【实际结果】：缺少 timeStamp 字段
```

**问题定位：**
- 文件 1: `src/pages/classGroup/CreateClassGroup.vue`
- 文件 2: `src/pages/classGroup/EditClassGroup.vue`
- 文件 3: `src/pages/classGroup/components/ChooseStudent/ImportStudentModal.vue`

### 4. Bug 修复

```javascript
// 修复前
sendAudioTalkEvent(EventAction.importStudentGroup, {
  p_r: false,
  evt_c: 200,
  evt_p: {
    importMethod: '新建导入',
    groupName: lastSaved?.groupName ?? '',
    groupCode: lastSaved?.groupCode ?? '',
    importCount,
  },
});

// 修复后
sendAudioTalkEvent(EventAction.importStudentGroup, {
  p_r: false,
  evt_c: 200,
  evt_p: {
    importMethod: '新建导入',
    groupName: lastSaved?.groupName ?? '',
    groupCode: lastSaved?.groupCode ?? '',
    importCount,
    timeStamp: new Date().getTime(), // ✅ 添加 timeStamp
  },
});
```

### 5. Code Review

#### 调用 code-review:review-local-changes 技能

修复完成后调用已安装的 review 技能审查本次改动，处理返回的反馈（P0/P1 需修复后重新 review）。

#### 编写 Test Case
```javascript
// 本地验证，不提交
describe('Bug Fix Verification - NPAD-17159', () => {
  it('CreateClassGroup.vue should include timeStamp', () => {
    const content = fs.readFileSync('CreateClassGroup.vue', 'utf-8');
    expect(content).toContain('timeStamp: new Date().getTime()');
  });

  it('EditClassGroup.vue should include timeStamp', () => {
    const content = fs.readFileSync('EditClassGroup.vue', 'utf-8');
    expect(content).toContain('timeStamp: new Date().getTime()');
  });

  it('ImportStudentModal.vue should include timeStamp', () => {
    const content = fs.readFileSync('ImportStudentModal.vue', 'utf-8');
    expect(content).toContain('timeStamp: new Date().getTime()');
  });
});
```

#### Review 结果
```
✅ Code Review 通过

检查结果：
- 0 test case fail
- 0 warnings
- 0 compile error
- 无新 bug 引入
- 无现有逻辑影响

准备编译验证...
```

### 5.5 编译验证 (新增)

> **本案例为 Vue/JS 项目**，使用对应的 lint/build 命令；Flutter 项目改用 `flutter pub get && flutter analyze`。

#### Vue/JS 项目
```bash
# 安装依赖（如尚未安装）
npm install

# 静态分析 / lint
npm run lint

# 确保输出:
# No errors / No warnings
```

#### 编译检查清单
- [ ] 0 compile errors
- [ ] 0 warnings
- [ ] lint 通过
- [ ] 无新的 lint 警告

#### 如果编译失败
```
❌ 编译错误

错误类型：[错误描述]
错误位置：[文件路径：行号]

返回 Bug Fix Skill 重新修复...
```

### 6. App 内冒烟验证

> **本案例为非 UI 数据上报 bug**，至少验证关键链路一次；如果是 UI bug，则必须在真实 App / 对应平台中做 live UI smoke check。该要求同样适用于 Flutter、Vue、React、Electron、WebView 等真实运行环境。

#### 本地验证
```bash
# 按 Teambition 描述实际触发一次导入学员链路
# 核对事件上报参数中已包含 timeStamp
```

#### 冒烟验证结果
```
✅ App 内冒烟验证通过

检查结果：
- 导入学员操作可正常执行
- evt_p 中包含 timeStamp
- 未发现新增异常
```

### 7. 通知用户 Review

向用户展示修复摘要：
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

### 8. 用户确认

**用户回复：**
```
确认
```

### 9. 提交代码 (commit only)

```bash
# 注意: git pull 已在步骤 2.5（修复前同步）执行，此处无需再 pull

# 1. 仅添加本次修复涉及的文件（禁止 git add . 或 git add -A）
git add src/pages/classGroup/CreateClassGroup.vue
git add src/pages/classGroup/EditClassGroup.vue
git add src/pages/classGroup/components/ChooseStudent/ImportStudentModal.vue

# 2. 提交
git commit -m "fix: 补充导入学员上报 timeStamp 参数

- CreateClassGroup.vue: 添加 timeStamp 到 importStudentGroup 事件
- EditClassGroup.vue: 添加 timeStamp 到 importStudentGroup 事件  
- ImportStudentModal.vue: 添加 timeStamp 到 importStudentSchedule 事件

基于 commit de16cfc7beea209f81aa1f9c6df471125a1b135c 的数据统计功能补充"

# ⚠️ 不要直接 push，先通知用户确认
```

### 10. 通知用户确认 push

```
代码已 commit (分支：feature-task-voice，commit: 1c8c6891e)

确认 push 到远端吗？
```

**用户回复：**
```
确认
```

### 11. Push 代码

```bash
git push origin feature-task-voice
```

### 12. 更新 Teambition 状态

**状态变更：**
```
开发中 → RESOLVED
```

**解决结果：**
```
完成
```

**执行者变更：**
```
梅天鹏 → 郑满全 (QA)
```

**添加进展备注：**
```
已根据 commit de16cfc7beea209f81aa1f9c6df471125a1b135c 为三个文件添加 timeStamp 参数，
代码已提交推送到 feature-task-voice 分支。
```

### 13. 完成

```
代码已提交并 push 到 feature-task-voice 分支 (commit: 1c8c6891e)
Teambition 状态已更新为 RESOLVED，执行者流转至 QA（郑满全）
```

## 流程图

```
用户提供任务链接
   ↓
检查任务 (执行者=梅天鹏？)
   ↓
是 → git pull --rebase (同步代码)
   ↓
读取 Bug 描述
   ↓
理解需求（编写方案前须确认清楚）
   ↓
不确定？── 是 ──→ 向用户提问 → 得到答复 ──┐
   ↓ 否                                  │
定位问题代码 ←──────────────────────────┘
   ↓
编写修复方案
   ↓
Code Review
   ↓
通过？─── 否 ──→ 重新修复
   ↓ 是
App 内冒烟验证
   ↓
通知用户 Review
   ↓
用户确认（第一次：确认提交）
   ↓
git add <修复文件>
   ↓
git commit
   ↓
通知用户确认 push（第二次）
   ↓
用户确认（第二次：确认 push）
   ↓
git push
   ↓
更新 Teambition 状态
   ↓
添加进展备注
   ↓
完成 🎉
```

## 关键要点

1. **手动触发**：用户提供 Teambition 任务链接来触发本技能
2. **需求先行**：编写修复方案前确认已理解需求；有歧义时先提问，再写方案与改代码
3. **身份验证**：只处理分配给梅天鹏的任务
4. **严格 Review**：0 fail, 0 warning, 0 error
5. **先验真实运行环境**：UI bug 在通知用户 review 前必须完成 live smoke check；Flutter、Vue、React、Electron、WebView 都适用
5. **Test Case**：本地验证，不提交
6. **循环修复**：直到完全通过
7. **用户确认**：提交前必须用户 review
8. **修改前 pull**：修复代码前先 pull 同步远端，提交时无需再 pull
9. **状态同步**：Teambition 状态与代码一致
10. **⚠️ 重要**：代码修复完成后**不要直接 push**，先通知用户 review
11. **📋 Teambition 状态更新**: 必须填写必填信息（解决结果 + 影响范围）

---

## Teambition 状态更新完整流程

### 状态流转
```
待开发 → 开发中 → RESOLVED
```

### 操作步骤

#### 1. 变更状态为 RESOLVED
```
点击状态按钮（如"待开发"）
  ↓
选择"开发中"（如果当前是待开发）
  ↓
再次点击状态（现在是"开发中"）
  ↓
选择"RESOLVED"
```

#### 2. 填写必填信息弹窗
```
┌─────────────────────────────────────┐
│  填写任务流转的必填信息          ×  │
├─────────────────────────────────────┤
│  ⚠️ 当任务从其他状态变更为         │
│     「RESOLVED」时，以下必填字段    │
│     不可为空。若无填写权限，请联   │
│     系项目管理员处理                │
├─────────────────────────────────────┤
│  解决结果：[待添加 ▼]               │
│  影响范围：[待添加 ▼]               │
├─────────────────────────────────────┤
│              [取消]  [确定]         │
└─────────────────────────────────────┘
```

**必填字段**:
- **解决结果**: 选择"完成"
- **影响范围**: 填写影响范围
  - 示例：`PC 端榜单编辑页面`
  - 示例：`Pad 端活动卡片`
  - 示例：`小程序首页`

#### 3. 变更执行者
```
点击执行者（梅天鹏）
  ↓
选择 QA 负责人（郑满全）
```

#### 4. 添加进展备注
```
点击"填写进展"
  ↓
输入修复说明（使用下方模板）
  ↓
点击"发布"
```

### 进展备注模板

```markdown
已修复 [问题描述]。

问题原因:
[简要说明]

修复方案:
[简要说明]

修改文件:
- [文件路径]

代码已提交并推送到 develop-3.5.0 分支。
```

### 完整示例

**备注内容**:
```markdown
已修复 pad 端活动卡片更多操作菜单删除按钮被遮挡问题。

问题原因:
- 新增导入按钮后，更多操作菜单变长
- 弹窗位置计算未考虑屏幕底部边界
- 导致弹窗超出屏幕，部分内容无法点击

修复方案:
- 修改 showPopDialog 方法，增加屏幕边界检测
- 当弹窗底部超出屏幕时，自动显示在触发按钮上方
- 确保菜单始终在屏幕内可见

修改文件:
- lib/business/classes_group/widgets/base/classes_group_utils.dart

代码已提交并推送到 develop-3.5.0 分支。
```

**必填信息**:
- 解决结果：完成
- 影响范围：Pad 端活动卡片

**状态变更**:
- 待开发 → 开发中 → RESOLVED

**执行者变更**:
- 梅天鹏 → 郑满全 (QA)

---

## 实际案例 2: NPAD-17443 (Flutter 窗口关闭问题)

### 任务信息

| 字段 | 值 |
|------|-----|
| **任务 ID** | NPAD-17443 |
| **标题** | 【历史问题】榜单编辑页面在电脑下方任务栏点击关闭，无法关闭 |
| **类型** | Flutter 桌面应用 |
| **执行者** | 梅天鹏 |

### Bug 分析

**问题描述**:
> 榜单编辑页面在电脑下方任务栏点击关闭，无法关闭

**对比分析**:
- ✅ 其他页面窗口可以正常关闭
- ❌ 只有 NPCRankingListPage 页面不行

**根本原因**:
- NPCRankingListPage 没有重写 `onWindowClose()` 方法
- Windows 任务栏窗口预览的 X 按钮触发原生关闭消息
- 其他能正常关闭的页面都有 `onWindowClose()` 方法

### 修复方案

**文件**: `lib/business/ranking_list/views/page/npc_ranking_list_page.dart`

```dart
/// 修复 NPAD-17443: 处理窗口原生关闭事件
/// Windows 任务栏窗口预览的 X 按钮会触发此方法
@override
void onWindowClose() {
  // 调用窗口管理器的 close 方法，正确处理 setPreventClose
  NPCWindowManager.instance.currentWindow().close();
  super.onWindowClose();
}
```

### Code Review

**检查清单**:
- [x] ❌ 不引入新的 bug
- [x] ❌ 不影响现有代码逻辑
- [x] ✅ 修复针对 Teambition 描述的问题
- [x] ✅ 与其他正常页面保持一致的实现

**Review 结果**:
```
✅ 通过

修改内容:
- 添加 onWindowClose() 方法重写
- 调用 NPCWindowManager.close() 处理原生关闭消息

影响范围:
- 仅影响榜单编辑页面
- 支持 Windows 任务栏预览关闭
- 不影响其他关闭方式
```

### 通知用户 Review

向用户展示修复摘要：
```
修复完成：NPAD-17443 - 榜单编辑页面无法通过 Windows 任务栏预览关闭

问题原因:
- NPCRankingListPage 没有重写 onWindowClose() 方法
- Windows 任务栏窗口预览的 X 按钮触发原生关闭消息

修复方案:
- 重写 onWindowClose() 方法
- 调用 NPCWindowManager.close() 正确处理 setPreventClose

修改文件:
- lib/business/ranking_list/views/page/npc_ranking_list_page.dart (+9 行)

Review + 编译验证通过，请确认是否提交代码。
```

### 用户确认后提交

```bash
# 1. 等待用户回复"确认"
# 注意: git pull 已在修复前（步骤 3 同步代码）执行，此处无需再 pull

# 2. 仅添加本次修复涉及的文件（禁止 git add . 或 git add -A）
git add lib/business/ranking_list/views/page/npc_ranking_list_page.dart

# 3. 提交代码
git commit -m "fix: 修复榜单编辑页面无法通过 Windows 任务栏预览关闭的问题 (NPAD-17443)

问题原因:
- 榜单页面没有重写 onWindowClose() 方法
- Windows 任务栏窗口预览的 X 按钮触发原生关闭消息

修复方案:
- 重写 onWindowClose() 方法
- 调用 NPCWindowManager.close() 正确处理 setPreventClose"

# 7. ⚠️ 不要直接 push，等待用户确认可以 push

# 8. 用户确认可以 push 后
git push origin develop-3.5.0
```

### 更新 Teambition

**状态变更**: 待开发 → RESOLVED  
**执行者变更**: 梅天鹏 → 郑满全 (QA)  
**备注**: 已修复，代码已提交，等待测试验证

---

## 注意事项

### 🚫 禁止直接 Push

修复完成后**不要直接 push 代码**，流程应该是：

```
修复完成 → Code Review → 通知用户 → 用户确认 → 提交代码 → 通知用户 → 用户确认 → Push 代码
```

### ✅ 正确流程

1. 修复 Bug
2. Code Review（确保 0 fail, 0 warning, 0 error）
3. **通知用户 review**（等待确认）
4. 用户确认后提交代码（commit）
5. **再次通知用户**（告知已 commit，询问是否可以 push）
6. 用户确认可以 push 后执行 push
7. 更新 Teambition 状态

### 📋 用户确认模板

**第一次确认（修复完成）**:
```
修复完成：[任务标题]
链接：[Teambition 链接]

问题原因：[简要说明]
修复方案：[简要说明]
修改文件：[文件路径]

Review + 编译验证通过，请确认是否提交代码。
```

**第二次确认（已 commit，询问 push）**:
```
代码已 commit (分支：[branch-name])
确认 push 到远端吗？
```
