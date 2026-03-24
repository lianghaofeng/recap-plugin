---
name: recap-zh
description: "对话记录与每日复盘。按日期将对话摘要存档到 docs/context/。当用户说「recap」「总结一下」「复盘」「今天做了什么」或想记笔记时触发。示例：'/recap-zh'、'/recap-zh note 修了认证bug'、'/recap-zh status'。周报/月报/搜索自动路由到子 skill。"
user-invocable: true
allowed-tools: "Read, Write, Edit, Bash, Glob, Grep"
metadata:
  version: "2.0.0"
---

# Recap — 对话记录与复盘

将对话提炼为结构化摘要，按日期存档到 `docs/context/`。

## 参数路由

| $ARGUMENTS | 操作 |
|---|---|
| 空 | 完整总结（见下方） |
| `weekly` | 调用 `recap:recap-weekly-zh` skill |
| `monthly` | 调用 `recap:recap-monthly-zh` skill |
| `note <内容>` | 追加快速笔记（见下方） |
| `status` | 查看今日状态（见下方） |
| `search <关键词>` | 调用 `recap:recap-search-zh` skill |
| `projects` | 调用 `recap:recap-projects-zh` skill |

当前参数值: $ARGUMENTS

如果参数匹配子 skill（weekly、monthly、search、projects），调用对应 skill 后结束。

---

## 完整总结（无参数）

1. 执行 `date '+%Y-%m-%d %H:%M'` 和 `git status --short`
2. 回顾对话 — 提炼以下内容（某项无内容则省略，不写"无"）：
   - **讨论内容** — 涉及的主题和技术点（2-5 条）
   - **完成工作** — 实际完成的编码、配置、调试等
   - **文件变更** — 从 git status 提取，标注 M/A/D
   - **关键决策** — "为什么选 A 不选 B"，附理由
   - **遗留问题** — 未完成的工作、后续计划
3. 检查 `docs/context/YYYY-MM-DD.md` 是否存在
4. 写入摘要（新建或作为 Session N 追加，用 `---` 分隔）
5. 更新 INDEX.md 和 META

### 文件格式

新建：`# YYYY-MM-DD 对话记录` → `## Session 1 (HH:MM)` → 各节。
追加：读取已有文件确定下一个 Session 编号，用 `---` 分隔追加。

---

## 快速笔记（`note <内容>`）

1. 执行 `date '+%Y-%m-%d %H:%M'`
2. 文件不存在 → 创建含标题和 `## 笔记` 节
3. 存在且有 `## 笔记` → 在其下追加
4. 存在但无 `## 笔记` → 在末尾添加该节
5. 格式：`- [HH:MM] 内容`

---

## 状态速查（`status`）

1. 检查 `docs/context/YYYY-MM-DD.md` 是否存在
2. 存在 → 显示 session 数量、最近时间、笔记数
3. 不存在 → "今天还没有 recap"

---

## META 同步（每次写入后）

写入 recap/笔记后，同时更新以下两个文件：

### 项目 META：`docs/context/META.json`
```json
{
  "project": "<git 根目录名>",
  "totalSessions": "<数量>",
  "lastSession": "<ISO 时间戳>",
  "recentTopics": ["<最新讨论内容>"],
  "remainingIssues": ["<最新遗留问题>"],
  "recapFiles": ["YYYY-MM-DD.md", ...]
}
```

### 全局 META：`~/.claude/recap/projects/<项目名>.json`
```bash
mkdir -p ~/.claude/recap/projects
```
```json
{
  "project": "<项目名>",
  "path": "<git 根目录绝对路径>",
  "lastSession": "<ISO 时间戳>",
  "lastRecapFile": "docs/context/YYYY-MM-DD.md",
  "remainingIssues": ["<最新遗留问题>"],
  "recentTopics": ["<最新讨论内容>"],
  "sessionCount": "<总数>"
}
```

---

## INDEX.md 维护

每次写入后更新 `docs/context/INDEX.md`。按月分组（最新在上），日期最新在上。📋 周报 📊 月报。不存在则创建。

## 文件写入

`mkdir -p docs/context/weekly docs/context/monthly`

使用 **Write** tool。WSL2 环境下如不可见改用 Bash heredoc。
