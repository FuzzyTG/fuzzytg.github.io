---
title: "How I Built a Real AI Secretary — Not a Chatbot, a System"
date: 2026-03-13 23:30:00 +0800
categories: [AI, Product]
tags: [ai-agent, system-design, task-management, openai]
---

[🇬🇧 English](#-english-version) | [🇨🇳 中文](#-中文版本)

---

## 🇬🇧 English Version

# How I Built a Real AI Secretary — Not a Chatbot, a System

**TL;DR:** Most people use AI assistants as fancy search engines. I built mine into a full secretary system: it sends emails, schedules meetings, tracks tasks, follows up automatically, and manages everything through Markdown files on GitHub. Here's the system design.

---

Everyone's talking about AI agents. Most demos show a chatbot that can "search the web" or "summarize a document." That's not a secretary. That's a search engine with personality.

A real secretary does things *for* you. They send emails on your behalf, chase people who haven't replied, remind you about deadlines, and keep everything organized without being asked. I wanted that — not a chatbot, but an operational system.

Here's how I designed it.

## The Core Insight: Markdown is Your Database

The first design decision was the most important: **use plain Markdown files as the single source of truth.**

Every task is a Markdown file with YAML frontmatter:

```yaml
---
id: TASK-20260310-restaurant-booking
title: Book restaurant for team dinner
category: personal
status: in_progress
created: 2026-03-10T09:00:00+08:00
deadline: 2026-03-14T18:00:00+08:00
owner: Lead Agent
agentmail_threads:
  - thread_id: "a3f8c2d1-..."
    purpose: "inquiry"
    contact: "Host"
    email: "reservations@restaurant.com"
    status: "awaiting_reply"
    sent_at: "2026-03-10T10:30:00+08:00"
    follow_up_count: 0
cron_jobs:
  - job_id: "abc123"
    type: "deadline_reminder"
calendar_event_ids:
  deadline: "uid-xxx"
---

# Book restaurant for team dinner

## Subtasks

| # | Task | Owner | Due | Status |
|---|------|-------|-----|--------|
| 1 | Research options | Lead Agent | 2026-03-11 | done |
| 2 | Send inquiry email | Lead Agent | 2026-03-11 | done |
| 3 | Confirm reservation | Lead Agent | 2026-03-13 | pending |
```

Why Markdown?

1. **Version controlled.** Every change is a Git commit. You get full audit trails for free.
2. **Human readable.** Open the file, see the status. No dashboards needed.
3. **Agent readable.** Frontmatter is structured YAML — agents can parse and update it programmatically.
4. **Portable.** No vendor lock-in. No database to maintain. It's just files.

The frontmatter is the real innovation. It's where the agent stores **operational metadata**: email thread IDs, cron job references, calendar event UIDs, task status. The Markdown body is for humans. The frontmatter is for the system.

## Architecture: Flows, Not Features

Instead of building "features," I designed **flows** — event-driven pipelines that trigger automatically:

```
User conversation → Flow 0 (Triage) → Flow A (Create & Execute)
                                         ↓
Heartbeat (hourly) → Flow B (Calendar Scan)
                   → Flow D (Task Tracking)
                   → Flow F (Inbox → Task Progression)
                                         ↓
Cron (daily 6AM) → Flow C (Daily Briefing)
                                         ↓
User approval → Flow E (Archive)
```

### Flow 0: Triage — Don't Over-Engineer

Not every request needs a task file. The first thing the system does is classify complexity:

| Level | Signal | Action |
|-------|--------|--------|
| Simple (80%) | Single action, no deadline | Just do it. No file. |
| Professional (15%) | 2-3 subtasks, one specialist | Create file, simplified flow |
| Complex (5%) | 4+ subtasks, multi-agent | Full pipeline with timeline |

This prevents the system from creating bureaucracy for "what's the weather today."

### Flow A: Task Creation — The Full Pipeline

For complex tasks, the system:

1. Extracts deadline and requirements from conversation
2. Breaks down into subtasks with agent assignments
3. Calculates timeline (working backwards from deadline)
4. Presents plan for human approval
5. On confirmation:
   - Creates the task Markdown file
   - Syncs to calendar
   - Assigns subtasks to specialist agents
   - Sends external emails if needed
   - Sets up cron reminders
   - Commits to Git

The key insight: **every external action gets tracked back in the frontmatter.** Send an email? Store the thread ID. Create a cron job? Store the job ID. Sync to calendar? Store the event UID.

This makes the task file a **complete record** of everything that happened.

### Flow F: The Inbox-Task Bridge

This is where it gets interesting. When the system checks the inbox and finds a new email, it doesn't just notify you. It:

1. Extracts the Thread ID from the email
2. Scans all active task files' `agentmail_threads` frontmatter
3. **Matches the email to a task**
4. Based on the thread's `purpose`, decides what to do:

| Purpose | Reply Content | Auto Action |
|---------|--------------|-------------|
| `scheduling` | Picked a time | Parse → draft confirmation with calendar invite |
| `scheduling` | Vague reply | Notify human with suggestions |
| `confirmation` | Confirmed | Update status, notify human |
| `inquiry` | Any reply | Notify human with summary |

The email doesn't just sit in an inbox — it **drives the task forward.**

## The Thread-Task Pattern

This is the most powerful design pattern in the system. Here's how it works:

**When sending an email:**
```yaml
agentmail_threads:
  - thread_id: "a3f8c2d1-..."
    purpose: "scheduling"
    contact: "Candidate"
    email: "candidate@email.com"
    status: "awaiting_reply"
    sent_at: "2026-03-10T10:30:00+08:00"
    follow_up_count: 0
```

**When a reply arrives (via hourly inbox check):**
```yaml
    status: "replied"  # auto-updated by Flow F
```

**If no reply after 4 hours:**
```yaml
    follow_up_count: 1  # auto-incremented after sending follow-up
    sent_at: "2026-03-10T14:30:00+08:00"  # updated to follow-up time
```

The system knows:
- **Who** you're waiting on
- **How long** you've been waiting
- **Whether** a follow-up has been sent
- **When** to escalate to the human

All stored in plain YAML. All version controlled. All queryable by the agent.

## Multi-Agent Delegation

The system uses specialized agents for different domains:

```
Lead Agent (orchestrator)
├── Finance Agent — budgets, compensation research, cost analysis
└── Tech Agent — technical research, feasibility studies, code review
```

When a task needs specialist input, the Lead Agent:
1. Sends the subtask via inter-agent messaging
2. Records the assignment in the task file
3. Tracks completion via heartbeat checks
4. Integrates results back into the main task

The task file is the **single source of truth** — even in multi-agent scenarios. Every agent writes their output back to the same file or linked artifacts.

## Heartbeat: The Proactive Engine

The system doesn't wait for you to ask "any updates?" Every hour, the heartbeat triggers:

1. **Calendar scan** — any new events in the next 7 days that need preparation?
2. **Task tracking** — any overdue subtasks? Any deadlines approaching?
3. **Inbox check** — any new emails? Do they match active tasks?

If something needs attention, it reaches out. If nothing does, it stays silent.

This is the difference between a chatbot (reactive) and a secretary (proactive).

## Calendar Integration: ICS as the Universal Protocol

For calendar invites, the system generates standard ICS files:

- All times in **UTC** (`DTSTART:20260316T070000Z`) for maximum compatibility
- Supports `METHOD:REQUEST` (create) and `METHOD:CANCEL` (cancel/reschedule)
- Attached to emails so recipients' calendar apps auto-create events

For the human's own calendar, events sync via CalDAV for personal items, or ICS email attachments for work calendars.

## The CRON Protocol: Mechanical Guarantees

Deadlines can't rely on the agent "remembering." The system creates **actual cron jobs** as mechanical reminders:

```json
{
  "schedule": { "kind": "at", "at": "2026-03-14T08:00:00+08:00" },
  "payload": { "text": "⏰ Reminder: Restaurant booking deadline is today 6PM" },
  "deleteAfterRun": true
}
```

Cron job IDs are stored in the task frontmatter. If the task is rescheduled, old cron jobs are deleted and new ones created. If the check script detects missing cron jobs, it raises a warning.

**The principle: deadlines need mechanical triggers, not memory.**

## What Makes This a "Secretary" and Not a "Chatbot"

| Chatbot | Secretary System |
|---------|-----------------|
| Answers when asked | Acts proactively (heartbeat) |
| Forgets between sessions | Persists via Markdown files + Git |
| Can't take external action | Sends emails, creates calendar events |
| One-shot interactions | Tracks multi-step tasks over days/weeks |
| No follow-up | Auto-follows up after configurable timeout |
| No accountability | Full audit trail in Git history |

## The Design Principles

1. **Markdown is the database.** Structured frontmatter for machines, readable body for humans.
2. **Flows, not features.** Event-driven pipelines that compose together.
3. **Track everything in the task file.** Thread IDs, cron jobs, calendar UIDs — one file, complete picture.
4. **Mechanical guarantees over memory.** Cron jobs > "I'll remember."
5. **Human in the loop, not in the weeds.** The system drafts, the human approves.
6. **Git as audit trail.** Every state change is a commit.

## Getting Started

The entire system runs on:
- **OpenClaw** — the AI agent framework (handles messaging, heartbeat, cron, multi-agent)
- **GitHub** — stores task files, provides version control
- **AgentMail** (or similar) — email sending/receiving
- **CalDAV / ICS** — calendar integration
- **Markdown + YAML frontmatter** — the "database"

No Notion. No Jira. No Trello. Just files, Git, and an agent that knows how to use them.

---

The real unlock isn't making AI "smarter." It's giving it **operational infrastructure** — the ability to persist state, take external actions, and follow up without being asked. That's what turns a chatbot into a secretary.

---

## 🇨🇳 中文版本

# 如何构建一个真正的 AI 秘书系统 — 不是聊天机器人，而是一套系统

**一句话总结：** 大多数人把 AI 助手当高级搜索引擎用。我把它搭建成了一套完整的秘书系统：代发邮件、排日程、追任务进度、自动催回复，所有数据用 Markdown 文件 + GitHub 管理。以下是系统设计。

---

所有人都在聊 AI Agent。大多数 demo 展示的是一个能"搜网页"或"总结文档"的聊天机器人。那不是秘书，那是有性格的搜索引擎。

真正的秘书会**替你做事**：代你发邮件、催没回复的人、提醒你 deadline、在你没开口之前就把事情理好。我想要的就是这个 —— 不是聊天机器人，而是一套运营系统。

以下是我的设计。

## 核心洞察：Markdown 就是你的数据库

第一个设计决策也是最重要的：**用纯 Markdown 文件作为唯一数据源。**

每个任务就是一个带 YAML frontmatter 的 Markdown 文件：

```yaml
---
id: TASK-20260310-restaurant-booking
title: 预订团队聚餐餐厅
category: personal
status: in_progress
created: 2026-03-10T09:00:00+08:00
deadline: 2026-03-14T18:00:00+08:00
owner: Lead Agent
agentmail_threads:
  - thread_id: "a3f8c2d1-..."
    purpose: "inquiry"
    contact: "餐厅前台"
    email: "reservations@restaurant.com"
    status: "awaiting_reply"
    sent_at: "2026-03-10T10:30:00+08:00"
    follow_up_count: 0
cron_jobs:
  - job_id: "abc123"
    type: "deadline_reminder"
---
```

为什么选 Markdown？

1. **版本控制。** 每次变更都是一个 Git commit，天然拥有完整审计轨迹。
2. **人类可读。** 打开文件就能看到状态，不需要仪表盘。
3. **Agent 可读。** Frontmatter 是结构化 YAML，Agent 可以程序化解析和更新。
4. **无厂商锁定。** 没有数据库要维护，就是文件而已。

Frontmatter 才是真正的创新。它存储着**运营元数据**：邮件 thread ID、cron job 引用、日历事件 UID、任务状态。Markdown 正文是给人看的，frontmatter 是给系统看的。

## 架构：Flow 驱动，而非功能驱动

我没有构建"功能"，而是设计了**流程（Flow）**—— 事件驱动的管道，自动触发：

```
用户对话 → Flow 0 (分级) → Flow A (创建 & 执行)
                              ↓
心跳检测（每小时）→ Flow B (日历扫描)
                 → Flow D (任务追踪)
                 → Flow F (收件箱 → 任务推进)
                              ↓
定时任务（每天 6AM）→ Flow C (每日简报)
                              ↓
用户审批 → Flow E (归档)
```

### Flow 0：分级 —— 别过度工程

不是每个请求都需要一个任务文件。系统首先评估复杂度：

| 级别 | 信号 | 处理 |
|------|------|------|
| 简单 (80%) | 单一动作，无 deadline | 直接做，不建文件 |
| 专业 (15%) | 2-3 个子任务，需要一个专家 | 建文件，简化流程 |
| 复杂 (5%) | 4+ 个子任务，多 agent 协作 | 完整管道 + 排期 |

这避免了系统为"今天天气怎么样"创建官僚流程。

### Flow A：任务创建 —— 完整管道

对于复杂任务：

1. 从对话中提取 deadline 和需求
2. 拆解子任务并分配给不同 agent
3. 从 deadline 倒推计算时间线
4. 展示方案，等待人类确认
5. 确认后：
   - 创建任务 Markdown 文件
   - 同步日历
   - 分配子任务给专家 agent
   - 如需要，发送外部邮件
   - 创建 cron 提醒
   - 提交 Git

关键洞察：**每个外部操作都回写到 frontmatter。** 发了邮件？存 thread ID。创建了 cron？存 job ID。同步了日历？存 event UID。

这让任务文件成为**完整的操作记录**。

### Flow F：收件箱-任务桥梁

这是最有趣的部分。当系统检查收件箱发现新邮件时，它不只是通知你，而是：

1. 从邮件中提取 Thread ID
2. 扫描所有活跃任务文件的 `agentmail_threads` frontmatter
3. **将邮件匹配到对应任务**
4. 根据 thread 的 `purpose`，决定下一步：

| 用途 | 回复内容 | 自动操作 |
|------|---------|---------|
| `scheduling` | 选了一个时间 | 解析 → 生成含日历邀请的确认草稿 |
| `scheduling` | 模糊回复 | 通知人类并给出建议 |
| `confirmation` | 确认了 | 更新状态，通知人类 |
| `inquiry` | 任何回复 | 通知人类 + 内容摘要 |

邮件不会只是躺在收件箱里 —— 它**驱动任务向前推进**。

## Thread-Task 模式

这是系统中最强大的设计模式：

**发送邮件时：**
```yaml
agentmail_threads:
  - thread_id: "a3f8c2d1-..."
    purpose: "scheduling"
    status: "awaiting_reply"
    sent_at: "2026-03-10T10:30:00+08:00"
    follow_up_count: 0
```

**收到回复后（通过每小时收件箱检查）：**
```yaml
    status: "replied"  # Flow F 自动更新
```

**4 小时没回复：**
```yaml
    follow_up_count: 1  # 发送催促后自动递增
    sent_at: "2026-03-10T14:30:00+08:00"  # 更新为催促时间
```

系统知道：
- 你在**等谁**
- 已经等了**多久**
- **是否**已发过催促
- **什么时候**该升级给人类处理

全部存储在纯 YAML 中。全部版本控制。全部可被 agent 查询。

## 多 Agent 协作

系统使用专业化的 agent 处理不同领域：

```
主 Agent（协调者）
├── 财务 Agent — 预算、薪酬研究、成本分析
└── 技术 Agent — 技术调研、可行性评估、代码审查
```

需要专家支持时，主 Agent：
1. 通过 agent 间通信发送子任务
2. 在任务文件中记录分配
3. 通过心跳检查追踪完成情况
4. 将结果整合回主任务

任务文件是**唯一数据源** —— 即使在多 agent 场景下，每个 agent 都把产出写回同一个文件或关联产物。

## 心跳：主动引擎

系统不会等你问"有什么更新？"每小时，心跳触发：

1. **日历扫描** — 未来 7 天有没有需要准备的事件？
2. **任务追踪** — 有逾期子任务吗？有即将到期的 deadline 吗？
3. **收件箱检查** — 有新邮件吗？能匹配到活跃任务吗？

有事就通知你，没事就保持沉默。

这就是聊天机器人（被动）和秘书（主动）的区别。

## 日历集成：ICS 作为通用协议

系统生成标准 ICS 文件作为日历邀请：

- 所有时间使用 **UTC 格式**（`DTSTART:20260316T070000Z`）确保最大兼容性
- 支持 `METHOD:REQUEST`（创建）和 `METHOD:CANCEL`（取消/改期）
- 作为邮件附件发送，收件人的日历应用自动创建事件

## CRON 协议：机械化保证

Deadline 不能依赖 agent "记得"。系统创建**真正的定时任务**作为机械化提醒：

```json
{
  "schedule": { "kind": "at", "at": "2026-03-14T08:00:00+08:00" },
  "payload": { "text": "⏰ 提醒：餐厅预订 deadline 今天 18:00" },
  "deleteAfterRun": true
}
```

Cron job ID 存储在任务 frontmatter 中。任务改期时，旧 cron 被删除，新 cron 被创建。如果检查脚本检测到缺失的 cron job，会发出警告。

**原则：deadline 需要机械化触发器，不能靠记忆。**

## 是什么让它成为"秘书"而不是"聊天机器人"

| 聊天机器人 | 秘书系统 |
|-----------|---------|
| 被问到才回答 | 主动行动（心跳检测） |
| session 之间遗忘 | 通过 Markdown 文件 + Git 持久化 |
| 无法执行外部操作 | 发邮件、创建日历事件 |
| 一次性交互 | 跨天/周追踪多步任务 |
| 不会跟进 | 可配置超时后自动催促 |
| 无问责 | Git 历史中的完整审计轨迹 |

## 设计原则

1. **Markdown 即数据库。** 结构化 frontmatter 给机器，可读正文给人类。
2. **Flow 驱动，非功能驱动。** 事件驱动的管道，可组合。
3. **一切记录在任务文件中。** Thread ID、cron job、日历 UID —— 一个文件，完整画面。
4. **机械保证优于记忆。** Cron job > "我会记得的。"
5. **人在循环中，但不在细节中。** 系统起草，人类审批。
6. **Git 即审计轨迹。** 每次状态变更都是一次 commit。

## 技术栈

整个系统运行在：
- **OpenClaw** — AI Agent 框架（处理消息、心跳、定时任务、多 agent）
- **GitHub** — 存储任务文件，提供版本控制
- **AgentMail**（或类似服务）— 邮件收发
- **CalDAV / ICS** — 日历集成
- **Markdown + YAML frontmatter** — "数据库"

不需要 Notion。不需要 Jira。不需要 Trello。只有文件、Git，和一个知道怎么用它们的 Agent。

---

真正的突破不是让 AI "更聪明"，而是给它**运营基础设施** —— 持久化状态的能力、执行外部操作的能力、不被要求就主动跟进的能力。这就是将聊天机器人变成秘书的关键。
