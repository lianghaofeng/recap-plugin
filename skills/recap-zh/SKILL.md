---
name: recap-zh
description: "对话记录与每日复盘。按日期将对话摘要存档到 docs/recap_context/。当用户说「recap」「总结一下」「复盘」「今天做了什么」或想记笔记时触发。示例：'/recap-zh'、'/recap-zh note 修了认证bug'、'/recap-zh status'。周报/月报/搜索/进度/方案自动路由到子 skill。"
user-invocable: true
allowed-tools: "Read, Write, Edit, Bash, Glob, Grep"
metadata:
  version: "2.1.0"
---

# Recap — 对话记录与复盘

将对话提炼为结构化摘要，按日期存档到 `docs/recap_context/`。

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
| `progress` | 调用 `recap:recap-progress-zh` skill |
| `progress update` | 调用 `recap:recap-progress-zh` skill，参数 "update" |
| `proposal` | 调用 `recap:recap-proposal-zh` skill |
| `proposal list` | 调用 `recap:recap-proposal-zh` skill，参数 "list" |
| `proposal <N>` | 调用 `recap:recap-proposal-zh` skill，参数为编号 |
| `context` | 调用 `recap:recap-context-zh` skill |
| `context <N>` | 调用 `recap:recap-context-zh` skill，参数为天数 |
| `context full` | 调用 `recap:recap-context-zh` skill，参数 "full" |
| `silent` 或 `silent on` | 开启静默模式（见下方） |
| `silent off` | 关闭静默模式（见下方） |
| `silent status` | 查看当前静默模式状态 |

当前参数值: $ARGUMENTS

如果参数匹配子 skill，调用对应 skill 后结束。

---

## 静默模式切换（`silent [on|off|status]`）

控制自动 recap（Stop hook）是否静默运行。

1. 根据参数确定操作：
   - `silent` 或 `silent on` → 开启
   - `silent off` → 关闭
   - `silent status` → 仅显示当前值

2. 开启/关闭时：写入 `~/.claude/recap/config.json`：
   ```bash
   mkdir -p ~/.claude/recap
   ```
   读取已有的 `~/.claude/recap/config.json`（如存在），更新 `"silent"` 字段后写回：
   ```json
   {
     "silent": true
   }
   ```
   如果文件已有其他字段，合并而非覆盖。

3. 确认信息：
   - 开启："静默模式已开启。自动 recap 将无感运行。"
   - 关闭："静默模式已关闭。自动 recap 将显示生成过程。"
   - 状态："静默模式当前为 **开启/关闭** 状态。"

---

## 完整总结（无参数）

1. 执行 `date '+%Y-%m-%d %H:%M'` 和 `git status --short`
2. 回顾对话 — 提炼以下内容（某项无内容则省略，不写"无"）：
   - **讨论内容** — 涉及的主题和技术点（2-5 条）
   - **完成工作** — 实际完成的编码、配置、调试等
   - **文件变更** — 从 git status 提取，标注 M/A/D
   - **关键决策** — "为什么选 A 不选 B"，附理由
   - **遗留问题** — 未完成的工作、后续计划
   - **委派任务**（可选）— 如使用了子 agent，列出：`[agent 类型] 完成内容摘要`。同时检查 `docs/recap_context/.agent-activity.jsonl` 中本次会话自动记录的 agent 完成条目，一并纳入。
3. 检查 `docs/recap_context/daily/YYYY/YYYY-MM-DD.md` 是否存在
4. 写入摘要（新建或作为 Session N 追加，用 `---` 分隔）
5. 写入后同步（见下方）

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
6. 写入后同步（见下方）

---

## 状态速查（`status`）

1. 检查 `docs/recap_context/daily/YYYY/YYYY-MM-DD.md` 是否存在
2. 存在 → 显示 session 数量、最近时间、笔记数
3. 不存在 → "今天还没有 recap"

---

## 写入后同步（每次 recap/笔记后执行）

### 1. META 同步

**项目 META** — `docs/recap_context/META.json`：
```json
{
  "project": "<git 根目录名>",
  "totalSessions": "<数量>",
  "lastSession": "<ISO 时间戳>",
  "recentTopics": ["<最新讨论内容>"],
  "remainingIssues": ["<最新遗留问题>"],
  "recapFiles": ["daily/YYYY/YYYY-MM-DD.md", ...],
  "proposals": ["001-xxx.md", ...]
}
```

**全局 META** — `~/.claude/recap/projects/<项目名>.json`：
```bash
mkdir -p ~/.claude/recap/projects
```
```json
{
  "project": "<项目名>",
  "path": "<git 根目录绝对路径>",
  "lastSession": "<ISO 时间戳>",
  "lastRecapFile": "docs/recap_context/daily/YYYY/YYYY-MM-DD.md",
  "remainingIssues": ["<最新遗留问题>"],
  "recentTopics": ["<最新讨论内容>"],
  "sessionCount": "<总数>"
}
```

### 2. DECISIONS.md 自动提取

如果 recap 包含**关键决策**节，将每条决策追加到 `docs/recap_context/DECISIONS.md`：

```markdown
# 决策日志

## YYYY-MM-DD
- 决策 — 理由
```

不存在则创建。在已有日期节下追加，或添加新日期节（最新在上）。

### 3. Git 自动提交

除非 `$RECAP_AUTO_COMMIT` 设为 `false`：

```bash
# 清理已处理的 agent 活动日志
rm -f docs/recap_context/.agent-activity.jsonl
git add docs/recap_context/
git commit -m "recap: YYYY-MM-DD session N summary"
```

---

## INDEX.md 维护

每次写入后更新 `docs/recap_context/INDEX.md`。按月分组（最新在上），日期最新在上。📋 周报 📊 月报 📝 方案。不存在则创建。

## 文件写入

`mkdir -p docs/recap_context/daily/$(date +%Y) docs/recap_context/weekly/$(date +%Y) docs/recap_context/monthly docs/recap_context/proposals`

使用 **Write** tool。WSL2 环境下如不可见改用 Bash heredoc。
