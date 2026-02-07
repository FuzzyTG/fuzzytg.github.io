---
title: "The Logic Matters More Than the Code"
date: 2026-02-07 16:00:00 +0800
categories: [Product, AI]
tags: [product-management, ai, building, claude-code]
---

[🇬🇧 English](#english) | [🇨🇳 中文](#chinese)

---

<div id="english"></div>
## 🇬🇧 English Version

# The Logic Matters More Than the Code: A PM's Journey Into Building With AI

For years, I was a product manager. I knew *what* to build and *why* — I just couldn't build it myself. I wrote user stories, prioritized roadmaps, and handed PRDs to engineers. My superpower was seeing the future product. My limitation was needing someone else to make it real.

Then AI coding agents arrived and I could suddenly ship myself. No sprint planning. No ticket grooming. Just me, Claude Code, and a production app.

Then I spent **2 hours debugging three lines of code** and learned something important: **Writing code was never the hard part. Thinking through what happens when the code runs — that's where the real work is.**

---

## The Wake-Up Call: When Metrics Lie

I'm building **Spar** (砺) — an iOS app that sharpens fuzzy thinking through confrontational AI dialogue. The product philosophy is deliberately aggressive: you ramble, the AI challenges you, and you either reach clarity or admit you're not ready yet.

This week users reported two bugs:
1. Threads wouldn't close even after clear decisions
2. AI asked unlimited follow-ups with no escape valve

I wrote a fix plan and sat down with Claude Code. The agent shipped the changes in 20 minutes. Tests passed.

Then I asked: **"What happens to the Clarity Score when the system pauses?"**

---

## Bug #1: Teaching Metrics to Lie

Spar tracks a **Clarity Score** showing how often your thinking is clear enough that the AI doesn't need to intervene.

The agent had just implemented a pause mechanism: after 8 consecutive interventions, the system would suggest taking a break. The pause response returned the same status code as genuine clarity — counting it as a successful turn.

**Your score would jump from 0% to 12.5%.**

The agent's reasoning: *"The system wisely paused. That's smart thread management."*

I saw it differently. The Clarity Score measures **the user's thinking quality**, not **the system's decision quality**. A forced timeout isn't the user winning — it's the system giving up.

**An engineer sees a calculation that works.** The math is right. The code executes. The tests pass.

**A PM sees a metric that lies to users about their progress.**

This is when I realized: **the ability to write code is no longer the competitive advantage. The ability to think through what the code means to users — that's the game now.**

---

## Bug #2: What Happens at Turn 9?

I asked: *"Walk me through turn 9. Turn 10. Turn 11."*

The agent realized: the user would get the **same pause suggestion every single turn.** The condition checked for "8 or more," not "exactly 8."

The fix was changing one character. But the bug was invisible in code review because **you had to simulate the user's journey** to catch it.

**This is classic PM thinking.** Engineers think in code paths. PMs think in **user journeys.**

---

## Bug #3: When the Product Soul Is at Stake

The agent proposed a "cooling phase" where the AI would gradually de-escalate before the hard pause.

I killed it instantly.

The app is called **Spar** — a whetstone. The entire product identity is that this AI **does not give you easy outs.**

A "cooling phase" where the AI pretends your vague answer was fine? That **contradicts everything the product stands for.**

The agent suggested it because it's a reasonable UX pattern. It was technically sound. **But it was wrong for this product.** Because of **product soul.**

This is where being a PM matters most. You need to know the soul of what you're building — and reject technically correct solutions that kill it.

---

## Bug #4: Contradicting Myself

My system prompt told the LLM to "proactively detect clarity and close threads" but also "ONLY close if the user explicitly says stop."

These instructions **contradicted each other.** The agent implemented exactly what I asked. I had to spot that **I'd asked for the impossible.**

PMs spend careers negotiating contradictory stakeholder demands. Now we have to spot the contradictions in our own specs.

---

## You're an Architect, Not a Contractor

When I started working with AI coding agents, I thought my job was describing **what** to build.

I was wrong.

The agent handles the *what*. My job is the **why**, the **what if**, and **what does this mean to users in production.**

**Working with an AI agent is being the architect on a construction site.**

The agent is the contractor: fast, accurate, tireless. It builds exactly what you spec.

But **you're the architect.** You need to know:
- **How the building will be used** (user journeys, not code paths)
- **What the building stands for** (product philosophy, not design patterns)  
- **Where your blueprints contradict themselves**
- **What happens when it rains** (edge cases across time)

The contractor tells you if the wall will stand. You decide if the wall should exist at all.

---

## 2 Hours to Ship 3 Lines

Three lines of logic changes. **Twenty minutes of coding. Two hours of thinking.**

- 45 minutes tracing user scenarios
- 30 minutes debating what the Clarity Score actually measures  
- 30 minutes reconciling contradictory instructions
- 15 minutes defending product philosophy

That ratio — **20 minutes of implementation, 2 hours of judgment** — is the new reality.

Those 2 hours **couldn't be automated.** Claude Code can't answer "should system pauses inflate the Clarity Score?" That requires understanding what the number means to users, why they trust it, how it shapes their perception of progress.

That's not a coding question. It's a product question. And product questions are finally the bottleneck.

---

## What This Means for PMs

If you're a PM who's been held back by not knowing how to code: **that barrier just disappeared.**

**What matters now:**

- Trace scenarios, not features
- Protect metric integrity  
- Catch your own contradictions

---

## PMs Might Actually Be Better at This

After ten years of feeling like "the person who can't build," I'm realizing **PMs might have an advantage in the AI era.**

We're better at **the questions that matter when code is free:**

- "What does this number mean to users?"
- "What happens across time?"  
- "Should this feature exist at all?"

Engineers answer "can we build this?" 

PMs answer "should we?" and "what happens after?"

When implementation is instant, the second set of questions becomes everything.

---

**Postscript:** It took 2 hours to ship 3 lines of code.

I'd spend those 2 hours again in a heartbeat.

Because the logic matters more than the code. And getting the logic right is **finally** the thing we can't hand off to anyone else — not to engineers, not to agents, not to anyone.

It's the PM's job now. And it turns out, it always was.

---

<div id="chinese"></div>
## 🇨🇳 中文版本

# 逻辑比代码重要：一个产品经理开始自己写代码的故事

我做了很多年产品经理。我知道该做什么，知道为什么要做——就是没法自己动手。写用户故事，排优先级，然后把 PRD 丢给工程师。我擅长的是看到产品未来的样子，但做不出来，得靠别人。

后来 AI 编程工具出来了，我突然能自己发布东西了。不用开什么冲刺会议，不用整理需求单，就我和 Claude Code，直接上线。

然后我花了**整整 2 小时调 3 行代码**，明白了一件事：**写代码根本不难，难的是想清楚代码跑起来会发生什么。**

---

## 醒悟：指标会骗人

我在做 **Spar（砺）**——一个 iOS 应用，通过 AI 对话来逼你把模糊的想法想清楚。产品理念就是要硬核：你说得含糊，AI 就挑战你，你要么说清楚，要么承认自己没想明白。

这周用户报了两个 bug：
1. 对话明明该结束了，但就是关不掉
2. AI 一直追着问，完全停不下来

我写了修复方案，跟 Claude Code 一起搞。20 分钟改完，测试通过。

然后我随口问了句：**"系统暂停的时候，清晰度评分怎么算？"**

---

## Bug #1：让指标撒谎

Spar 会追踪一个**清晰度评分**，显示你说得够不够清楚，清楚到 AI 不用插话的程度。

刚实现的暂停机制是这样的：AI 连续干预 8 次后，系统建议你歇会儿。但暂停时返回的状态码，跟真正想清楚了是一样的——也算一次成功对话。

**你的评分会从 0% 直接跳到 12.5%。**

AI 的逻辑是：*"系统很聪明，知道该暂停了，这是好的线程管理。"*

我觉得不对。清晰度评分衡量的是**你的思维质量**，不是**系统的决策质量**。强制暂停不是你赢了——是系统放弃了。

**工程师看到的是计算没问题。**数学对，代码跑得通，测试过了。

**产品经理看到的是这个评分在骗用户。**

就在那一刻我明白了：**会写代码已经不是优势了，能想清楚代码对用户意味着什么，这才是关键。**

---

## Bug #2：第 9 轮会怎样？

我问：*"带我过一遍第 9 轮会发生什么，第 10 轮呢，第 11 轮呢？"*

AI 意识到：用户会**每一轮都收到同样的暂停提示。**判断条件是"8 次或更多"，不是"刚好 8 次"。

改起来很简单，改一个字符就行。但这个 bug 你看代码是看不出来的，**你得真的模拟用户的使用过程**才能发现。

**这就是 PM 的思维方式。**工程师想的是代码逻辑，产品经理想的是**用户用起来的感觉。**

---

## Bug #3：产品灵魂不能丢

AI 建议加一个"缓冲阶段"，让 AI 在彻底暂停前慢慢放松要求。

我马上否了。

这应用叫 **Spar（砺）**——磨刀石。整个产品的核心就是这个 AI **不会给你轻松过关的机会。**

搞个"缓冲阶段"，让 AI 假装你说得还行？这**跟产品的理念完全相反。**

AI 建议这个是因为从 UX 角度看挺合理的，技术上也没问题。**但对这个产品来说就是错的。**因为违背了**产品的灵魂。**

这是做 PM 最关键的时候。你得知道自己做的东西是什么，然后拒绝那些技术上没问题、但会毁掉产品特色的方案。

---

## Bug #4：自己打自己脸

我的系统提示词里让 LLM"主动判断用户是否想清楚了，然后结束对话"，又说"只能在用户明确说停的时候才能结束"。

这俩指令**根本矛盾。**AI 完全按我说的做了，我得自己发现**我要求的东西本身就不可能。**

产品经理天天要协调各种矛盾的需求。现在得学会发现自己写的需求里自相矛盾的地方。

---

## 你是建筑师，不是包工头

我刚开始用 AI 编程工具的时候，以为我的工作是描述**要做什么**。

我想错了。

AI 负责*要做什么*。我的工作是**为什么**、**如果……会怎样**，还有**这对真实用户意味着什么。**

**跟 AI 一起工作，就像在工地上当建筑师。**

AI 是包工头：快、准、不累。你说啥它做啥。

但**你是建筑师。**你得知道：
- **这房子是怎么用的**（用户怎么用，不是代码怎么跑）
- **这房子代表什么**（产品理念，不是设计模式）
- **你的图纸哪里自相矛盾**
- **下雨了怎么办**（各种边界情况）

包工头告诉你墙会不会倒。你决定这堵墙该不该存在。

---

## 3 行代码，2 小时

改了三行逻辑。**编码 20 分钟，思考 2 小时。**

- 45 分钟推演用户场景
- 30 分钟讨论清晰度评分到底衡量什么
- 30 分钟理清矛盾的指令
- 15 分钟坚持产品理念

这个比例——**实现 20 分钟，判断 2 小时**——就是现在的常态。

这 2 小时**没法自动化。**Claude Code 回答不了"系统暂停该不该算进清晰度评分？"这得理解这个数字对用户意味着什么，他们为什么相信它，它怎么影响他们对自己进步的感知。

这不是编程问题，是产品问题。产品问题终于成了瓶颈。

---

## 对产品经理来说意味着什么

如果你是个因为不会编程而受限的产品经理：**这个限制刚消失了。**

**现在重要的是：**

- 推演场景，不是罗列功能
- 保护指标的可信度
- 发现自己的矛盾

---

## 产品经理可能反而更适合

做了十年"那个不会写代码的人"，我现在觉得**产品经理在 AI 时代可能反而有优势。**

我们更擅长**当代码不再是门槛时更重要的问题：**

- "这个数字对用户来说意味着什么？"
- "时间长了会怎样？"
- "这个功能到底该不该做？"

工程师回答"能不能做"。

产品经理回答"该不该做"和"做完了会怎样"。

当实现变得轻而易举，后面这些问题就成了一切。

---

**后记：**3 行代码花了 2 小时。

我一点都不后悔。

因为逻辑比代码重要。把逻辑想对是**终于**成了我们没法推给别人的事——不能推给工程师，不能推给 AI，推给谁都不行。

这是产品经理的活儿。其实一直都是。