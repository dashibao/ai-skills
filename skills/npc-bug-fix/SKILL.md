---
name: npc-bug-fix
description: Use when handling Teambition bug tasks, including bug triage, code fixes, review loops, local test-case validation, compile checks, user review confirmation, code submission, and Teambition status updates.
---

# npc-bug-fix

This is a complete automated bug fix and code review skill set, applicable to all workspaces, projects, and sessions.

## 技能概述

当用户提供 Teambition bug 任务链接时，本技能指导完成修复、code review、编译验证、用户确认、代码提交和 Teambition 状态更新的完整流程。默认处理分配给 **梅天鹏 (meitianpeng@xdf.cn)** 的任务。**在编写修复方案之前**，须确认已完全理解需求；若有歧义或缺失信息，应先向用户提问并得到答复后再继续。

## 完整流程

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Bug Fix + Code Review Flow                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. 📬 接收 Teambition 任务                                               │
│     └─→ 用户提供任务链接 (如：https://www.teambition.com/task/xxx)          │
│                                                                          │
│  2. 🔍 检查任务详情                                                       │
│     └─→ 执行者 == 梅天鹏 ?                                               │
│         ├─→ YES: 继续修复流程                                             │
│         └─→ NO: 询问用户：这个bug现在 xx(执行者)名下，继续修复吗？              │
│             └─→ 用户回复确认继续 继续修复流程                                 │
│                                                                          │
│     ⚠️ Use Playwrite open teambition task. 必须完成以下两步（缺一不可）：      │
│     ├─→ A. 阅读 bug 描述（文字 + 图片/视频附件）                              │
│     │   └─→ 备注/附件：Teambition 备注区域的图片是需求/设计的关键来源，           │
│     │       视频是bug复现和bug现象的回放，必须逐一查看每个附件，不可跳过           │
│     └─→ B. 读取任务动态/活动                                                │
│         └─→ 通过 API: `/api/tasks/{taskId}/activities` 获取动态            │
│         └─→ 动态可能包含：评论、执行者变更、字段更新、关联任务链接等         │
│         └─→ 特别关注评论中的线索（如"端上参数不对"、"参考上一个bug"等）      │
│                                                                          │
│  3. 🔄 同步代码 (必须在修改前)                                             │
│     ├─→ 确认工作区干净 (git status)                                       │
│     │   └─→ 有未提交改动？ → 提示用户先处理                                 │
│     └─→ git pull --rebase                                                │
│                                                                          │
│  4. 🔍 理解问题 & 根因分析                                                 │
│     ├─→ 基于步骤 2 已获取的需求和动态信息理解问题                          │
│     ├─→ 判断问题类型（UI 异常 / 功能错误 / 性能问题 / 数据上报等）          │
│     ├─→ 定位相关代码（文件、函数、调用链路）                                │
│     ├─→ 输出根因分析，向用户说明你对 bug 的理解                             │
│     ├─→ 用户确认？                                                        │
│     │   ├─→ YES: 根因理解正确 → 进入步骤 5                                  │
│     │   └─→ NO: 用户提出异议 → 重新分析，直到用户确认为止                    │
│     │   └─→ 信息不足？ → 列出具体问题，请用户回答后再继续                    │
│     └─→ ⚠️ 未经用户确认根因分析，不得开始编码修复                            │
│                                                                          │
│  5. 🐛 Bug Fix Skill                                                     │
│     ├─→ 基于已确认的根因分析编写修复方案                                    │
│     └─→ 完成后 → Code Review Skill                                        │
│                                                                          │
│  6. 🔎 Code Review (使用已安装的 code-review:review-local-changes)         │
│     ├─→ 调用 code-review:review-local-changes 技能审查本次改动             │
│     ├─→ 额外要求：                                                        │
│     │   ├─→ ❌ 不能引入新的 bug                                           │
│     │   ├─→ ❌ 不能影响现有代码逻辑                                       │
│     │   └─→ ✅ 根据 Teambition 描述编写 test case 本地验证                  │
│     ├─→ 测试结果：                                                        │
│     │   ├─→ FAIL: 返回 Bug Fix Skill 重新修复                              │
│     │   └─→ PASS (0 fail, 0 warning, 0 error): → 编译验证                  │
│     └─→ 循环直到：0 test case fail, 0 warnings, 0 compile error           │
│                                                                          │
│  7. ✅ 编译验证                                                           │
│     ├─→ 检测项目类型并运行对应命令：                                       │
│     │   ├─→ Flutter:  flutter pub get && flutter analyze                 │
│     │   ├─→ Node/Vue/React: npm install && npm run lint (或 eslint)      │
│     │   └─→ 其他项目：使用项目中配置的 lint / build 命令                  │
│     ├─→ 确保：0 errors, 0 warnings                                       │
│     └─→ 如有编译错误：返回 Bug Fix Skill 重新修复                          │
│                                                                          │
│  8. 🧪 App 内冒烟验证                                                     │
│     ├─→ 在真实 App / 对应平台中复现并验证修复结果                          │
│     ├─→ UI bug: 必须做 live UI smoke check 并核对截图/录屏/肉眼结果         │
│     │   └─→ 适用于 Flutter、Vue、React、Electron、WebView 等真实运行环境    │
│     ├─→ 非 UI bug: 至少验证关键操作链路无回归                              │
│     └─→ FAIL: 返回 Bug Fix Skill 重新修复                                  │
│                                                                          │
│  9. 📝 通知用户 Review                                                    │
│     └─→ 向用户展示修复摘要，等待确认                                         │
│                                                                         │
│ 10. ✅ 用户确认                                                           │
│     └─→ 用户回复确认                                                       │
│                                                                         │
│ 11. 💾 提交代码 (commit only)                                             │
│     ├─→ git add <只添加本次修复的文件>                                      │
│     └─→ git commit -m "fix: [bug 描述]"                                  │
│                                                                         │
│ 12. 🔔 通知用户确认 push                                                  │
│     └─→ 展示 commit 信息（分支、hash、摘要），等待用户确认可以 push             │
│                                                                         │
│ 13. ⬆️  Push 代码                                                        │
│     └─→ git push                                                        │
│                                                                         │
│ 14. ✅ 更新 Teambition 状态                                              │
│     ├─→ 状态：待开发 → 开发中                                              │
│     ├─→ 状态：开发中 → RESOLVED                                           │
│     ├─→ 解决结果：完成                                                    │
│     └─→ 添加进展备注（10字左右的修复说明）                                     │
│                                                                          │
│ 15. 🧪 最终验证                                                           │
│     ├─→ 确认代码已 push                                                  │
│     ├─→ 确认 Teambition 状态已更新                                         │
│     └─→ 确认编译正常 + smoke check 已完成                                  │
│                                                                          │
│ 16. 🎉 完成                                                              │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

## 配置要求

### 1. Teambition 登录

使用企业邮箱登录：
- 邮箱：`meitianpeng@xdf.cn`
- 登录方式：钉钉扫码 or 企业免密登录

### 2. Code Review

使用已安装的 `code-review:review-local-changes` 技能（位于 `~/.claude/skills/code-review-review-local-changes/`）。无需额外安装。

#### 调用方式

修复完成后，调用该技能审查本次未提交的改动。技能会自动：
1. 通过 `git diff` 收集本次改动
2. 分派多个专项 review agent（正确性、可维护性、测试覆盖等）
3. 汇总反馈并给出改进建议

#### 处理 review 反馈

- **有 P0/P1 问题**：返回步骤 5 修复后重新 review
- **仅 P2 及以下建议**：酌情采纳，继续下一步
- **循环直到**：无 P0/P1 问题

#### 额外 Bug-Fix 检查项（在标准 review 之上）

- ❌ 不能引入新的 bug
- ❌ 不能影响现有代码逻辑
- ✅ 修复必须针对 Teambition 描述的问题
- ✅ 根据 Teambition 描述编写 test case 本地验证（不提交）
- ✅ 通过标准：0 test case fail, 0 warnings, 0 compile error
- ✅ UI 类 bug 在通知用户 review 前必须完成 live UI smoke check
- ✅ 非 UI 类 bug 至少验证关键操作链路一次

## 技能文件结构

```
npc-bug-fix/
├── SKILL.md                          # 本文件
├── references/
│   ├── teambition-api.md             # Teambition API 文档
│   └── review-checklist.md           # Review 检查清单
└── examples/
    └── sample-flow.md                # 示例流程
```

## 使用方式

用户提供 Teambition 任务链接后，按本 SKILL.md 流程执行：

```
用户: "处理这个 bug: https://www.teambition.com/task/xxx"
→ 读取 Teambition 任务
→ 按本技能流程执行修复
```

## 用户确认模板

### 修复完成 — 请求 review

向用户展示修复摘要，等待确认后再提交：

```
修复完成：[任务标题]
链接：[Teambition 链接]

修复内容：
- [修复点 1]
- [修复点 2]

Review + 编译验证：✅ 0 fail, 0 warning, 0 error
App 内冒烟验证：✅ 已完成

请确认是否提交代码。
```

### 提交完成 — 请求 push

```
代码已 commit，分支：[branch-name]

确认 push 到远端吗？
```

## 注意事项

1. **⚠️ 必须先阅读备注和所有动态，再开始修复**：
   - Teambition 备注区域可能包含设计稿、截图等附件，必须**逐一查看每个附件**
   - 任务动态中的评论往往包含关键线索（如"端上参数不对"、"参考上一个bug"等）
   - **跳过这两步是严重失误，会导致修复方向错误**
2. **修改代码前必须 pull**：在步骤 3 同步代码，确保基于最新远端开始修复
3. **先澄清需求再写方案**：在步骤 4「理解问题 & 根因分析」中，确认预期行为、复现条件、影响范围等已清楚；**输出根因分析并向用户说明，用户确认理解正确后才进入步骤 5 编码修复**；不确定时向用户提问，得到答复后再继续
4. **只提交任务相关文件**：`git add` 仅添加本次修复涉及的文件，禁止 `git add .` 或 `git add -A`
5. **Test case 不提交**：仅本地验证使用
6. **循环修复**：如果 review 发现问题，自动返回 bug fix 流程
7. **UI bug 必须做 live smoke check**：在真实 App / 对应平台里验证修复结果；适用于 Flutter、Vue、React、Electron、WebView 等环境；无法执行时必须明确记录阻塞原因
8. **用户确认**：关键节点（提交、push）必须先向用户展示摘要并等待确认
9. **状态同步**：确保 Teambition 状态与代码提交一致
10. **⚠️ 禁止直接 Push**：修复完成后不要直接 push，先通知用户 review
11. **✅ 编译验证**：修改代码后必须确保编译正常，无编译错误和警告
12. **以实际接口返回字段名为准**：接口字段可能与预期不同，需确认实际返回的字段名（如 `createName` vs `creatorName`）

## 故障处理

| 问题                | 解决方案                 |
| ------------------- | ------------------------ |
| Teambition 登录失败 | 使用钉钉扫码重新登录     |
| Code Review 卡住    | 检查 review 技能配置     |
| Git push 失败       | 先执行 git pull --rebase |
| Test case 失败      | 返回 bug fix 重新修复    |

## 版本历史

- **v1.8** (2026-04-17): 将步骤 4 拆分为「理解问题 & 根因分析」和「Bug Fix」两个独立步骤；编码修复前必须输出根因分析并经用户确认
- **v1.7** (2026-04-15): 在步骤 2 强制要求阅读备注附件和所有动态；备注区的图片是需求/设计关键来源，不可跳过；注意事项增加"以实际接口字段名为准"
- **v1.6** (2026-04-13): 在 Bug Fix Skill 流程中增加「读取任务动态/活动」步骤；动态中的评论和变更记录可能包含关键修复线索（如"参数不对"、"参考上一个bug"等）
- **v1.5** (2026-04-10): 在用户 review 前新增 App 内冒烟验证步骤；UI bug 强制执行 live UI smoke check，并将结果纳入模板与最终验证
- **v1.4** (2026-04-10): 修复两处一致性问题：(1) 主流程步骤 9 拆分为 commit + 独立 push 确认步骤，消除单次确认即可 push 的漏洞；(2) 编译验证改为按项目类型检测，不再硬编码 Flutter 命令
- **v1.3** (2026-04-10): skill `name` 改为 `npc-bug-fix`
- **v1.2** (2026-04-10): 编写修复方案前增加需求确认；不确定时向用户提问
- **v1.1** (2026-04-10): 修复流程缺陷：同步代码移至修改前、仅暂存任务文件、移除不存在的脚本引用、统一执行者字段、移除后台监听声明
- **v1.0** (2026-03-16): 初始版本，基于实际 Teambition 任务 NPAD-17159 流程编写

## 许可证

Apache 2.0
