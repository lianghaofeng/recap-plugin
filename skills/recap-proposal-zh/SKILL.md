---
name: recap-proposal-zh
description: "创建、列出和查看设计方案。从对话中提取方案设计，存为结构化文档到 docs/context/proposals/。当用户说「写个方案」「设计文档」「保存这个计划」或执行 '/recap-zh proposal' 时触发。示例：'/recap-zh proposal'、'/recap-zh proposal list'、'/recap-zh proposal 001'。"
user-invocable: false
allowed-tools: "Read, Write, Edit, Bash, Glob, Grep"
metadata:
  version: "2.1.0"
---

# 设计方案

创建和管理从对话中提取的结构化设计方案。

## 参数

| $ARGUMENTS | 操作 |
|---|---|
| 空 / `proposal` | 从当前对话创建新方案 |
| `list` | 列出所有方案 |
| `<编号>` (如 `001`) | 查看指定方案 |

## 创建方案

1. 执行 `date '+%Y-%m-%d'`
2. 确定下一个方案编号：
   ```bash
   ls docs/context/proposals/*.md 2>/dev/null | wc -l
   ```
   下一编号 = 数量 + 1，零填充到 3 位（001、002……）
3. 回顾对话中的设计讨论、方案规划、架构决策
4. 写入 `docs/context/proposals/NNN-<slug>.md`
5. 更新 `docs/context/META.json` — 将文件名添加到 `"proposals"` 数组
6. 更新 `docs/context/INDEX.md` — 添加 📝 条目

`mkdir -p docs/context/proposals`

### 方案格式

```markdown
# 方案 NNN：<标题>

- **状态**：draft
- **日期**：YYYY-MM-DD
- **作者**：<从对话上下文获取或 "unknown">

## 问题
为什么需要这个改动。解决什么问题。

## 方案
具体的实现方案。要具体、可操作。

## 取舍
考虑过哪些替代方案，为什么不选。

## 实现步骤
实现此方案的关键步骤。

## 参考
- 相关 recap：[YYYY-MM-DD](../YYYY-MM-DD.md)
- 相关决策见 DECISIONS.md
```

### 状态值

| 状态 | 含义 |
|------|------|
| `draft` | 初始创建，仍在完善 |
| `accepted` | 批准实施 |
| `implemented` | 已完成 |
| `rejected` | 决定不做 |

## 列出方案

1. 读取 `docs/context/proposals/` 下所有文件
2. 提取每个方案的标题和状态
3. 表格展示：

```
| # | 标题 | 状态 | 日期 |
|---|------|------|------|
| 001 | 跨项目 META | implemented | 2026-03-24 |
| 002 | 进度追踪 | draft | 2026-03-24 |
```

无方案时告知用户。

## 查看方案

1. 读取 `docs/context/proposals/NNN-*.md`（按编号前缀匹配）
2. 展示内容
3. 未找到时列出可用方案

## 规则

- Slug 用小写、连字符分隔，最多 5 个词（如 `cross-project-meta`）
- 新方案始终为 `draft` 状态
- 问题节不超过 3 句话
- 实现步骤要可操作，不要模糊目标
