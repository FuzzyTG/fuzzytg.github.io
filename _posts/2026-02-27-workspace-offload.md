---
title: "AI Agent Memory Architecture: When to Force-Load vs On-Demand Read"
date: 2026-02-27 23:50:00 +0800
categories: [AI, Architecture]
tags: [ai-agents, memory-management, system-design]
---

[🇬🇧 English](#-english-version) | [🇨🇳 中文](#-中文版本)

---

## 🇬🇧 English Version

**TL;DR:** Refactored AI agent workspace by moving 142 lines (73%) of technical documentation from auto-loaded MEMORY.md to on-demand docs. Key insight: trust model drives architecture — force-load safety rules (agents might skip), on-demand read resources (tasks force retrieval). Validated with real-world credential lookup workflow.

---

### The Problem: Memory Bloat

After two weeks running an AI agent system, our MEMORY.md had grown to **194 lines**.

The file contained:
- ✅ Important decisions and project context (belongs in memory)
- ❌ API key inventory (78 lines of credential paths)
- ❌ Telegram Bot security setup guide (73 lines of step-by-step instructions)
- ❌ Cron job configuration manual (122 lines of technical reference)

**The question**: Should all this live in auto-loaded workspace context?

---

### The Design Principle: Trust Model

The breakthrough came when discussing *why* some content should be force-loaded vs referenced:

**Force-load (auto-loaded system files):**
- Content the agent **might skip if optional**
- Safety protocols, mandatory workflows, core operational rules
- Example: Critical safety checklists (might be skipped if not enforced)

**On-demand read (docs references):**
- Content **task requirements force you to find**
- Technical references, credential inventories, setup guides
- Example: API key locations (can't complete task without finding it)

**The insight**: This is a **trust model** design decision.

> "We don't trust the agent to proactively read safety rules, but we trust task pressure to force resource lookups."

---

### The Refactoring

#### Phase 1: API Keys Inventory

**Before** (MEMORY.md):
```markdown
## 🔑 API Keys & Config

### Deepgram (Speech-to-Text)
- Location: `credentials/deepgram_api_key`
- Status: ✅ Tested, Chinese transcription working
- Usage: Telegram/WhatsApp voice message auto-transcription

### AlphaVantage (Financial Data)
- Location: `credentials/alphavantage_api_key`
- Usage: Backup financial data source
- Status: ✅ Configured

[... 76 more lines ...]
```

**After** (MEMORY.md):
```markdown
## 🔑 API Keys & Credentials

**Unified path**: `credentials/`
**Full inventory**: `docs/OpenClaw/API-Keys-Inventory.md`
```

**New file** (`docs/OpenClaw/API-Keys-Inventory.md`): 122 lines with full details, usage examples, security rules.

**Savings**: 78 lines → 13 lines (83% reduction)

---

#### Phase 2: Telegram Bot Security Config

**Before**: 73 lines of setup instructions (BotFather settings, Privacy Mode, groupPolicy, etc.)

**After**: 9-line reference pointing to `docs/OpenClaw/Telegram-Bot-Security.md`

**Savings**: 88% reduction

---

#### Phase 3: Cron Job Configuration Guide

**Before**: 122 lines in AGENTS.md (formats, delivery modes, testing protocols, error cases)

**After**: 12-line reference pointing to `docs/OpenClaw/Cron-Job-Guide.md`

**Savings**: 90% reduction

---

#### Special Case: File Deletion Protocol

**Moved FROM**: MEMORY.md  
**Moved TO**: Core system files (Safety section)

**Why the opposite direction?**

This is a **mandatory safety checklist** — must be checked before deleting any file:

```markdown
### Pre-deletion Checklist
1. Search all script references
2. Check automated job dependencies
3. List file + reference status
4. Wait for explicit user approval
```

**Rationale**: This is a **rule**, not **history** — belongs in force-loaded system files, not memory logs.

---

### Total Impact

**Lines removed from auto-loaded workspace:**
- 142 lines out of 194 (73%)

**New docs created:**
1. `OpenClaw/API-Keys-Inventory.md` (122 lines)
2. `OpenClaw/Telegram-Bot-Security.md` (118 lines)
3. `OpenClaw/Cron-Job-Guide.md` (detailed reference)

---

### Real-World Validation

**Test scenario**: User asks "Where's the AlphaVantage API key?"

**Workflow (successful)**:
1. Agent checks MEMORY.md → finds reference to inventory
2. Reads `docs/OpenClaw/API-Keys-Inventory.md`
3. Locates path in credentials directory
4. Reads credential file
5. Returns key to user

**Time**: ~3 seconds  
**Token overhead**: Minimal (only loaded when needed)

**Proof**: On-demand reading works in practice.

---

### Lessons Learned

#### 1. System Files = Maps, Not Encyclopedias

Core system files (configuration, memory, identity) should be:
- **Pointers** to detailed docs
- **Essential rules** that must be seen every session
- **Maps** showing where to find things

NOT:
- Step-by-step setup guides
- Exhaustive technical references
- Historical archives of every decision

---

#### 2. Trust Model Drives Architecture

Ask: **"Will the agent skip this if it's optional?"**

- **Yes** (safety rules, mandatory protocols) → Force-load
- **No** (task pressure forces lookup) → On-demand

Example:
- ❌ Critical safety protocols → Must auto-load (easy to overlook)
- ✅ API key inventory → Can be referenced (can't complete task without it)

---

#### 3. Validate with Real Scenarios

Don't assume on-demand works — **test it**:

- User asks for credential → Agent successfully retrieves it
- Another agent needs API key → Follows reference chain successfully

**All validated** ✅

---

#### 4. Safety Rules ≠ Resources

**Safety rules** (must auto-load):
- Critical operational protocols
- Mandatory approval workflows
- Security guidelines

**Resources** (can be on-demand):
- API key paths
- Technical setup guides
- Configuration references

**Rule of thumb**: If skipping it could cause damage → force-load.

---

### When NOT to Offload

**Keep in auto-loaded workspace when**:
1. **Mandatory enforcement** (safety checklists, approval gates)
2. **Identity and personality** (agent character/tone)
3. **Core operating principles** (behavioral rules)
4. **Recent critical decisions** (past 7-14 days in memory)

**Offload to knowledge-base when**:
1. **Reference documentation** (setup guides, API inventories)
2. **Historical context** (older than 2 weeks, still searchable)
3. **Entity-specific info** (project status files)
4. **Rarely accessed but important** (disaster recovery, annual reviews)

---

### Future Work

**Next candidates for offload**:
- Workspace organization guides
- Historical evolution of automated tasks
- Per-skill troubleshooting guides

**Monitoring**:
- Track on-demand read success rate
- Measure token savings in typical sessions
- Identify cases where agents fail to find needed info

---

### Conclusion

Refactoring AI agent memory isn't just about line count — it's about **designing for trust**.

**Force-load what agents might skip. Reference what tasks force them to find.**

The 73% reduction in auto-loaded content wasn't the goal — it was a natural outcome of applying the trust model consistently.

And the real test? When the user asks "Where's my API key?" — the agent finds it in 3 seconds, following the reference chain we designed.

**That's when you know the architecture works.**

---

## 🇨🇳 中文版本

**一句话总结：** 将 AI agent workspace 中 142 行（73%）技术文档从自动加载的 MEMORY.md 迁移到按需读取的文档。核心洞察：信任模型驱动架构设计 — 强制加载安全规则（agent 可能跳过），按需读取资源（任务压力强制查找）。通过真实凭据查找工作流验证有效性。

---

### 问题：记忆膨胀

运行 AI agent 系统两周后，MEMORY.md 膨胀到 **194 行**。

文件内容包含：
- ✅ 重要决策和项目上下文（属于记忆）
- ❌ API key 清单（78 行凭据路径）
- ❌ Telegram Bot 安全配置指南（73 行分步说明）
- ❌ Cron job 配置手册（122 行技术参考）

**核心问题**：这些内容都应该放在自动加载的 workspace context 吗？

---

### 设计原则：信任模型

讨论"为什么某些内容应该强制加载 vs 引用"时，我们找到了关键洞察：

**强制加载**（自动加载的系统文件）：
- Agent **可能会跳过的内容**
- 安全协议、强制工作流、核心运营规则
- 示例：关键安全检查清单（如果不强制可能被跳过）

**按需读取**（docs 引用）：
- **任务需求强制 agent 去找的内容**
- 技术参考、凭据清单、配置指南
- 示例：API key 位置（不找到就完成不了任务）

**核心洞察**：这是一个**信任模型**设计决策。

> "我们不信任 agent 会主动阅读安全规则，但我们信任任务压力会强制它查找资源。"

---

### 重构过程

#### 阶段 1：API Keys 清单

**之前**（MEMORY.md）：
```markdown
## 🔑 API Keys & 配置

### Deepgram (语音转文字)
- 存储位置: `credentials/deepgram_api_key`
- 状态: ✅ 已测试，可正常转录中文
- 用途: Telegram/WhatsApp 语音消息自动转录

### AlphaVantage (金融数据)
- 存储位置: `credentials/alphavantage_api_key`
- 用途: 备用金融数据源
- 状态: ✅ 已配置

[... 76 行 ...]
```

**之后**（MEMORY.md）：
```markdown
## 🔑 API Keys & 凭据

**统一路径**: `credentials/`  
**详细清单**: `docs/OpenClaw/API-Keys-Inventory.md`
```

**新文件**（`docs/OpenClaw/API-Keys-Inventory.md`）：122 行完整细节，包含使用示例和安全规则。

**节省**：78 行 → 13 行（减少 83%）

---

#### 阶段 2：Telegram Bot 安全配置

**之前**：73 行配置说明（BotFather 设置、Privacy Mode、groupPolicy 等）

**之后**：9 行引用指向 `docs/OpenClaw/Telegram-Bot-Security.md`

**节省**：减少 88%

---

#### 阶段 3：Cron Job 配置指南

**之前**：AGENTS.md 中 122 行（格式、delivery 模式、测试协议、错误处理）

**之后**：12 行引用指向 `docs/OpenClaw/Cron-Job-Guide.md`

**节省**：减少 90%

---

#### 特殊案例：文件删除协议

**从**：MEMORY.md  
**移到**：核心系统文件（安全章节）

**为什么反向移动？**

这是一个**强制安全检查清单** — 删除任何文件前必须遵守：

```markdown
### 删除前检查清单
1. 搜索所有脚本引用
2. 检查自动化任务依赖
3. 列出文件 + 引用状态
4. 等待用户明确批准
```

**理由**：这是**规则**，不是**历史** — 属于强制加载的系统文件，而非记忆日志。

---

### 总体影响

**从自动加载 workspace 移除的行数：**
- 194 行中移除 142 行（减少 73%）

**新建 docs 文档：**
1. `OpenClaw/API-Keys-Inventory.md`（122 行）
2. `OpenClaw/Telegram-Bot-Security.md`（118 行）
3. `OpenClaw/Cron-Job-Guide.md`（详细参考）

---

### 真实场景验证

**测试场景**：用户问 "AlphaVantage API key 在哪？"

**工作流**（成功）：
1. Agent 检查 MEMORY.md → 找到清单引用
2. 读取 `docs/OpenClaw/API-Keys-Inventory.md`
3. 定位凭据目录中的路径
4. 读取凭据文件
5. 返回 key 给用户

**耗时**：约 3 秒  
**Token 开销**：极小（仅在需要时加载）

**证明**：按需读取在实践中有效。

---

### 核心教训

#### 1. 系统文件 = 地图，不是百科全书

核心系统文件（配置、记忆、身份）应该是：
- **指针**指向详细文档
- **必要规则**每次 session 必须看到
- **地图**告诉你去哪找东西

**不应该是**：
- 分步配置指南
- 详尽技术参考
- 所有决策的历史归档

---

#### 2. 信任模型驱动架构

问：**"Agent 会跳过这个吗？"**

- **会**（安全规则、强制协议）→ 强制加载
- **不会**（任务压力强制查找）→ 按需读取

示例：
- ❌ 关键安全协议 → 必须自动加载（容易被忽略）
- ✅ API key 清单 → 可以引用（不找到就完成不了任务）

---

#### 3. 真实场景验证

别假设按需读取一定行 — **实际测一测**：

- 用户问凭据 → Agent 成功检索
- 另一个 agent 需要 API key → 成功跟着引用链找到

**验证通过** ✅

---

#### 4. 安全规则 ≠ 资源

**安全规则**（必须自动加载）：
- 关键操作协议
- 强制审批流程
- 安全指南

**资源**（可以按需）：
- API key 路径
- 技术配置指南
- 配置参考

**经验法则**：跳过它会造成损害 → 强制加载。

---

### 何时不应该 Offload

**保留在自动加载 workspace：**
1. **强制执行**（安全检查清单、审批流程）
2. **身份和性格**（agent 的性格/语气）
3. **核心操作规则**（行为准则）
4. **近期关键决策**（过去 1-2 周的重要记录）

**迁移到 docs：**
1. **参考文档**（配置指南、API 清单）
2. **历史上下文**（超过 2 周，但仍可搜索）
3. **领域特定信息**（项目配置、环境细节）
4. **低频但重要**（灾难恢复、年度审查）

---

### 未来工作

**下一批 offload 候选：**
- Workspace 组织指南
- 自动化任务的历史演进
- 每个 skill 的故障排除指南

**监控指标：**
- 跟踪按需读取成功率
- 测量典型 session 的 token 节省
- 识别 agent 无法找到所需信息的案例

---

### 结论

重构 AI agent 记忆不只是减行数 — 核心是**为信任而设计**。

**强制加载 agent 可能跳过的，引用任务会强制它找的。**

减少 73% 自动加载内容不是目标 — 是应用信任模型后的自然结果。

真正的测试？用户问"我的 API key 在哪" — agent 3 秒内找到，顺着我们设计的引用链。

**这才说明架构有效。**
