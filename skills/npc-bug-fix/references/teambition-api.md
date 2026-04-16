# Teambition API 参考

## 认证

### 企业邮箱登录
- URL: `https://www.teambition.com/login/password`
- 邮箱：`meitianpeng@xdf.cn`
- 登录方式：钉钉扫码 or 企业免密

## 任务操作

### 获取任务详情
```
GET https://www.teambition.com/task/{taskId}
```

### 获取任务动态/活动
```
GET https://www.teambition.com/api/tasks/{taskId}/activities
```

**响应示例：**
```json
[
  {
    "_id": "...",
    "action": "activity.comment",
    "content": {
      "comment": "端上参数不对，上一个bug有链接",
      "creator": "李晓雨51"
    },
    "created": "2026-04-13T03:03:47.584Z"
  },
  {
    "_id": "...",
    "action": "activity.task.update.executor",
    "content": {
      "executor": "侯锐6",
      "oldExecutor": "李晓雨51"
    }
  }
]
```

**常见 action 类型：**
- `activity.comment` - 评论
- `activity.task.update.executor` - 执行者变更
- `activity.task.customfield.update.v2` - 自定义字段更新
- `activity.task.create.from.ancestor` - 从父任务创建

**用途：** 动态中可能包含关键线索，如：
- 评论中的错误分析（"参数不对"、"参考上一个bug"）
- 相关任务链接
- 需求变更记录

### 更新任务状态
```
POST https://www.teambition.com/api/tasks/{taskId}/status
{
  "status": "RESOLVED",
  "resolution": "完成"
}
```

### 添加任务进展
```
POST https://www.teambition.com/api/tasks/{taskId}/activities
{
  "content": "修复说明...",
  "type": "progress"
}
```

### 更新执行者
```
POST https://www.teambition.com/api/tasks/{taskId}/assignee
{
  "assigneeId": "{qaUserId}"
}
```

## 状态流转

```
待开发 → 开发中 → RESOLVED → CLOSED
                ↓
            重新打开
```

## 备注

- 状态变更会自动记录到动态
- 执行者变更会自动通知相关人员
- 进展备注支持 Markdown 格式
