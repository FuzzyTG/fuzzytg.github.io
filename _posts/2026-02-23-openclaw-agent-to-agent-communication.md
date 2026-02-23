---
title: "How to Set Up Agent-to-Agent Communication in OpenClaw"
date: 2026-02-23
tags: [openclaw, multi-agent, automation]
---


**TL;DR:** Telegram bots can't see each other's messages in groups (API limitation). Use OpenClaw's built-in `sessions_send` tool instead. Enable two settings: `agentToAgent.enabled: true` and `sessions.visibility: "all"`.

---

I run multiple agents on a single OpenClaw gateway. Each handles a different domain of my work. Recently I wanted them to talk to each other: I tell Agent A something, and it delegates a question to Agent B, who then responds directly in my private chat.

Getting this to work wasn't obvious. Here's what I learned.

## The Goal

- Tell Agent A to send a message to Agent B
- Agent B processes the message and responds in my existing conversation with Agent B
- All agents share context when needed

## What Didn't Work: Telegram Group Chat

My first instinct was to put all the bots in a Telegram group chat. The idea was simple — they'd all see each other's messages and respond naturally.

It doesn't work. **Telegram's Bot API does not deliver bot messages to other bots in groups.** When Bot A sends a message in a group, Telegram only notifies Bot A itself. Bot B never receives the update. This is a platform-level restriction to prevent bot spam loops — no amount of configuration can change it.

This is worth emphasizing: **this is not an OpenClaw limitation.** It's how Telegram's Bot API is designed. Any framework or tool building on Telegram bots faces the same constraint. A bot in a group can see messages from humans, but it is blind to messages from other bots.

## What About Telegram Channels?

Telegram has a workaround at the API level. Unlike groups, **Channels** do deliver bot messages to other bots via `channel_post` updates. OpenClaw's codebase is aware of this and includes a `channel_post` handler that normalizes channel posts into the standard message pipeline.

However, Channels are fundamentally **one-way broadcast feeds**, not conversation spaces. There's no threading, no reply context, no back-and-forth group chat UX. Messages from your bots would appear as a flat stream of announcements. It works technically — Bot A posts, Bot B sees it and posts a response — but the experience is nothing like a group conversation.

I explored this path and decided against it. If you need a true multi-agent group chat, Telegram simply isn't the right surface for it. The platform was designed with a clear boundary: bots don't talk to bots in groups, and channels aren't conversations.

## What Works: Built-In Agent-to-Agent Messaging

OpenClaw has a dedicated agent-to-agent (a2a) system using the `sessions_send` tool. When Agent A calls this tool targeting Agent B:

1. Agent A sends a message to Agent B's session
2. The two agents can negotiate internally (up to 5 turns, invisible to you)
3. Agent B announces its final response in its own channel — your private chat with Agent B

No Telegram group needed. The communication happens inside the gateway.

## The Configuration

This is the part that tripped me up. There are **two separate gates**, and both must be open.

### Gate 1: Agent-to-Agent Policy

```json
{
  "tools": {
    "agentToAgent": {
      "enabled": true
    }
  }
}
```

This allows agents to send messages to each other. Omitting the `allow` list means all agents on the gateway can communicate — which is fine for a single-user setup.

If you want to restrict which agents can talk, add an explicit allow list:

```json
{
  "tools": {
    "agentToAgent": {
      "enabled": true,
      "allow": ["agent-01", "agent-02", "agent-03"]
    }
  }
}
```

### Gate 2: Session Visibility

```json
{
  "tools": {
    "sessions": {
      "visibility": "all"
    }
  }
}
```

This is the one I missed initially. The default visibility is `"tree"`, which restricts agents to only seeing their own session and any sub-sessions they spawned. Cross-agent communication requires `"all"`.

**This is a global setting** — it applies to every agent on the gateway. There's no per-agent override for session visibility.

### Full Minimal Config

```json
{
  "tools": {
    "sessions": {
      "visibility": "all"
    },
    "agentToAgent": {
      "enabled": true
    }
  }
}
```

## Is `visibility: "all"` a Security Risk?

This was my first concern. With `"all"`, any agent can:

- List sessions belonging to other agents
- Read conversation history from other agents
- Send messages to other agents

For a multi-user gateway or agents with different trust levels, this matters. But for a single-user setup where all agents work for you, it's a feature. Shared context between your own agents is the whole point.

Importantly, `visibility: "all"` does **not** eagerly load all session data into every agent's context. It's purely an access control flag. Session data is only fetched when an agent explicitly calls `sessions_list` or `sessions_history`. There's no token cost until an agent actually uses it, and individual history fetches are capped at 80KB.

## Gotchas

### Announce Delivery Noise

When Agent B processes an A2A message, its response goes through the announce delivery flow. This means the reply gets pushed to your private chat with Agent B — which is the intended behavior. But **internal chatter also leaks through.** If Agent B replies `NO_REPLY` or sends a status update back to Agent A, announce still pushes it to you.

In practice, a short A2A exchange between two agents can produce 3-4 unwanted messages in your Telegram. The agents aren't doing anything wrong — the delivery system doesn't distinguish between "response meant for the user" and "internal acknowledgment."

There's no clean fix yet. You can mitigate it by:
- Keeping A2A instructions terse (reduce back-and-forth)
- Using `timeoutSeconds` on `sessions_send` to limit negotiation time
- Telling agents explicitly not to send acknowledgments back

### Prompt Compliance

A2A messages are processed like any other user message in the target agent's session. The receiving agent follows its own system prompt, personality, and judgment. If you tell Agent A to instruct Agent B to "send a message to the user using the message tool," Agent B might decide that's unnecessary, or interpret the instruction differently.

