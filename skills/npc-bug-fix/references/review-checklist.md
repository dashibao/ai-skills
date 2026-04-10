# Code Review 检查清单

## 核心要求（必须满足）

### ❌ 禁止事项
- [ ] 不能引入新的 bug
- [ ] 不能影响现有代码逻辑
- [ ] 不能修改与 bug 无关的代码
- [ ] 不能删除现有 test case

### ✅ 必须事项
- [ ] 编写修复方案前已理解需求；若有歧义或缺失信息，已向用户提问并得到答复
- [ ] 修复必须针对 Teambition 描述的问题
- [ ] 必须编写 test case 验证修复
- [ ] test case 仅用于本地验证（不提交）
- [ ] 必须通过所有现有 test case
- [ ] UI 类 bug 已完成 live UI smoke check（Flutter、Vue、React、Electron、WebView 等真实运行环境均适用）
- [ ] 非 UI 类 bug 已完成至少一次关键链路验证

## Test Case 验证

### 编写要求
```javascript
// 根据 Teambition 描述编写
// 示例：bug 描述提到"导入时间未上报"
describe('Bug Fix Verification', () => {
  it('should include timeStamp in evt_p', () => {
    const result = sendAudioTalkEvent('importStudentGroup', {
      evt_p: {
        importMethod: '新建导入',
        timeStamp: new Date().getTime(), // 必须包含
      }
    });
    expect(result.evt_p.timeStamp).toBeDefined();
  });
});
```

### 通过标准
- [ ] 0 test case fail
- [ ] 0 warnings
- [ ] 0 compile error
- [ ] 所有新 test case 通过
- [ ] 所有现有 test case 通过
- [ ] smoke check 通过，结果可复述

## Code Review 流程

### 1. 自动 Review

调用已安装的 `code-review:review-local-changes` 技能审查本次未提交的改动。
技能会分派多个专项 agent 并汇总反馈；P0/P1 问题必须修复后重新 review。

### 2. 检查项目
- [ ] 代码风格一致
- [ ] 变量命名清晰
- [ ] 注释充分
- [ ] 无 console.log 遗留
- [ ] 无调试代码

### 3. 安全检查
- [ ] 无敏感信息泄露
- [ ] 无 SQL 注入风险
- [ ] 无 XSS 风险
- [ ] 参数校验完整

### 4. 性能检查
- [ ] 无明显的性能退化
- [ ] 无内存泄漏风险
- [ ] 循环有终止条件

## 循环修复流程

```
Code Review
    ↓
发现问题？
    ├─→ YES: 返回 Bug Fix Skill 重新修复
    │         ↓
    │      修复后再次 Review
    │         ↓
    │      (循环直到通过)
    │
    └─→ NO: 通过
              ↓
         App 内冒烟验证
              ↓
           通知用户 Review
```

## Review 通过后的操作

1. 向用户展示修复摘要，请求确认
2. UI bug 先做 live UI smoke check；非 UI bug 先验证关键链路
3. 等待用户回复确认
4. `git add <本次修复的文件>`（禁止 `git add .` 或 `git add -A`）
5. 执行 git commit
6. 向用户展示 commit 信息，请求确认 push
7. 执行 git push
8. 更新 Teambition 状态为 RESOLVED
9. 添加进展备注

## 备注

- Review 不通过时，必须明确指出问题
- 每次修复后必须重新运行所有 test case
- UI 类问题如果无法完成真实 App / 前端运行环境冒烟验证，必须明确记录阻塞原因
- 循环次数记录到日志，超过 5 次需要人工介入
