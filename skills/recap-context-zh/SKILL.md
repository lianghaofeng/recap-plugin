---
name: recap-context-zh
description: "加载项目上下文。从 recap 历史恢复 META、进度、决策和近期对话摘要，实现跨会话连续性。用法：'/recap-zh context'、'/recap-zh context 7'、'/recap-zh context full'。"
user-invocable: false
allowed-tools: "Read, Bash, Glob, Grep"
metadata:
  version: "1.0.0"
---

# 上下文加载器 — 项目上下文恢复

从 recap 历史中恢复项目上下文，使 agent 在新会话中快速获得完整的项目认知。

## 参数

| $ARGUMENTS | 操作 |
|---|---|
| 空或 `3` | 加载 META + PROGRESS 摘要 + 最近 3 天 recap 要点 |
| `<N>`（数字） | 加载 META + PROGRESS 摘要 + 最近 N 天 recap 要点 |
| `full` | 加载 META + PROGRESS + DECISIONS + 最近 3 天完整 recap |

当前参数值: $ARGUMENTS

---

## 执行步骤

### 第 1 步：项目 META

读取 `docs/recap_context/META.json`（如存在），输出为：

```
📌 项目: <名称>
   会话总数: <数量> | 最近: <日期>
   近期话题: <话题>
   遗留问题: <问题>
```

如果 META.json 不存在，提示："未找到项目上下文。请先执行 /recap-zh 初始化。"

### 第 2 步：进度快照

读取 `docs/recap_context/PROGRESS.md`（如存在），只提取：
- **当前焦点** / **Current Focus** 节（前 10 行）
- **后续步骤** / **Next Steps** 节（前 10 行）

不存在则跳过。

### 第 3 步：近期对话

从参数确定 N（默认 3）。

```bash
ls -1 docs/recap_context/????-??-??.md 2>/dev/null | sort -r | head -N
```

遍历每个文件（最新在前）：

- **普通模式**（默认）：只提取 `### 讨论内容` / `### Topics` 和 `### 遗留问题` / `### Remaining Issues` 节
- **完整模式**（`full` 参数）：读取全部内容

### 第 4 步：决策记录（仅完整模式）

如果参数为 `full`，读取 `docs/recap_context/DECISIONS.md`，显示最近 10 条决策。

### 第 5 步：Agent 活动

如果 `docs/recap_context/.agent-activity.jsonl` 存在，读取并显示今天的条目：

```
🤖 近期 agent 活动:
   - [test-runner] 17:31 — 已完成
   - [Explore] 17:35 — 已完成
```

---

## 输出格式

以结构化、简洁的格式呈现所有加载的上下文。利用这些信息指导本次会话的后续响应——不要简单地输出原始文件内容。

加载完成后确认："上下文已加载，可以继续工作。"
