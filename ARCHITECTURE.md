# Recap Plugin — 架构全览

> 版本: 2.2.0 | 更新: 2026-03-24

## 版本演进

### v1.0 — 基础 Recap (commit: `5f84d57`)

最小可用版本，两个 skill + 两个 command。

```
新增文件:
  commands/recap.md, recap-zh.md, status.md, status-zh.md
  skills/recap/SKILL.md, recap-zh/SKILL.md
  .claude-plugin/plugin.json, marketplace.json
```

**能力**: 手动生成对话摘要、快速笔记、状态查看。
**输出**: `docs/context/YYYY-MM-DD.md`（每日对话日志）。

---

### v1.1 — Hook 自动化 (commit: `6badb87`)

将 hook 从 settings.json 移到插件级 `hooks/hooks.json`，实现安装即自动激活。

```
新增文件:
  hooks/hooks.json
修改:
  skills/recap/SKILL.md, recap-zh/SKILL.md
```

**能力**: Stop hook 自动提醒生成 recap。

---

### v2.0 — 全局插件 + 跨项目 (commit: `a2395c4`)

核心架构升级：两级 META 系统、skill 拆分、跨项目感知。

```
新增文件:
  skills/recap-weekly/SKILL.md, recap-weekly-zh/SKILL.md
  skills/recap-monthly/SKILL.md, recap-monthly-zh/SKILL.md
  skills/recap-search/SKILL.md, recap-search-zh/SKILL.md
  skills/recap-projects/SKILL.md, recap-projects-zh/SKILL.md
  commands/projects.md, projects-zh.md
修改:
  hooks/hooks.json — 新增 UserPromptSubmit hook
  skills/recap/SKILL.md — 新增参数路由表
  .claude-plugin/plugin.json — 版本 2.0.0
```

**能力**: 周报、月报、历史搜索、跨项目状态、错误恢复、每日首次提醒。

---

### v2.1 — 方案 + 进度 + 决策 (commit: `2710d7e`)

知识管理层：设计方案文档化、进度追踪、决策日志自动提取。

```
新增文件:
  skills/recap-progress/SKILL.md, recap-progress-zh/SKILL.md
  skills/recap-proposal/SKILL.md, recap-proposal-zh/SKILL.md
  commands/proposal.md, proposal-zh.md
修改:
  hooks/hooks.json — Stop hook 增强（pending.log + 错误恢复）
  skills/recap/SKILL.md — 路由表扩展 + DECISIONS.md 自动提取 + 委派任务节
```

**能力**: `/recap progress`、`/proposal`、DECISIONS.md 自动维护、委派任务模板。

---

### v2.2 — 上下文加载 + Agent 协作 (commit: `ee40e34`)

上下文连续性：自动/手动上下文恢复、子 agent 活动追踪。

```
新增文件:
  skills/recap-context/SKILL.md, recap-context-zh/SKILL.md
修改:
  hooks/hooks.json — 新增 SubagentStop hook + UserPromptSubmit 增强（META 加载）
  skills/recap/SKILL.md — 路由表 + activity log 消费
  .claude-plugin/plugin.json — 版本 2.2.0
```

**能力**: 自动上下文恢复、`/recap context`、子 agent 活动日志、协作记录自动填充。

---

## 当前架构

### 文件结构

