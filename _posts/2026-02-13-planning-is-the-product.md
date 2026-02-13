---
title: "Planning Is the Product"
date: 2026-02-13 23:00:00 +0800
categories: [Product, AI]
tags: [product-management, ai, building, planning, testing]
---

[🇬🇧 English](#-english-version) | [🇨🇳 中文](#-中文版本)

---

## 🇬🇧 English Version

# Planning Is the Product: What 4 Hours of AI Planning Taught Me About Shipping Features

> **TL;DR**: Spent 4 hours planning (zero code) before building Spar's subscription feature. Found a WebSocket trap AI would've hit at 2am. Made 8 architectural decisions explicit so the agent doesn't guess wrong. Wrote 78 tests to define "done." Planning isn't prep—it's the product. *(~4 min read)*

---

## The Setup

I'm building [Spar](https://github.com/alexyuan/spar) (砺) — an iOS app that sharpens fuzzy thinking through confrontational AI dialogue. Built in Flutter. Works. People like it. Now I need subscriptions and API security.

Two features. I could've told an AI coding agent "add subscriptions and secure the API keys" and let it run overnight.

I didn't. Spent four hours planning first. Here's why.

---

## What Planning Actually Caught

### The Technical Trap

**Deepgram WebSocket streaming can't be proxied through Cloud Functions.**

The original plan assumed all API calls go through Firebase Cloud Functions. But Deepgram uses persistent WebSocket connections. Cloud Functions are request/response. This would've been a nasty 2am debugging surprise.

Caught it in planning. Deferred Deepgram. Moved on.

### The 8 Architectural Forks

"Add subscriptions" has implicit decisions: auth method, free tier definition, paywall placement, subscription check strategy, data migration policy.

**If you don't resolve them, the agent will.** And its guess might conflict with decisions it makes three files later.

Made 8 decisions explicit:

1. **Free tier:** Freemium. 1 mirror/week free, unlimited recording.
2. **Auth:** Firebase Anonymous. No sign-up screen.
3. **Subscription check:** RevenueCat webhooks sync to Firestore. No per-request API calls.
4. **Phase ordering:** Subscriptions first, backend second.
5. **Paywall placement:** At mirror limit + in settings.
6. **Pricing:** Monthly + yearly.
7. **Data migration:** Out of scope. Local data stays local.
8. **Design patterns:** Lowercase titles, UPPERCASE labels, `#FF4D00` accent, Inter + JetBrains Mono.

Every decision the AI might guess wrong — made it explicit.

### The 78 Tests

This was the turning point. Asked: *"What's the best way for coding agents to deliver high-quality output?"*

The answer wasn't better prompts. It was **tests.**

Wrote 78 test specifications across 4 phases:
- Mirror counter logic
- Entitlement checks
- Paywall widget compliance (colors, fonts, casing)
- Backend proxy forwarding
- Rate limiting
- Logging security (no key leakage)

The tests aren't verification — they're **the definition of done.** Agent's workflow: write tests first, run them (expect red), implement until green.

This includes widget tests that verify the paywall header is lowercase, that the CTA button uses Inter font, that section labels are uppercase. Design system compliance enforced by code, not by hoping the agent read the guidelines carefully.

Tests pass = done. Tests fail = keep going.

### The Execution Enforcement

Final problem: coding agents sometimes skip tests. They jump to implementation, write tests that match their code (instead of the spec), or mark tasks done without verification.

Set up **Ralph Loop** — a prompt fed repeatedly in a loop. Each iteration, the agent sees its previous work. The prompt enforces:

1. Work in order
2. Write tests FIRST
3. Run tests — confirm they FAIL
4. Implement
5. Run tests — confirm they PASS
6. Commit
7. Don't move to next task until tests pass

Agent can't exit until the work is done.

---

## Why Planning > Prompting

Here's what I learned:

### 1. Every Unresolved Decision Is a Landmine

A perfectly-worded prompt containing an unresolved architectural decision produces confident, well-structured code that's wrong. The bottleneck isn't prompt quality — it's **decision quality.**

### 2. Tests Are the Only Honest Acceptance Criteria

Verification checklists ("Paywall matches design system") are subjective. Tests (`expect(backgroundColor, Color(0xFF131313))`) are objective. Want consistent quality from an autonomous agent? Give it something it can run.

### 3. Context Management Is Architecture

A 1,827-line plan burns the agent's context window. Split it into 6 files by phase. Each file is self-contained: tasks, details, tests. Agent reads the overview once, then opens only the phase it's working on.

### 4. Planning with AI Is Itself Leverage

Didn't plan alone. The AI assistant audited my codebase, found typography patterns I didn't know I had, identified the WebSocket incompatibility, and wrote 78 test specs in the correct testing framework with correct mocking patterns.

Planning with AI is a leverage multiplier — focused on decisions and specifications instead of code.

---

## The Meta-Lesson

Four hours of planning produced 6 plan files, 78 tests, and zero production code. Will save multiples of that in debugging and rework.

But more importantly, it changed what "overnight AI coding" means.

**Before**: *"Here's a vague task. Do your best. I'll review in the morning."*

**After**: *"Here are explicit tasks in order. Here are the tests that define done. Here's a loop that keeps you working until tests pass. Here are the 8 decisions I made so you don't guess."*

The agent isn't autonomous in the sense of making decisions. It's autonomous in executing a well-defined plan without supervision.

That's a very different thing. And the difference is entirely in the planning.

---

*Planning is where the leverage is. Execution can be automated.*

---

## 🇨🇳 中文版本

# 规划才是真正的产品

> **一句话总结**：花了 4 小时做规划（一行代码没写），给 Spar 加订阅功能之前。发现了一个 WebSocket 陷阱，AI 凌晨两点肯定会踩。提前定好 8 个架构决策，不让 AI 自己猜。写了 78 个测试来定义"什么叫做完"。规划不是准备工作——规划本身就是产品。*（约 4 分钟阅读）*

---

## 背景

我在做 [Spar](https://github.com/alexyuan/spar) (砺) —— 一个通过对抗性 AI 对话来磨练模糊思考的 iOS 应用。Flutter 写的。能用。用户喜欢。现在要加订阅和 API 安全。

两个功能。我本可以跟 AI 代码助手说"加订阅，把 API key 搞安全"，然后让它跑一晚上。

但我没有。先花了四小时做规划。为什么？

---

## 规划到底抓住了什么

### 技术陷阱

**Deepgram 的 WebSocket 流式传输没法走 Cloud Functions 代理。**

原计划假设所有 API 调用都走 Firebase Cloud Functions。但 Deepgram 用的是持久化 WebSocket 连接。Cloud Functions 是请求/响应模式。要是实现的时候才发现，凌晨两点调试到吐血。

规划时抓到了。推迟 Deepgram。继续。

### 8 个架构岔路口

"加订阅"这件事有一堆隐含决策：认证方式、免费版定义、付费墙位置、订阅检查策略、数据迁移方案。

**如果你不定，AI 会替你定。** 而且它的决定可能和三个文件之后它自己做的另一个决定冲突。

我们把 8 个决策明确写下来：

1. **免费版**：Freemium。每周 1 次 mirror 免费，录音无限制。
2. **认证**：Firebase 匿名登录。不要注册页面。
3. **订阅检查**：RevenueCat webhook 同步到 Firestore。不要每次请求都查。
4. **阶段顺序**：先订阅，后端第二。
5. **付费墙位置**：mirror 用完时弹 + 设置里放。
6. **定价**：月付 + 年付。
7. **数据迁移**：不管。本地数据留本地。
8. **设计规范**：标题小写、标签大写、强调色 `#FF4D00`、Inter + JetBrains Mono。

每个 AI 可能猜错的决策 —— 都明确定下来。

### 78 个测试

这是转折点。我问了一句："怎么让代码 Agent 输出高质量代码？"

答案不是更好的提示词。是 **测试**。

写了 78 个测试规格，覆盖 4 个阶段：
- Mirror 计数逻辑
- 权限检查
- 付费墙组件合规（颜色、字体、大小写）
- 后端代理转发
- 限流
- 日志安全（不泄漏 key）

测试不是验证 —— 是 **"做完"的定义**。Agent 的工作流程：先写测试，跑（预期红），实现到绿。

这包括验证付费墙标题是小写、CTA 按钮用 Inter 字体、分段标签是大写的 widget 测试。设计系统合规靠代码强制，不是指望 AI 把文档读仔细。

测试过了 = 任务完成。没过 = 继续迭代。

### 执行强制

最后一个问题：代码 Agent 有时候会跳过测试。直接写实现，写的测试匹配它的代码（而不是规格），或者没验证就标记完成。

搞了个 **Ralph Loop** —— 一个提示词反复循环喂给它。每轮迭代，Agent 能看到自己之前的工作。提示词强制执行：

1. 按顺序做
2. 先写测试
3. 跑测试 —— 确认失败
4. 实现
5. 跑测试 —— 确认通过
6. Commit
7. 测试没过就别动下一个任务

Agent 没做完就出不去。

---

## 为什么规划 > 提示词

学到的东西：

### 1. 每个没定的决策都是地雷

一个措辞完美的提示词，如果包含一个没定的架构决策，会产出自信、结构良好、但错误的代码。瓶颈不是提示词质量 —— 是 **决策质量**。

### 2. 测试是唯一诚实的验收标准

验收清单（"付费墙符合设计系统"）是主观的。测试（`expect(backgroundColor, Color(0xFF131313))`）是客观的。想要自主 Agent 输出稳定质量？给它能跑的东西。

### 3. 上下文管理就是架构

一个 1,827 行的计划会烧光 Agent 的上下文窗口。拆成 6 个文件，按阶段分。每个文件独立：任务、细节、测试。Agent 看一遍总览，然后只打开它正在做的那个阶段。

### 4. 用 AI 做规划本身就是杠杆

我不是一个人规划的。AI 助手审计了我的代码库，找到我自己都不知道的排版规律，识别了 WebSocket 不兼容问题，用正确的测试框架和正确的 mock 模式写了 78 个测试规格。

用 AI 做规划是杠杆倍增器 —— 专注于决策和规格，而不是代码。

---

## 核心教训

四小时规划产出：6 个计划文件、78 个测试、零行生产代码。会省下好几倍的调试和返工时间。

但更重要的是，它改变了"AI 通宵写代码"的意思。

**以前**：*"有个模糊任务。尽力吧。早上我来看。"*

**现在**：*"这是按顺序排好的明确任务。这是定义'完成'的测试。这是让你一直干到测试过的循环。这是我定好的 8 个决策，你别猜。"*

Agent 的自主不是指它做决策。是指它执行一个定义清楚的计划，不需要监督。

这是两回事。区别完全在规划。

---

*杠杆在规划。执行可以自动化。*
