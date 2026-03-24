---
name: recap-monthly-zh
description: "生成月报，汇总本月所有每日记录和周报。当用户说「月报」「本月总结」或执行 '/recap-zh monthly' 时触发。"
user-invocable: false
allowed-tools: "Read, Write, Edit, Bash, Glob, Grep"
metadata:
  version: "2.0.0"
---

# 月报

汇总本月所有每日记录和周报，生成月度报告。

## 执行步骤

1. 执行 `date '+%Y-%m-%d'` 获取当前日期
2. 确定本月 1 号到今天的范围
3. 读取所有每日记录和周报
4. 生成月报
5. 写入 `docs/context/monthly/YYYY-MM.md`
6. 更新 `docs/context/INDEX.md`

`mkdir -p docs/context/monthly`

## 格式

```markdown
# YYYY 年 MM 月 月报

## 月度概要
- 2-3 句话总结主要成果和方向

## 按周回顾

### 第 1 周 (MM-01 ~ MM-07)
- 要点

### 第 2 周 (MM-08 ~ MM-14)
- 要点

（按实际周列出）

## 关键里程碑
- 重要功能、架构变更

## 技术债务与遗留
- 累积未解决问题
```

## INDEX.md

在对应月份下添加 📊 前缀的条目。
