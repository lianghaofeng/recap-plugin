---
name: recap-weekly-zh
description: "生成周报，汇总本周每日对话记录。当用户说「周报」「本周总结」或执行 '/recap-zh weekly' 时触发。"
user-invocable: false
allowed-tools: "Read, Write, Edit, Bash, Glob, Grep"
metadata:
  version: "2.0.0"
---

# 周报

汇总本周每日对话记录，生成周报。

## 执行步骤

1. 执行 `date '+%Y-%m-%d'` 获取今天日期
2. 计算本周一的日期
3. 读取周一到今天的所有 `docs/recap_context/YYYY-MM-DD.md`
4. 生成周报
5. 写入 `docs/recap_context/weekly/YYYY-WXX.md`（XX = ISO 周数）
6. 更新 `docs/recap_context/INDEX.md`

`mkdir -p docs/recap_context/weekly`

## 格式

```markdown
# YYYY 第 XX 周 周报 (MM-DD ~ MM-DD)

## 本周概要
- 一句话总结主要成果

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
- 后续跟进
```

## INDEX.md

在对应月份下添加 📋 前缀的条目。
