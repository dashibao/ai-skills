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
