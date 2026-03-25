---
name: recap-search-zh
description: "搜索历史对话记录。当用户说「搜索记录」「查找历史」或执行 '/recap-zh search <关键词>' 时触发。示例：'/recap-zh search 认证'、'/recap-zh search 数据库迁移'。"
user-invocable: false
allowed-tools: "Read, Bash, Glob, Grep"
metadata:
  version: "2.0.0"
---

# 记录搜索

按关键词搜索历史对话记录。

## 参数

搜索关键词为 $ARGUMENTS 中 "search " 之后的内容。

## 执行步骤

1. 提取搜索关键词
2. 搜索所有 recap 文件：
   ```bash
   grep -r -i -n "<关键词>" docs/recap_context/ --include="*.md" | head -30
   ```
3. 同时搜索 META.json：
   ```bash
   grep -i "<关键词>" docs/recap_context/META.json 2>/dev/null
   ```
4. 按日期分组展示结果，附上下文
5. 总结：哪些会话讨论了该主题、关键发现

## 输出格式

```
### "<关键词>" 搜索结果

**在 N 个文件中找到：**

#### YYYY-MM-DD
- [Session 2] 在...的上下文中讨论了<主题>
- 相关摘录："..."

#### YYYY-MM-DD
- [Session 1] 与<主题>相关...

**总结：** 该主题在 N 个会话中被讨论，主要涉及...
```

未找到结果时，告知用户并建议其他搜索词。
