---
title: "Multi-Agent Cron Job Delivery: How to Route Messages to the Right Bot"
date: 2026-02-24
tags: [openclaw, multi-agent, cron, automation]
---

[🇬🇧 English](#-english-version) | [🇨🇳 中文](#-中文版本)

---

## 🇬🇧 English Version

# Multi-Agent Cron Job Delivery: How to Route Messages to the Right Bot

**TL;DR:** In a multi-agent OpenClaw setup, cron job delivery has a subtle gotcha: the `announce` flow always sends from the default bot, regardless of which agent ran the job. For non-default agents, you need `delivery.mode: "none"` + explicit `message` tool calls with `accountId`. This post documents the problem, the fix, and the patterns we landed on after 21 cron jobs and 3 painful restarts.

---

## The Setup

I run a multi-agent OpenClaw system with three agents:

- **Agent Alpha** (default) — daily operations, heartbeat checks
- **Agent Beta** — scheduled reports, alerts
- **Agent Gamma** — monitoring, scans

Each agent has its own Telegram bot. When Agent Beta sends a report, it should come from the Bot B. When Agent Alpha sends a notification, it comes from the Bot A. Simple enough, right?

## The Problem

Cron jobs in OpenClaw support an `agentId` field. You'd expect that setting `agentId: "beta"` would make the message come from the Bot B. It doesn't — at least not through the default delivery path.

Here's why:

OpenClaw has two delivery paths for isolated cron job responses:

1. **Direct outbound** — triggered when the response contains structured content (media, channel data) or targets a forum thread
2. **Announce flow** — used for all text-only responses

The announce flow **does not respect per-agent channel-account bindings**. It always delivers using the default agent's bot token. So if your secondary agent produces a beautiful report, it arrives in Telegram from the Bot A. Your users see the wrong identity. Confusion ensues.

## The Fix

The solution is straightforward once you know the constraint:

- **Default agent (Alpha):** Use `delivery.mode: "announce"` — it works correctly because the default bot IS the main agent's bot
- **Any other agent:** Use `delivery.mode: "none"` and have the agent send via the `message` tool with explicit `accountId`

### Pattern A: Main Agent (Simple)

```json
{
  "agentId": "main",
  "sessionTarget": "isolated",
  "wakeMode": "now",
  "payload": {
    "kind": "agentTurn",
    "message": "Generate a weather report for Suzhou."
  },
  "delivery": {
    "mode": "announce",
    "to": "123456789"
  }
}
```

The agent produces a response. The system delivers it. Done.

### Pattern B: Non-Default Agent (Message Tool Required)

```json
{
  "agentId": "beta",
  "sessionTarget": "isolated",
  "wakeMode": "now",
  "payload": {
    "kind": "agentTurn",
    "message": "## Delivery Instructions\nUse message tool:\n- action: send\n- channel: telegram\n- accountId: beta\n- target: 123456789\n\nAfter sending, reply NO_REPLY.\n\n## Task\nGenerate today's scheduled report."
  },
  "delivery": {
    "mode": "none"
  }
}
```

Key details:

- **`delivery.mode: "none"`** — disables the announce flow, keeps the message tool available
- **Delivery instructions go at the TOP of the prompt** — not the bottom. Agents are more likely to follow instructions they see first
- **Hardcode the target** — don't let the agent guess who to send to
- **`accountId: "beta"`** — this is what controls which bot sends the message
- **End with `NO_REPLY`** — prevents the system from posting a duplicate summary

## The Gotchas

### 1. `accountId` is NOT a valid delivery field

This seems intuitive but doesn't work:

```json
{
  "delivery": {
    "mode": "announce",
    "accountId": "beta",
    "to": "123456789"
  }
}
```

The delivery schema has `additionalProperties: false`. Unknown fields are silently rejected. Your job runs, your message sends — from the wrong bot. No error, no warning.

### 2. `wakeMode: "next-heartbeat"` causes delays

With a 1-hour heartbeat interval, your 7:00 AM weather report might arrive at 7:45 AM. Always use `"now"` for isolated jobs.

### 3. Prompt placement matters

Put delivery instructions at the top:

```
## Delivery Instructions (MUST follow)
Use message tool with accountId: "beta", target: "123456789"
Reply NO_REPLY after sending.

---

## Your Actual Task
Generate the scheduled report...
```

When instructions are at the bottom, agents occasionally "forget" them after generating a long response.

### 4. `sessionTarget` and `payload.kind` are strictly paired

| sessionTarget | payload.kind | Use case |
|---|---|---|
| `"isolated"` | `"agentTurn"` | Standalone task, delivered via announce or message tool |
| `"main"` | `"systemEvent"` | Inject reminder into the agent's active session |

Mix them and validation fails. No exceptions.

## Testing Protocol

Never deploy a recurring cron job without testing first:

```json
{
  "name": "Test Beta delivery",
  "deleteAfterRun": true,
  "schedule": {
    "kind": "at",
    "at": "2026-02-23T15:00:00.000Z"
  },
  "agentId": "beta",
  "sessionTarget": "isolated",
  "wakeMode": "now",
  "payload": {
    "kind": "agentTurn",
    "message": "Send '✅ Test successful' via message tool with accountId: beta, target: 123456789. Reply NO_REPLY."
  },
  "delivery": { "mode": "none" }
}
```

Check three things:
1. ✅ Message arrived on Telegram
2. ✅ Sent from the correct bot (Bot B, not Bot A)
3. ✅ Delivered to the correct chat

Only then convert to a recurring schedule.

## Rules Summary

| Agent | Delivery Mode | How Message Is Sent |
|---|---|---|
| Main/default | `announce` | System handles it automatically |
| Any other agent | `none` | Agent uses `message` tool with `accountId` |

Five rules to remember:

1. Main agent → `announce`. Non-default agents → `none` + message tool
2. Never put `accountId` in the delivery object
3. Hardcode `target` in the prompt
4. Always use `wakeMode: "now"` for isolated jobs
5. Test with `deleteAfterRun: true` before enabling recurring

## The Cost of Getting This Wrong

We configured 21 cron jobs across 3 agents. The first attempt had Beta jobs using `announce` mode — every scheduled report arrived from the Bot A. Users couldn't tell which agent was talking. We also tried putting `accountId` in the delivery object, which was silently ignored. Three restarts and a lot of debugging later, we landed on the patterns above.

The underlying issue is a reasonable architectural choice: the announce flow is a convenience feature that routes through the default bot. It's not broken — it's just not designed for multi-agent identity. Once you know that, the workaround is clean and reliable.

Document your patterns. Test before deploying. And never trust a field name that looks like it should work without checking the schema first.

---

## 🇨🇳 中文版本

# Multi-Agent Cron Job 消息路由：如何确保消息从正确的 Bot 发送

**一句话总结：** 在多 Agent 的 OpenClaw 系统中，cron job 的 `announce` 模式总是通过默认 bot 发送，不管实际执行的是哪个 agent。非默认 agent必须用 `delivery.mode: "none"` + `message` tool 手动发送，并指定 `accountId`。本文记录了踩坑过程和最终方案。

---

## 背景

我跑了一个三 agent 的 OpenClaw 系统：

- **Agent Alpha**（默认 agent）— 日常运营、心跳检查
- **Agent Beta** — 定期报告、提醒
- **Agent Gamma** — 监控、扫描

每个 agent 对应一个独立的 Telegram bot。Agent Beta 发报告，应该从 Bot B 出来。Agent Alpha 发通知，从 Bot A 出来。逻辑很清楚。

## 问题

Cron job 支持 `agentId` 字段。直觉上，设了 `agentId: "beta"` 消息就该从 Bot B 发出。但实际不是。

OpenClaw 的 isolated cron job 有两条投递路径：

1. **Direct outbound** — 响应包含结构化内容（媒体、频道数据）或目标是论坛帖子时触发
2. **Announce flow** — 所有纯文本响应走这条路

Announce flow **不区分 agent 的 channel-account 绑定**，始终用默认 agent 的 bot token 发送。结果就是：Agent Beta 精心生成的报告，从 Bot A 发出来了。

## 解决方案

知道了原因，方案很直接：

- **默认 agent（Alpha）：** 用 `delivery.mode: "announce"` — 默认 bot 就是主 agent 的 bot，没问题
- **其他 agent：** 用 `delivery.mode: "none"`，让 agent 通过 `message` tool 发送，指定 `accountId`

### 模式 A：主 Agent（简单）

```json
{
  "agentId": "main",
  "sessionTarget": "isolated",
  "wakeMode": "now",
  "payload": {
    "kind": "agentTurn",
    "message": "生成苏州天气预报。"
  },
  "delivery": {
    "mode": "announce",
    "to": "123456789"
  }
}
```

Agent 生成响应，系统自动投递。完事。

### 模式 B：非默认 Agent（需要 message tool）

```json
{
  "agentId": "beta",
  "sessionTarget": "isolated",
  "wakeMode": "now",
  "payload": {
    "kind": "agentTurn",
    "message": "## 发送方式\n用 message tool 发送：\n- action: send\n- channel: telegram\n- accountId: beta\n- target: 123456789\n\n发送完回复 NO_REPLY。\n\n## 任务\n生成今日 每日播报。"
  },
  "delivery": { "mode": "none" }
}
```

关键点：

- **`delivery.mode: "none"`** — 关闭 announce，保留 message tool
- **发送指令放在 prompt 最前面** — agent 更容易遵守先看到的指令
- **硬编码 target** — 不要让 agent 自己推断收件人
- **`accountId: "beta"`** — 这才是控制哪个 bot 发送的关键
- **最后回复 `NO_REPLY`** — 防止系统再发一条重复摘要

## 踩坑记录

### 1. `accountId` 不是 delivery 的合法字段

这看起来很合理，但不生效：

```json
{
  "delivery": {
    "mode": "announce",
    "accountId": "beta",
    "to": "123456789"
  }
}
```

Delivery schema 设了 `additionalProperties: false`，未知字段被静默丢弃。Job 正常运行，消息正常发送 — 就是从错误的 bot 发的。没有报错，没有警告。

### 2. `wakeMode: "next-heartbeat"` 导致延迟

心跳间隔 1 小时的话，7:00 的天气预报可能 7:45 才到。Isolated job 一律用 `"now"`。

### 3. Prompt 中指令位置很重要

发送指令放顶部：

```
## 发送方式（必须遵守）
用 message tool，accountId: "beta"，target: "123456789"
发送完回复 NO_REPLY。

---

## 实际任务
生成播报...
```

放底部的话，agent 生成长内容后偶尔会"忘掉"发送指令。

### 4. `sessionTarget` 和 `payload.kind` 严格配对

| sessionTarget | payload.kind | 用途 |
|---|---|---|
| `"isolated"` | `"agentTurn"` | 独立任务，通过 announce 或 message tool 投递 |
| `"main"` | `"systemEvent"` | 向 agent 主 session 注入提醒 |

搞混了验证直接报错。

## 测试流程

永远先测试，再上线：

```json
{
  "name": "测试 Beta 投递",
  "deleteAfterRun": true,
  "schedule": {
    "kind": "at",
    "at": "2026-02-23T15:00:00.000Z"
  },
  "agentId": "beta",
  "sessionTarget": "isolated",
  "wakeMode": "now",
  "payload": {
    "kind": "agentTurn",
    "message": "用 message tool 发送 '✅ 测试成功'，accountId: beta，target: 123456789。发完回复 NO_REPLY。"
  },
  "delivery": { "mode": "none" }
}
```

验证三件事：
1. ✅ Telegram 收到消息
2. ✅ 从正确的 bot 发送（Bot B，不是 Bot A）
3. ✅ 发到正确的聊天

通过后再改成定期任务。

## 规则总结

| Agent | Delivery Mode | 发送方式 |
|---|---|---|
| 主/默认 agent | `announce` | 系统自动处理 |
| 其他 agent | `none` | Agent 用 `message` tool + `accountId` |

五条铁律：

1. 主 agent → `announce`；非默认 agent → `none` + message tool
2. 永远不要在 delivery 对象里放 `accountId`
3. 在 prompt 里硬编码 `target`
4. Isolated job 一律用 `wakeMode: "now"`
5. 用 `deleteAfterRun: true` 测试通过后再改定期

## 代价

我们配了 21 个 cron job，横跨 3 个 agent。第一次尝试 Beta 的 job 全用了 `announce` — 所有定期报告都从 Bot A 发出来。还试过在 delivery 里加 `accountId`，被静默忽略。三次重启、大量调试之后，才定下了上面的方案。

根本原因其实是一个合理的架构选择：announce flow 是个便利功能，走默认 bot 通道。它没坏 — 只是不是为多 agent 身份设计的。知道这一点后，workaround 干净可靠。

记录你的 pattern。部署前先测试。永远不要相信一个"看起来应该生效"的字段名 — 先查 schema。