```
recap-plugin/                              # 插件根目录
├── .claude-plugin/
│   ├── plugin.json                        # 插件元数据 (name, version, description)
│   └── marketplace.json                   # 市场发布配置
├── hooks/
│   └── hooks.json                         # 3 个生命周期 hook
├── commands/                              # 8 个用户命令（薄包装层）
│   ├── recap.md          → recap skill
│   ├── recap-zh.md       → recap-zh skill
│   ├── status.md         → recap skill (status)
│   ├── status-zh.md      → recap-zh skill (status)
│   ├── projects.md       → recap-projects skill
│   ├── projects-zh.md    → recap-projects-zh skill
│   ├── proposal.md       → recap-proposal skill
│   └── proposal-zh.md    → recap-proposal-zh skill
├── skills/                                # 16 个 skill 模块
│   ├── recap/SKILL.md                     # 主入口 + 路由器 (EN)
│   ├── recap-zh/SKILL.md                  # 主入口 + 路由器 (ZH)
│   ├── recap-weekly/SKILL.md              # 周报聚合
│   ├── recap-weekly-zh/SKILL.md
│   ├── recap-monthly/SKILL.md             # 月报聚合
│   ├── recap-monthly-zh/SKILL.md
│   ├── recap-search/SKILL.md              # 历史搜索
│   ├── recap-search-zh/SKILL.md
│   ├── recap-projects/SKILL.md            # 跨项目状态
│   ├── recap-projects-zh/SKILL.md
│   ├── recap-progress/SKILL.md            # 进度追踪
│   ├── recap-progress-zh/SKILL.md
│   ├── recap-proposal/SKILL.md            # 设计方案
│   ├── recap-proposal-zh/SKILL.md
│   ├── recap-context/SKILL.md             # 上下文加载
│   └── recap-context-zh/SKILL.md
├── README.md
└── LICENSE
```

### 运行时数据结构

```
项目级 (docs/context/):                     全局级 (~/.claude/recap/):
├── META.json              ←── 双向同步 ──→ projects/<project>.json
├── PROGRESS.md                              pending.log
├── DECISIONS.md
├── INDEX.md
├── .agent-activity.jsonl  (临时)
├── YYYY-MM-DD.md          (日志)
├── weekly/YYYY-WXX.md     (周报)
├── monthly/YYYY-MM.md     (月报)
└── proposals/NNN-*.md     (方案)
```

---

## 调用链

### 1. 会话开始 — UserPromptSubmit Hook

```
用户输入 prompt
    │
    ▼
UserPromptSubmit hook 触发（每天首次）
    │
    ├── 1. 标记今日已执行 (/tmp/recap_session_YYYYMMDD)
    │
    ├── 2. 错误恢复检查
    │   └── 读取 ~/.claude/recap/pending.log
    │       └── 如果当前项目有 PENDING 记录 且 recap 文件不存在
    │           └── 输出: [recap-recovery] 提醒生成补救 recap
    │
    ├── 3. 自动上下文加载 (v2.2 新增)
    │   ├── 读取 docs/context/META.json → 输出项目快照
    │   └── 读取 docs/context/PROGRESS.md → 提取当前焦点 + 后续步骤
    │
    ├── 4. 遗留问题提醒
    │   └── 读取最新 docs/context/YYYY-MM-DD.md
    │       └── 提取 "### Remaining Issues" / "### 遗留问题" 下的条目
    │
    └── 5. 跨项目状态
        └── 遍历 ~/.claude/recap/projects/*.json
            └── 过滤 24h 内活跃的其他项目，显示有 open issues 的
```

### 2. 会话中 — 命令调用