This isn't a bug — it's how LLMs work. Treat A2A instructions like you'd treat delegation to a colleague: be specific, put the important part first, and expect some interpretation.

## Summary

| What | Setting |
|---|---|
| Let agents message each other | `tools.agentToAgent.enabled: true` |
| Let agents see each other's sessions | `tools.sessions.visibility: "all"` |
| Restrict which agents can communicate | `tools.agentToAgent.allow: [...]` (optional) |
| Per-agent visibility override | Not supported (global only) |
| Telegram group chat with multiple bots | Not possible (Bot API limitation) |
| Telegram channel as workaround | Technically works, poor UX (broadcast-only, no threading) |

---

*Tested on OpenClaw v2026.2.x. This behavior may change in future versions.*

---

# 如何在 OpenClaw 中配置 Agent 间通信

**一句话总结：** Telegram Bot API 不允许 bot 在群里看到其他 bot 的消息。用 OpenClaw 内置的 `sessions_send` 工具代替。需要两个配置：`agentToAgent.enabled: true` 和 `sessions.visibility: "all"`。

---

我在一个 OpenClaw gateway 上跑多个 agent，每个负责不同领域。最近我想让他们互相通信：我告诉 Agent A 一件事，A 委托给 Agent B，B 直接在我和他的私聊里回复。

搞定这个花了点功夫。以下是我学到的。

## 目标

- 让 Agent A 给 Agent B 发消息
- Agent B 处理后在我和 B 的私聊中回复
- 需要时 agent 之间共享上下文

## 不行的方案：Telegram 群聊

第一反应是把所有 bot 拉进一个 Telegram 群。想法很简单——他们都能看到彼此的消息，自然地回复。

不行。**Telegram Bot API 不会把 bot 的消息推送给群里的其他 bot。** Bot A 在群里发消息，只有 Bot A 自己收到通知，Bot B 永远看不到。这是平台层面的限制，防止 bot 垃圾消息循环——再怎么配置也改不了。

这点很重要：**这不是 OpenClaw 的限制。** 这是 Telegram Bot API 的设计。任何基于 Telegram bot 的框架都面临同样的约束。

## Telegram Channel 呢？

Channel 确实能把 bot 消息推送给其他 bot（通过 `channel_post`）。OpenClaw 也有对应的处理逻辑。

但 Channel 本质是**单向广播**，不是对话空间。没有线程、没有回复上下文、没有来回对话的体验。我试了这条路，放弃了。

## 真正有效的方案：内置 A2A 通信

OpenClaw 有专门的 agent-to-agent 系统，使用 `sessions_send` 工具：

1. Agent A 发消息到 Agent B 的 session
2. 两个 agent 可以内部协商（最多 5 轮，对你不可见）
3. Agent B 的最终回复通过 announce 推送到你和 B 的私聊

不需要 Telegram 群。通信在 gateway 内部完成。

## 配置

关键点：有**两道门**，都得打开。

### 第一道门：Agent-to-Agent 策略

```json
{
  "tools": {
    "agentToAgent": {
      "enabled": true
    }
  }
}
```

不设 `allow` 列表 = 所有 agent 都能互相通信。单用户场景下没问题。

### 第二道门：Session 可见性

```json
{
  "tools": {
    "sessions": {
      "visibility": "all"
    }
  }
}
```

默认是 `"tree"`（只能看自己的 session）。跨 agent 通信需要 `"all"`。**这是全局设置**，没有 per-agent 覆盖。

### 完整最小配置

```json
{
  "tools": {
    "sessions": {
      "visibility": "all"
    },
    "agentToAgent": {
      "enabled": true
    }
  }
}
```

## `visibility: "all"` 安全吗？

设成 `"all"` 后，任何 agent 都能列出其他 agent 的 session、读历史、发消息。

多用户场景或信任级别不同的 agent 需要注意。但单用户场景下，agent 之间共享上下文本来就是目的。

重要：`visibility: "all"` **不会**主动把所有 session 数据加载到每个 agent 的上下文里。它只是访问控制标志。只有 agent 显式调用 `sessions_list` 或 `sessions_history` 时才会拉取数据，单次上限 80KB。

## 踩坑记录

### Announce 刷屏

Agent B 处理 A2A 消息后，回复走 announce delivery 推送到你的私聊。但**内部通信也会泄漏**——Agent B 回 `NO_REPLY` 或发状态更新给 Agent A 时，announce 照样推给你。

一次简短的 A2A 交互可能产生 3-4 条不想要的消息。缓解方法：
- A2A 指令写简洁，减少来回
- 用 `timeoutSeconds` 限制协商时间
- 明确告诉 agent 不要回发确认消息

### Prompt 服从度

A2A 消息在目标 agent 的 session 里按普通用户消息处理。接收方有自己的 system prompt 和判断力。你让 Agent A 指示 Agent B "用 message tool 给用户发消息"，Agent B 可能觉得没必要，或者理解不同。

这不是 bug——LLM 就是这样。把 A2A 指令当作委托给同事：要具体，重要的放前面，允许一定的自主判断。

## 总结

| 配置项 | 设置 |
|---|---|
| 允许 agent 互相发消息 | `tools.agentToAgent.enabled: true` |
| 允许 agent 看到其他 session | `tools.sessions.visibility: "all"` |
| 限制哪些 agent 可以通信 | `tools.agentToAgent.allow: [...]`（可选） |
| Per-agent 可见性覆盖 | 不支持（仅全局） |
| Telegram 群聊多 bot | 不可行（Bot API 限制） |
| Telegram Channel 作为替代 | 技术上可行，体验差（单向广播，无线程） |

---

*测试环境：OpenClaw v2026.2.x。此行为可能在未来版本中修复。*
