---
name: recap-progress-zh
description: "追踪和更新项目进度。显示当前焦点、已完成里程碑、阻塞项和下一步计划。当用户说「查看进度」「更新进度」或执行 '/recap-zh progress' 时触发。示例：'/recap-zh progress'、'/recap-zh progress update'。"
user-invocable: false
allowed-tools: "Read, Write, Edit, Bash, Glob, Grep"
metadata:
  version: "2.1.0"
---

# 进度追踪

维护当前项目的活跃进度文档。

## 参数

| $ARGUMENTS | 操作 |
|---|---|
| 空 / `progress` | 显示当前进度 |
| `update` | 从对话上下文更新进度 |

## 显示进度

1. 读取 `docs/context/PROGRESS.md`
2. 存在 → 展示内容
3. 不存在 → "暂无进度文件。使用 `/recap-zh progress update` 创建。"

## 更新进度

1. 执行 `date '+%Y-%m-%d %H:%M'`
2. 执行 `git log --oneline -10` 查看近期活动
3. 读取已有的 `docs/context/PROGRESS.md`（如存在）
4. 读取 `docs/context/META.json`（如存在）
5. 回顾当前对话上下文
6. 写入/覆盖 `docs/context/PROGRESS.md`（始终全量重写，非追加）

## 格式

```markdown
# 进度 — <项目名>

> 最后更新：YYYY-MM-DD HH:MM

## 当前焦点
- 正在进行的工作

## 已完成里程碑
- [YYYY-MM-DD] 里程碑描述
- [YYYY-MM-DD] 里程碑描述

## 阻塞项
- 阻塞描述及解决所需条件

## 下一步
- 按优先级排列的待办事项
```

## 规则

- 始终覆盖整个文件（非追加）— 这是当前状态的快照
- 每节保持简洁（3-7 条）
- 已完成里程碑按时间倒序
- 无阻塞项时省略该节
- 从最新 recap 的遗留问题填充下一步