```
用户输入 /recap [args]
    │
    ▼
commands/recap.md (薄包装)
    │
    └── "调用 recap:recap skill，参数为 $ARGUMENTS"
        │
        ▼
    skills/recap/SKILL.md (路由器)
        │
        ├── 无参数 ──────────→ 执行「完整总结」流程
        │                       │
        │                       ├── 1. date + git status
        │                       ├── 2. 提炼 Topics/Work Done/Files/Decisions/Issues
        │                       ├── 3. 读取 .agent-activity.jsonl → 填充委派任务
        │                       ├── 4. 写入 docs/context/YYYY-MM-DD.md
        │                       └── 5. Post-Write Sync
        │                             ├── META 同步 (项目级 + 全局级)
        │                             ├── DECISIONS.md 自动提取
        │                             ├── 清理 .agent-activity.jsonl
        │                             ├── git add + commit
        │                             └── INDEX.md 更新
        │
        ├── "weekly" ────────→ recap:recap-weekly skill
        │                       └── 读 Mon~Today 的日志 → weekly/YYYY-WXX.md
        │
        ├── "monthly" ───────→ recap:recap-monthly skill
        │                       └── 读当月所有日志+周报 → monthly/YYYY-MM.md
        │
        ├── "note <text>" ───→ 执行「快速笔记」流程
        │                       └── 在当日文件追加 [HH:MM] 内容
        │
        ├── "status" ────────→ 执行「状态速查」流程
        │                       └── 显示当日 session 数、笔记数
        │
        ├── "search <q>" ────→ recap:recap-search skill
        │                       └── grep 全部 docs/context/ + META.json
        │
        ├── "projects" ──────→ recap:recap-projects skill
        │                       └── 读 ~/.claude/recap/projects/*.json → 表格
        │
        ├── "progress" ──────→ recap:recap-progress skill
        │   "progress update"    └── 显示/更新 PROGRESS.md
        │
        ├── "proposal" ──────→ recap:recap-proposal skill
        │   "proposal list"      └── 创建/列出/查看 proposals/NNN-*.md
        │   "proposal N"
        │
        └── "context" ───────→ recap:recap-context skill  (v2.2 新增)
            "context N"          └── 加载 META + PROGRESS + 近 N 天摘要
            "context full"           + (full 模式) DECISIONS + 完整 recap
```

### 3. 子 Agent 完成 — SubagentStop Hook (v2.2 新增)

```
主 Agent 委派子 Agent（Explore/test-runner/等）
    │
    ▼
子 Agent 完成
    │
    ▼
SubagentStop hook 触发
    │
    ├── 检查 docs/context/ 是否存在
    │   ├── 不存在 → 跳过（项目未使用 recap）
    │   └── 存在 ↓
    │
    ├── 追加一行到 docs/context/.agent-activity.jsonl:
    │   {"ts":"ISO","agent":"<description>","action":"completed"}
    │
    └── 输出: [recap] Sub-agent '<name>' completed. Activity logged.
```

### 4. 会话结束 — Stop Hook

```
对话即将结束
    │
    ▼
Stop hook 触发
    │
    ├── 检查今日 recap 文件是否在 180 秒内更新过
    │   ├── 是 → exit 0（已有新鲜 recap，跳过）
    │   └── 否 ↓
    │
    ├── 写入 PENDING 标记到 ~/.claude/recap/pending.log
    │   格式: PENDING:<timestamp>:<project-path>
    │
    └── 输出提醒: [auto-recap] 请现在生成对话摘要...
        包含完整指令: 写 recap → 更新 META → 更新 DECISIONS → git commit
```

---

## 数据流图

```
┌─────────────────────────────────────────────────────────────────┐
│                        Session Lifecycle                         │
│                                                                  │
│  ┌──────────┐    ┌──────────────┐    ┌───────────┐              │
│  │  Start   │    │   Running    │    │   Stop    │              │
│  │  Hook    │    │              │    │   Hook    │              │
│  └────┬─────┘    └──────┬───────┘    └─────┬─────┘              │
│       │                 │                   │                    │
│       ▼                 ▼                   ▼                    │
│  ┌─────────┐    ┌──────────────┐    ┌───────────┐              │
│  │META.json│    │ SubagentStop │    │pending.log│              │
│  │PROGRESS │◄───┤   ↓          │    └───────────┘              │
│  │遗留问题  │    │.agent-       │          │                    │
│  │跨项目   │    │ activity.jsonl│          ▼                    │
│  └─────────┘    └──────────────┘    ┌───────────┐              │
│       │                 │           │ 提醒生成  │              │
│       │                 │           │  recap    │              │
│       │                 ▼           └─────┬─────┘              │
│       │         ┌──────────────┐          │                    │
│       │         │  /recap      │◄─────────┘                    │
│       │         │  (手动/自动) │                                │
│       │         └──────┬───────┘                                │
│       │                │                                        │
│       │                ▼                                        │
│       │    ┌───────────────────────┐                            │
│       │    │     Post-Write Sync   │                            │
│       │    ├───────────────────────┤                            │
│       │    │ • META.json 更新      │──→ docs/context/META.json  │
│       │    │ • 全局 META 更新      │──→ ~/.claude/recap/projects/│
│       │    │ • DECISIONS.md 追加   │──→ docs/context/DECISIONS.md│
│       │    │ • INDEX.md 更新       │──→ docs/context/INDEX.md   │
│       │    │ • 清理 activity log   │                            │
│       │    │ • git add + commit    │                            │
│       │    └───────────────────────┘                            │
│       │                                                         │
│       │    ┌───────────────────────┐                            │
│       └───►│  /recap context      │                             │
│            │  (手动上下文恢复)     │                             │
│            │  读取 META+PROGRESS   │                             │
│            │  +近 N 天 recap 摘要  │                             │
│            └───────────────────────┘                            │
└─────────────────────────────────────────────────────────────────┘
```

