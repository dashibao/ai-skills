# Teambition API Reference

Reference for Steps 2 and 13 of the npc-bug-fix workflow.

---

## Authentication

| Method | Detail |
|--------|--------|
| Enterprise email | `meitianpeng@xdf.cn` |
| Login URL | `https://www.teambition.com/login/password` |
| Auth methods | DingTalk QR scan · Enterprise SSO (passwordless) |

---

## Endpoints

### Get Task Detail

```
GET https://www.teambition.com/task/{taskId}
```

### Get Task Activities

```
GET https://www.teambition.com/api/tasks/{taskId}/activities
```

**Response:**
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

**Common `action` values:**

| Action | Meaning |
|--------|---------|
| `activity.comment` | Comment — often contains fix clues |
| `activity.task.update.executor` | Executor changed |
| `activity.task.customfield.update.v2` | Custom field updated |
| `activity.task.create.from.ancestor` | Created from parent task |

When scanning activities, prioritise comments. They frequently contain critical context: error analysis, links to related bugs, parameter corrections.

### Update Task Status

```
POST https://www.teambition.com/api/tasks/{taskId}/status
{
  "status": "RESOLVED",
  "resolution": "完成"
}
```

### Post Progress Note

```
POST https://www.teambition.com/api/tasks/{taskId}/activities
{
  "content": "...",
  "type": "progress"
}
```

### Update Executor

```
POST https://www.teambition.com/api/tasks/{taskId}/assignee
{
  "assigneeId": "{userId}"
}
```

---

## Status Transitions

```
待开发 → 开发中 → RESOLVED → CLOSED
                ↑
           (re-open)
```

When transitioning to **RESOLVED**, the modal requires two fields:

| Field | Value |
|-------|-------|
| 解决结果 | 完成 |
| 影响范围 | Affected platform / page (e.g. "PC 端榜单编辑页面") |

---

## Progress Note Template (Step 13)

```
已修复 [问题描述]。

问题原因：[简要说明]
修复方案：[简要说明]
修改文件：[文件路径列表]

代码已提交并推送到 [branch] 分支。
```

Markdown is supported in progress notes.
