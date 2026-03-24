---
name: recap-projects-zh
description: "显示跨项目状态概览。读取 ~/.claude/recap/projects/ 下的全局 META，展示所有跟踪项目的最新会话和遗留问题。当用户说「项目状态」「跨项目」或执行 '/recap-zh projects' 时触发。"
user-invocable: false
allowed-tools: "Read, Bash, Glob, Grep"
metadata:
  version: "2.0.0"
---

# 跨项目状态

展示 recap 插件跟踪的所有项目状态概览。

## 执行步骤

1. 读取 `~/.claude/recap/projects/*.json` 下的所有文件
2. 解析每个文件的项目元数据
3. 按 `lastSession` 降序排列（最近的在前）
4. 格式化展示
5. 高亮当前项目

## 数据来源

`~/.claude/recap/projects/` 下每个文件代表一个项目：
```json
{
  "project": "项目名",
  "path": "/绝对路径",
  "lastSession": "ISO 时间戳",
  "lastRecapFile": "docs/context/YYYY-MM-DD.md",
  "remainingIssues": ["问题1", "问题2"],
  "recentTopics": ["话题1", "话题2"],
  "sessionCount": 10
}
```

## 输出格式

```
# 跨项目状态

> 当前项目：**<当前项目名>**

| 项目 | 最近会话 | 会话数 | 遗留问题 |
|------|---------|--------|---------|
| **当前项目** ← | 2026-03-24 17:30 | 12 | 2 |
| 其他项目 | 2026-03-23 14:00 | 45 | 3 |

## 活跃项目（24h 内）

### 当前项目
- 最近话题：JWT 认证、数据库 schema
- 遗留：refresh token 逻辑

### 其他项目
- 最近话题：结算流程、支付 API
- 遗留：错误处理、负载测试

## 不活跃项目（>24h）
- **旧项目**（14 天前）— 0 个遗留问题
```

如果 `~/.claude/recap/projects/` 不存在或为空，告知用户暂无跨项目数据，会在 recap 会话后自动生成。