---

## Skill 模块一览

| Skill | 版本 | 触发方式 | 工具权限 | 核心职责 |
|-------|------|---------|---------|---------|
| `recap` | 2.1.0 | `/recap` | Read,Write,Edit,Bash,Glob,Grep | 路由器 + 完整总结 + 笔记 + 状态 |
| `recap-zh` | 2.1.0 | `/recap-zh` | 同上 | 中文版路由器 |
| `recap-weekly` | 2.0.0 | `/recap weekly` | 同上 | 周报聚合 |
| `recap-monthly` | 2.0.0 | `/recap monthly` | 同上 | 月报聚合 |
| `recap-search` | 2.0.0 | `/recap search <q>` | Read,Bash,Glob,Grep | 历史搜索 |
| `recap-projects` | 2.0.0 | `/projects` | Read,Bash,Glob,Grep | 跨项目状态表 |
| `recap-progress` | 2.1.0 | `/recap progress` | Read,Write,Edit,Bash,Glob,Grep | 进度文档维护 |
| `recap-proposal` | 2.1.0 | `/proposal` | 同上 | 设计方案管理 |
| `recap-context` | 1.0.0 | `/recap context` | Read,Bash,Glob,Grep | 上下文恢复 |

每个 EN skill 有对应 `-zh` 变体，共 **16 个 skill**。中文版功能完全对等，仅输出语言不同。

## Hook 一览

| Hook | 触发时机 | 核心逻辑 |
|------|---------|---------|
| `Stop` | 对话结束 | 检查 recap 新鲜度 → 写 pending.log → 提醒生成 recap |
| `UserPromptSubmit` | 每天首次 prompt | 错误恢复 → META 加载 → PROGRESS 摘要 → 遗留问题 → 跨项目状态 |
| `SubagentStop` | 子 agent 完成 | 写 .agent-activity.jsonl → 输出提示 |

## Command → Skill 映射

```
/recap        → recap:recap              (参数透传)
/recap-zh     → recap:recap-zh           (参数透传)
/status       → recap:recap + "status"
/status-zh    → recap:recap-zh + "status"
/projects     → recap:recap-projects
/projects-zh  → recap:recap-projects-zh
/proposal     → recap:recap-proposal     (参数透传)
/proposal-zh  → recap:recap-proposal-zh  (参数透传)
```

命令文件是**薄包装层**，只做一件事：`调用 recap:<skill-name> skill，参数为 $ARGUMENTS`。

## 关键设计原则

1. **渐进式上下文加载** — skill 按需加载，不预加载全部 16 个（每个 40-120 行），避免占用上下文窗口
2. **两级 META** — 项目级 JSON 管项目内聚合，全局级 JSON 管跨项目感知，无文件锁竞争
3. **Hook 驱动** — 3 个生命周期 hook 覆盖会话起止和子 agent，无需用户手动触发
4. **临时 vs 持久** — `.agent-activity.jsonl` 是临时数据（recap 后清理），有价值的信息提炼进持久文档
5. **双语对等** — EN/ZH 功能完全一致，通过 `-zh` 后缀区分，共享同一数据格式
6. **错误恢复** — pending.log + 每日标记，确保 recap 不丢失
