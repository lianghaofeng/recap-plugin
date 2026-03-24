---
name: recap-zh
description: "对话记录与每日复盘工具。按日期将对话摘要存档到 docs/context/，支持完整对话总结、周报/月报和快速笔记。当用户说「recap」「总结一下」「复盘」「今天做了什么」「周报」「月报」或想记笔记时触发。对话结束时如果没有生成摘要会自动提醒。"
user-invocable: true
allowed-tools: "Read, Write, Edit, Bash, Glob, Grep"
metadata:
  version: "1.0.0"
---

# Recap — 对话记录与复盘

将对话提炼为结构化摘要，按日期存档到 `docs/context/`，方便回顾和追踪工作进展。

## 参数解析

| $ARGUMENTS | 操作 |
|---|---|
| 空 | 生成当次对话的完整总结 |
| `weekly` | 汇总本周对话，生成周报 |
| `monthly` | 汇总本月对话，生成月报 |
| `note <内容>` | 追加一条手动笔记 |
| `status` | 查看今天的 recap 状态 |

当前参数值: $ARGUMENTS

---

## 操作 1：完整总结（无参数）

### 执行步骤

1. **获取当前时间**：`date '+%Y-%m-%d %H:%M'`
2. **获取文件变更**：`git status --short`
3. **回顾对话** — 提炼以下内容：
   - **讨论内容** — 涉及的主题和技术点（2-5 条）
   - **完成工作** — 实际完成的编码、配置、调试等
   - **文件变更** — 从 git status 提取，标注 M(修改)/A(新增)/D(删除)
   - **关键决策** — "为什么选 A 不选 B"，附简要理由
   - **遗留问题** — 未完成的工作、已知 bug、后续计划
   
   某项没有内容时省略该节，不写"无"。

4. **检查当天文件**：`ls docs/context/YYYY-MM-DD.md 2>/dev/null`
5. **写入摘要**
6. **更新 INDEX.md**

### 新建文件格式

```markdown
# YYYY-MM-DD 对话记录

## Session 1 (HH:MM)

### 讨论内容
- 主题 1
- 主题 2

### 完成工作
- 工作 1
- 工作 2

### 文件变更
- M path/to/file.cpp
- A path/to/new_file.hpp

### 关键决策
- 决策 — 理由

### 遗留问题
- 问题描述
```

### 追加到已有文件

读取已有文件确定下一个 Session 编号，用 `---` 分隔追加：

```markdown

---

## Session N (HH:MM)

### 讨论内容
...
```

---

## 操作 2：周报（`weekly`）

### 执行步骤

1. 获取本周一到今天的日期范围
2. 读取范围内所有 `docs/context/YYYY-MM-DD.md`
3. 写入 `docs/context/weekly/YYYY-WXX.md`（XX = ISO 周数）
4. 更新 INDEX.md

### 格式

```markdown
# YYYY 第 XX 周 周报 (MM-DD ~ MM-DD)

## 本周概要
- 一句话总结

## 每日摘要

### 周一 (MM-DD)
- 要点

### 周二 (MM-DD)
- 要点

（无记录的跳过）

## 本周完成
- 汇总完成工作

## 遗留与下周计划
- 未完成事项
```

---

## 操作 3：月报（`monthly`）

### 执行步骤

1. 确定本月 1 号到今天
2. 读取所有每日记录和周报
3. 写入 `docs/context/monthly/YYYY-MM.md`
4. 更新 INDEX.md

### 格式

```markdown
# YYYY 年 MM 月 月报

## 月度概要
- 2-3 句话总结

## 按周回顾

### 第 1 周 (MM-01 ~ MM-07)
- 要点

## 关键里程碑
- 重要功能、架构变更

## 技术债务与遗留
- 累积未解决问题
```

---

## 操作 4：手动笔记（`note <内容>`）

### 执行步骤

1. 获取当前时间
2. 提取笔记内容（去掉 "note " 前缀）
3. 检查当天文件：
   - **不存在** → 创建文件，含标题和笔记节
   - **有 `## 笔记` 节** → 在其下追加
   - **无 `## 笔记` 节** → 在文件末尾添加

### 格式

```markdown
## 笔记
- [HH:MM] 内容
```

---

## 操作 5：状态速查（`status`）

1. 检查 `docs/context/YYYY-MM-DD.md` 是否存在
2. 存在：显示 session 数量、最近 session 时间、笔记数
3. 不存在：显示"今天还没有 recap"

---

## INDEX.md 维护

每次写入后更新 `docs/context/INDEX.md`。

### 格式

```markdown
# 对话记录索引

## 2026-03
- [03-24](2026-03-24.md) - 当天主要内容描述
- [03-23](2026-03-23.md) - 描述
- 📋 [第 12 周周报](weekly/2026-W12.md)
- 📊 [3 月月报](monthly/2026-03.md)
```

### 规则

- 按月分组，最新月份在上
- 每天一行，最新日期在上
- 已有条目更新描述
- 📋 周报 📊 月报
- INDEX.md 不存在则创建

---

## 文件写入

需要时先创建目录：`mkdir -p docs/context/weekly docs/context/monthly`

使用 **Write** tool 写入文件。WSL2 环境下如果 Write tool 创建的文件 Windows 不可见，改用 Bash heredoc：

```bash
cat > docs/context/YYYY-MM-DD.md << 'RECAP_EOF'
内容
RECAP_EOF
```

---

## 目录结构

```
docs/context/
├── INDEX.md
├── 2026-03-24.md
├── weekly/
│   └── 2026-W12.md
└── monthly/
    └── 2026-03.md
```
