# Build Your Own AI Middle Platform

## How we run 10 AI agents 24/7 on a single laptop — and how you can too.

> **Author**: Jason Wang (王磊), Founder & Architect @ QuanXun IoT (FusionConnect)
> **License**: MIT | **Date**: 2026-07-18 | **Version**: v2.2
> **GitHub**: [github.com/leijasonw/ai-middle-platform](https://github.com/leijasonw/ai-middle-platform)

---

## The TL;DR

```text
 10 AI agents  |  66+ tools  |  109K knowledge base  |  $50-100/month
────────────────────────────────────────────────────────────────────────
  1 laptop     |  33 days to rebuild a BOSS system  |  1 month production
────────────────────────────────────────────────────────────────────────
  No VC funding  |  No cloud bill  |  No 10-person team  |  Just strategy
```

We run a production AI middle platform — 10 Feishu (Lark) AI Agents covering finance,
R&D, operations, marketing, sales, investment, and supply chain — on a single
ThinkPad X1 Extreme. Total API cost: ~$50-100/month. The equivalent in human
headcount would be a 10-person team costing $30,000-80,000/month.

This document is not a product pitch. It's a **playbook** — the architecture
decisions, the mistakes, the surprising wins, and the concrete steps to build
your own.

---

## 1. Why we built this

### The problem

QuanXun IoT is a small company. We build IoT connectivity solutions — eSIM,
device management, data pipelines. Like every startup, we needed:

- 24/7 customer support (in Chinese)
- Financial analysis and FP&A
- Code review and architecture
- Marketing content and social media
- Investment research
- Operations and supply chain management

A 10-person professional team would cost us $30k-80k/month. We didn't have that.
What we had was: one experienced engineer, one ThinkPad laptop, and the
willingness to try something unconventional.

### The insight

```text
  Conventional wisdom:  "Hire people for each function."
  Our thesis:           "Build agents for each function."

  Key assumption: if an agent can handle 80% of routine work in a domain,
  one human + 10 agents = 10-person team output at 1% of the cost.
```

### The constraint

We committed to **zero cloud infrastructure**. Everything had to run on a single
development laptop. No Kubernetes. No GPU instances. No dedicated servers.
This constraint turned out to be our biggest advantage — it forced simplicity.

---

## 2. The journey: From zero to production

```text
 Timeline
 ┌─────────────────────────────────────────────────────────────────┐
 │  2024: Experiment with single-agent chatbots (useful but weak)  │
 │  2025: Discover AI agent frameworks (LangChain, AutoGPT)        │
 │  2025.12: Find Hermes Agent — lightweight, MCP-native           │
 │  2026.04: First 3 agents in production (customer support)        │
 │  2026.06: 10 agents, all business functions covered              │
 │  2026.07: BOSS system rebuilt 33 days (was 11 months old PHP)   │
 │  2026.07+: Productization — turning this into a deliverable      │
 └─────────────────────────────────────────────────────────────────┘
```

---

## 3. Architecture: The six-layer model

```text
╔══════════════════════════════════════════════════════════════════════════════════╗
║                AI Middle Platform · Six-Layer Architecture                       ║
║         "The model is the brain. The MCPs are the hands.                         ║
║          The platform is the team."                                             ║
╚══════════════════════════════════════════════════════════════════════════════════╝

┌─ Layer 1: User Access ──────────────────────────────────────────────────────────┐
│                                                                                  │
│    ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                         │
│    │  Feishu Bot  │  │  Web Admin   │  │  REST API    │                         │
│    │  ×10 Apps    │  │  (planned)   │  │  (planned)   │                         │
│    └──────┬───────┘  └──────┬───────┘  └──────┬───────┘                         │
└──────────┼──────────────────┼──────────────────┼────────────────────────────────┘
           │                  │                  │
           ▼                  ▼                  ▼
┌─ Layer 2: Gateway Layer ────────────────────────────────────────────────────────┐
│  Ports 13000-13009 → Each maps to one agent profile                             │
│  Framework: Hermes Gateway v0.18.2 · Isolation: Independent process per agent   │
└─────────────────────────────────────────────────────────────────────────────────┘
           │
           ▼
┌─ Layer 3: Agent Layer ──────────────────────────────────────────────────────────┐
│  10 profiles × 43+ agency roles × 188 global agencies                            │
│  Each: system prompt + model config + tool grant + Feishu identity              │
└─────────────────────────────────────────────────────────────────────────────────┘
           │
           ▼
┌─ Layer 4: MCP Tool Layer ───────────────────────────────────────────────────────┐
│                                                                                  │
│  ╔══════════════════════════════════════════════════════════════════════════╗   │
│  ║  MCP Factory (Core Innovation)                qxmcp CLI                  ║   │
│  ║  ┌───────────────────────────────────────────────────────────────────┐ ║   │
│  ║  │  scan → discover     install → deploy    verify → sandbox-test    │ ║   │
│  ║  │  bind  → agent-link  update  → upgrade   status → health-check    │ ║   │
│  ║  └───────────────────────────────────────────────────────────────────┘ ║   │
│  ╚══════════════════════════════════════════════════════════════════════════╝   │
│                                                                                  │
│  11 MCP Servers · 66+ tools                                                      │
│  AnySearch(web), Scrapling(crawl), Qdrant(search), Feishu(docs),                 │
│  GitHub(code), Tushare(finance), Draw.io(diagrams), OpenMontage(video),          │
│  Agent-Reach(social), AI Berkshire(invest), codebase-memory(code intel)          │
└─────────────────────────────────────────────────────────────────────────────────┘
           │
           ▼
┌─ Layer 5: Knowledge & Data Layer ───────────────────────────────────────────────┐
│                                                                                  │
│  Qdrant vector DB · 109,305 entries · 2560-dim embeddings · 4 pipelines         │
│  Local Ollama (qwen3-embedding:4b) · Two-stage review: candidate→published       │
│                                                                                  │
│  ┌────────────────────────────────────────────────────────────────────────┐    │
│  │  Knowledge Auto-Growth System                                          │    │
│  │                                                                         │    │
│  │  Chat → Agent captures → Qdrant writes → All agents see → Organization │    │
│  │  learns                                                                 │    │
│  │                                                                         │    │
│  │  "Today a colleague asks about expenses → Agent answers from KB.        │    │
│  │   Tomorrow CEO changes market strategy → KB auto-updates → 3 days      │    │
│  │   later all agents know."                                               │    │
│  └────────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────────┘
           │
           ▼
┌─ Layer 6: Model Layer (Hybrid Architecture) ─────────────────────────────────────┐
│                                                                                  │
│    ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐             │
│    │  Local (Free)     │  │  Cloud (Cheap)    │  │  Cloud (Power)   │             │
│    │  ─────────────── │  │  ─────────────── │  │  ─────────────── │             │
│    │  Ollama           │  │  DeepSeek Flash   │  │  DeepSeek Pro    │             │
│    │  qwen3-embed:4b   │  │  3 RPM free tier  │  │  60 RPM          │             │
│    │  Sensitive data    │  │  Daily tasks      │  │  Complex analysis│             │
│    │  Zero API cost     │  │  $50-100/month    │  │  On-demand       │             │
│    └──────────────────┘  └──────────────────┘  └──────────────────┘             │
│                                                                                  │
│    ┌──────────────────┐                                                         │
│    │  Cloud (Special)  │    Smart Routing:                                       │
│    │  ─────────────── │      Knowledge → Local (free, private)                 │
│    │  GLM-5.2          │      Daily use → DS Flash (cheap, fast)                │
│    │  Chinese content  │      Deep analysis → DS Pro (powerful, on-demand)      │
│    │  Zhipu API        │      Chinese content → GLM-5.2 (specialized)           │
│    └──────────────────┘                                                         │
│                                                                                  │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │  The real stack: what's actually running on the laptop                    │  │
│  │                                                                          │  │
│  │  10 independent Hermes processes. Each one in its own terminal:           │  │
│  │    hermés serve --profile default       --port 13000                     │  │
│  │    hermés serve --profile caiwuzhushou  --port 13001                     │  │
│  │    hermés serve --profile zhuli         --port 13002                     │  │
│  │    ... (10 in total, ports 13000-13009)                                   │  │
│  │                                                                          │  │
│  │  NOT Docker. NOT a wrapper. NOT a SaaS shell. NOT WorkBuddy.             │  │
│  │  Each agent IS the Hermes process. The Feishu bot is just its channel.    │  │
│  │                                                                          │  │
│  │  Models in use (all via API, no local inference except embeddings):      │  │
│  │    platform-director → DeepSeek V4 Pro (the one writing this)            │  │
│  │    default/caiwu/zhuli/product/ecom/ops → DeepSeek V4 Flash              │  │
│  │    stockexpert/manager → DeepSeek V4 Pro                                  │  │
│  │    engineeringdir/marketingdir → GLM-5.2 (Zhipu API)                     │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. What we learned (7 hard-won lessons)

### 4.1 The tools matter more than the model

When we started, we thought the LLM was the differentiator. Wrong. The tools
are what make an agent useful. Without tools, a model is an expensive text
generator. With tools, it's an autonomous worker.

Our most-used tools, ranked by call frequency: Scrapling (web crawling) →
Feishu (documents) → Tushare (finance) → AnySearch (web search) → Qdrant (KB).

**Advice**: Pick your MCP servers carefully. A $0 tool that saves 10 minutes/day
is more valuable than a $100/month model with slightly better reasoning.

### 4.2 One agent per domain, not one agent for everything

The single most impactful decision: giving each business domain its own
dedicated agent with its own Feishu bot identity. Users naturally adapt —
they know to ask "财务助手" for expenses, "市场总监" for content.

No prompt bloat. Each system prompt stays under 2,000 tokens.

**Advice**: Fight the urge to build one super-agent. Build many small ones.

### 4.3 Most of the cost is in API calls, not hardware

```text
  Monthly costs:   Laptop $0 | DS Flash $30-60 | DS Pro $10-20 | GLM $10-20
  Total: ~$58-112/month — 95-98% cheaper than hiring humans
```

**Advice**: Optimize for token usage, not compute.

### 4.4 The knowledge base is more important than the agent

We have 109,305 knowledge entries. We spent more time building the ingestion
pipeline (4 automated sources, 30-second indexing) than tuning any single agent.

When the knowledge base reflects the organization, every new agent hits the
ground running.

### 4.5 Fallback chains are non-negotiable

Primary model fails → backup model → graceful error. In one month of production,
the primary model has hit rate limits ~5-10 times. Each time, the fallback
kicked in silently. Users never noticed.

### 4.6 You don't need Kubernetes (and probably don't need Docker either)

Our entire system runs on WSL: a VBS script on Windows boot starts 10 gateway
processes and 7 background services. Zero containerization. Docker would add
complexity without value for a single-machine deployment.

**Advice**: Add architectural overhead only when your constraints demand it.

---

## 5. The proof: BOSS system rewrite (33 days)

```text
┌────────────────────────────────────────────────────────────────────────────────┐
│                      BOSS System: Before vs After                               │
├────────────────────────────────┬───────────────────────────────────────────────┤
│  Old (PHP ThinkPHP 3.x)        │  New (Java Spring Boot + AI Agents)            │
├────────────────────────────────┼───────────────────────────────────────────────┤
│  11 months development         │  33 days (AI-assisted)                         │
│  3-person external team        │  1 architect + AI agents                       │
│  50 commits                    │  193 commits                                   │
│  11,100 files / 458 MB         │  1,813 files / 21 MB                           │
│  0 tests                       │  241 test files (Spock+Groovy+H2)              │
│  MVC (389 controllers)         │  DDD 4-layer (domain/app/infra/interfaces)     │
│  No API docs                   │  Swagger + API.md                              │
│  No idempotency                │  idempotencyKey + retry_count                   │
│  No audit trail                │  TraceFilter + async logging                    │
└────────────────────────────────┴───────────────────────────────────────────────┘
```

11 months → 33 days. Not because AI writes code faster, but because AI agents
removed coordination overhead — no standups, no spec reviews, no PR waiting.

---

## 6. Seven pillars: What makes this platform irreplaceable

After a month in production, we've identified seven pillars that form our
**unreplicable competitive moat**. Any competitor can copy one pillar, but
no one can copy how they work together:

```text
┌────────────────────────────────────────────────────────────────────────────────────┐
│                                                                                    │
│  Pillar ①  MCP Factory — One-command tool discovery                                │
│  ────────────────────────────────────────────────                                   │
│                                                                                    │
│  Competitors:    Manually configure each MCP (engineer required)                   │
│  Our approach:   qxmcp scan → discover / install → deploy / bind → link            │
│                                                                                    │
│  Why it matters:                                                                   │
│    SMEs have no DevOps. We let them "install a weather MCP" with one sentence.     │
│    Others write shell scripts. We run one command.                                  │
│                                                                                    │
│  OpenHarness inspiration:                                                          │
│    Their MCP client is clean. We extend it into a full package manager with        │
│    sandbox-verify, auto-config, batch-update, and security signatures.             │
│                                                                                    │
├────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                    │
│  Pillar ②  Enterprise Knowledge Base — Locally built, automatically grown         │
│  ────────────────────────────────────────────────                                   │
│                                                                                    │
│  Competitors:    RAG = upload PDF → chunk → search → one-off Q&A                  │
│  Our approach:   Chat → Agent captures → Qdrant writes → All agents see → Org learns│
│                                                                                    │
│  Why it matters:                                                                   │
│    Others have a "document search engine". We have an "organizational nervous      │
│    system." A colleague talks to finance agent → knowledge auto-recorded.          │
│    Next week someone asks the same question → agent already knows.                 │
│                                                                                    │
│  OpenHarness inspiration:                                                           │
│    Their MEMORY.md is elegant for single-agent. We scale it: vector DB,            │
│    multi-source pipelines, two-stage review, cross-agent sharing.                  │
│                                                                                    │
├────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                    │
│  Pillar ③  Open-source model pulling (Ollama layer)                               │
│  ────────────────────────────────────────────────                                   │
│                                                                                    │
│  qxmodel pull llama3 → run locally → zero API cost → data never leaves             │
│                                                                                    │
│  The value is not Ollama itself — it's the "one-command pull + configure + switch" │
│  experience. SMEs are terrified of data leaving their premises. We solve that.    │
│                                                                                    │
├────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                    │
│  Pillar ④  Hybrid local-cloud model architecture                                   │
│  ────────────────────────────────────────────────                                   │
│                                                                                    │
│  Sensitive data  → Local Ollama (free, private)                                   │
│  Daily tasks     → DeepSeek Flash (cheap, 3 RPM free tier)                        │
│  Complex analysis → DeepSeek Pro (powerful, on-demand)                            │
│  Chinese content → GLM-5.2 (specialized, 60 RPM)                                  │
│                                                                                    │
│  Smart routing ensures: data security + cost optimization + capability on demand. │
│                                                                                    │
│  OpenHarness inspiration:                                                          │
│    Their provider workflows (`oh setup`) guide model choice. We extend it:         │
│    automatic model routing based on task type, data sensitivity, and cost.        │
│                                                                                    │
├────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                    │
│  Pillar ⑤  Knowledge auto-precipitation — The hardest part for SMEs               │
│  ────────────────────────────────────────────────                                   │
│                                                                                    │
│  4 automated sources:                                                               │
│    · Daily chat conversations → Agent memory captures                             │
│    · Document inbox → 30-second ingestion pipeline                                 │
│    · Obsidian notes → auto-formatted and indexed                                   │
│    · Manual feeds → batch import                                                   │
│                                                                                    │
│  What SMEs fear most: "Only Lao Wang knows."                                       │
│  Our system makes "what Lao Wang knows" automatically become "what the             │
│  company knows." This is not knowledge management — it's knowledge auto-healing.  │
│                                                                                    │
├────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                    │
│  Pillar ⑥  Code repository & engineering (already mature)                         │
│  ────────────────────────────────────────────────                                   │
│                                                                                    │
│  GitHub MCP (26 tools) + codebase-memory (15 tools)                               │
│  Validated in BOSS rewrite: 33 days vs 11 months. Mature but not core moat.      │
│                                                                                    │
│  OpenHarness inspiration:                                                          │
│    Their plugin compatibility with Claude Code ecosystem → we should do similar   │
│    for Hermes profiles, allowing direct reuse of community Skills.                 │
│                                                                                    │
├────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                    │
│  Pillar ⑦  Natural Language Platform Manager — The crown jewel                    │
│  ────────────────────────────────────────────────                                   │
│                                                                                    │
│  This is where every other platform falls short.                                   │
│                                                                                    │
│  ┌──────────────────────────────────────────────────────────────────────────┐     │
│  │                                                                           │     │
│  │  User: "I need a lawyer who can review contracts."                        │     │
│  │  System: ① Creates agent-lvshi profile                                    │     │
│  │          ② Searches and installs legal MCPs                                │     │
│  │          ③ Configures lawyer system prompt                                 │     │
│  │          ④ Binds Feishu app → New "Lawyer" bot is born                    │     │
│  │          ⑤ Replies: "✅ Lawyer agent ready. Ask me anything."             │     │
│  │                                                                           │     │
│  │  User: "Organize the last 3 months of customer complaints into KB."       │     │
│  │  System: ① Scans Feishu history → extracts complaints                     │     │
│  │          ② Auto-classifies (returns / quality / logistics)                │     │
│  │          ③ Writes to Qdrant → tags (customer/complaint)                  │     │
│  │          ④ Replies: "✅ 312 complaints organized and indexed."           │     │
│  │                                                                           │     │
│  │  User: "Switch all agents to Kimi tomorrow, it's cheaper."                │     │
│  │  System: ① Pulls Kimi API config                                          │     │
│  │          ② Switches all agent model providers                              │     │
│  │          ③ Runs compatibility tests                                       │     │
│  │          ④ Replies: "✅ Switched to Kimi. Est. 30% cost reduction."       │     │
│  │                                                                           │     │
│  │  User: "Show me this week's agent usage and cost."                        │     │
│  │  System: ① Queries 10 agent logs                                          │     │
│  │          ② Calculates token/cost per agent                                 │     │
│  │          ③ Replies with usage report + optimization suggestions           │     │
│  │                                                                           │     │
│  └──────────────────────────────────────────────────────────────────────────┘     │
│                                                                                    │
│  Implementation:                                                                   │
│    Agent "platform-manager" with system-level MCP tools:                           │
│      · qxmcp        — Factory operations (scan/install/bind/update)               │
│      · profile      — Agent creation/deletion/modification                        │
│      · model-mgmt   — Model switching / cost monitoring                           │
│      · knowledge    — Knowledge base operations                                   │
│      · system-stats  — Monitoring and reporting                                   │
│                                                                                    │
│  The ultimate moat:                                                                │
│    Everyone requires "technical people to configure."                              │
│    We say: "CEO just says 'I need a lawyer' → system builds it."                  │
│    We're not selling a tool. We're selling a CTO that you can talk to.            │
│                                                                                    │
│  OpenHarness inspiration:                                                          │
│    Their ohmo "personal agent" concept is the closest thing. We extend it:         │
│    ohmo is a personal assistant. Our platform-manager is an organizational        │
│    architect — it builds, configures, and optimizes the entire AI team.           │
│                                                                                    │
└────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Comparison: How we stack against the alternatives

### 7.1 OpenHarness (HKUDS, 14.5K⭐)

[OpenHarness](https://github.com/HKUDS/OpenHarness) is a brilliant piece of
engineering. 11,733 lines of Python, "The model is the agent, the code is the
harness" — a beautiful formulation. We deeply admire their work.

```text
                  OpenHarness                        QuanXun
                  ─────────────────────────────────────────────────
  Focus:          Single-agent harness                Multi-agent platform
  Philosophy:     Model = Agent, Code = Harness       MCPs = Hands, Platform = Team
  Codebase:       11K lines                           500K+ (Hermes + Platform)
  Agents:         1 + subagent spawn                  10 independent profiles
  MCP:            Manual config                       MCP Factory (auto-discover)
  Knowledge:      MEMORY.md (text file)               Qdrant (109K vectors + 4 pipelines)
  Models:         Any Anthropic/OpenAI-compatible     Hybrid local-cloud (auto-routing)
  Self-growth:    None                                Natural Language Manager
  Dry-run:        ✅ Excellent                        ❌ (planned — adopting!)
  Permissions:    ✅ 3-level + path rules              ❌ (planned — adopting!)
  Skills:         ✅ .md compatible with anthropics    Agency roles (188 global)
                                                                                        
  What we admire:                                                                       
    · Code readability: 11K lines anyone can understand                              
    · Dry-run preview: check before execute — adopting this!                         
    · Multi-level permissions: Default/Auto/Plan + path rules — adopting this!       
    · Skills system: pure Markdown, drop-in loading — studying for compatibility     
    · Plugin ecosystem: Claude Code compatibility — evaluating for Hermes profiles   
                                                                                        
  What we do differently:                                                               
    · Multi-agent team (not sub-agent spawn): complete isolation                    
    · MCP Factory (not manual config): package manager for tools                     
    · Hybrid local-cloud models (not single provider): data privacy + cost optimal   
    · Knowledge auto-growth (not text file): 109K vectors, 4 pipelines, 2-stage review
    · Natural language manager (not CLI config): CEO-friendly self-growing system    
    · Productized delivery (not just open-source): priced, packaged, supported       
```

### 7.2 Dify / Coze

```text
                  Dify / Coze                         QuanXun
                  ─────────────────────────────────────────────────
  Philosophy:     Workflow builder                    Agent team builder
  Architecture:   LLM app platform                    Multi-agent gateway
  Multi-agent:    Workflow DAG                        Independent profiles
  Knowledge:     RAG pipeline                         Vector DB + auto-growth
  Deployment:    Docker Compose                       Bare metal / Docker (optional)
                                                                                        
  Key difference:                                                                       
    Dify/Coze = Build an LLM application.                                              
    QuanXun    = Hire an AI team.                                                       
    Apps get outdated. Teams grow.                                                      
```

### 7.3 Claude Code / Cursor

Interactive coding assistants — powerful but single-user and single-session.
Our agents run 24/7, interact through chat, and handle non-coding domains
(finance, marketing, operations).

---

## 8. The playbook: How to replicate this

### Step 1: Choose your agent framework

We use **Hermes Agent** by Nous Research. Alternatives worth evaluating:

| Framework | When to choose |
|-----------|---------------|
| Hermes Agent | Multi-profile, MCP-native, lightweight — our choice |
| OpenHarness | Single-agent, hackable Python, beautiful codebase |
| Claude Code | Best model (Claude), but single-user only |
| Dify | Workflow-heavy, visual builder, Docker-only |

### Step 2: Set up your knowledge base

This is the foundation — more important than any agent. You need:

1. **A vector database**: Qdrant. Install: `docker run qdrant/qdrant`
2. **An embedding model**: qwen3-embedding:4b via local Ollama
3. **An ingestion pipeline**: Auto-process new documents into the vector DB

**Don't skip the ingestion pipeline.** Manual upload dies within a week.

### Step 3: Define your agent profiles

For each business function:
- System prompt + role description
- Model: cheapest that works for this domain
- MCP tools needed
- Knowledge base filters

Start with 3 agents. Add more only after the first 3 are stable.

### Step 4: Connect to communication platforms

We use Feishu (Lark). For global teams: Telegram / Slack / Discord.
**One bot identity per agent** — they should feel like different people.

### Step 5: Monitor, iterate, and let it grow

Track: cost per agent / response time / fallback rate / user satisfaction.
Iterate on agents people use most. Kill the ones nobody uses.
Configure the **Natural Language Manager** — then the system grows itself.

---

## 9. What OpenHarness taught us (and what we're adopting)

We studied OpenHarness deeply. Here's what we're bringing into our platform:

```text
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│  1. Dry-run preview (adopting)                                          │
│     oh --dry-run → preview agent actions before execution               │
│     We're adding: qx preview "Ask engineer agent to refactor module X"  │
│     → shows what tools would be called, what files would be changed,    │
│       what the estimated cost would be, without executing anything.     │
│                                                                          │
│  2. Three-level permission model (adopting)                             │
│     Default / Auto / Plan + path-level rules                            │
│     Our current model: Feishu permissions + approval smart mode          │
│     Our upgrade: Tiered agent permissions — Standard (interactive),     │
│     Autonomous (sandbox), Read-Only (review)                            │
│                                                                          │
│  3. Skills as .md files (evaluating)                                    │
│     OpenHarness loads .md skills on-demand                              │
│     We have Agency roles (188 global). Can we bridge the two?           │
│     Plan: Make our Agency roles compatible with Skills .md format       │
│     → Community can write skills in a single .md file                  │
│                                                                          │
│  4. Code readability as a feature (philosophy)                          │
│     Their 11K lines inspired us: "What if our MCP factory was just      │
│     5K lines of clean Python?" — We're designing qxmcp with this goal.  │
│                                                                          │
│  5. Provider workflows (extending)                                      │
│     oh setup → guided model selection                                   │
│     We're extending: qx setup → guided agent team setup                 │
│     "How many agents? What domains? Local or cloud? Any special tools?" │
│     → 5 minutes to a running AI team.                                   │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 10. What's next: Open-source & productization roadmap

```text
┌──────────────────────────────────────────────────────────────────────────┐
│  2026                                                                     │
│  7月 ✅ Architecture docs, ADR, OpenHarness comparison                   │
│  8月 ⬜ Hermes submodule + upgrade system                                │
│      ⬜ qxmcp CLI v0.1 (scan/install/bind/status)                       │
│      ⬜ install.sh one-command deploy                                    │
│  9月 ⬜ qxmcp v0.2 (sandbox-verify/update/registry)                     │
│      ⬜ Dry-run preview mode (inspired by OpenHarness)                   │
│      ⬜ Three-level permissions (inspired by OpenHarness)                 │
│      ⬜ Web Admin GUI MVP                                                │
│  10月⬜ 2-3 customer deliveries                                          │
│      ⬜ Community launch (HN/V2EX/Reddit)                                │
│  11月⬜ SaaS subscription launch                                         │
│      ⬜ Natural Language Manager v1.0                                    │
│      ⬜ MCP marketplace (community contributions)                         │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## Appendix: MCP tool ecosystem

```text
┌────────────────────────────────────────────────────────────────────────────────┐
│                    MCP Tool Ecosystem (11 Servers, 66+ Tools)                   │
├────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  AnySearch 🌐        4 tools    Web search, batch, extract, domain discovery    │
│  Scrapling 🕷️       14 tools   Web crawling, screenshot, form filling          │
│  Qdrant 📚            7 tools   Knowledge search, filter, stats                │
│  Feishu 📋           15 tools   Document CRUD, tables, messaging               │
│  GitHub 🐙           26 tools   PR/Issue/Code/Search management                │
│  Tushare 📊          22 tools   Stock data, futures, macro indicators          │
│  Draw.io ✏️          ~3 tools   Architecture diagrams                          │
│  OpenMontage 🎬      52 tools   AI video production                             │
│  Agent-Reach 🌐      ~10 tools  Social media scraping (13+ platforms)          │
│  AI Berkshire 🏦     19 Skills  Value investing research                        │
│  codebase-memory 🧠  15 tools   Code intelligence (semantic search)            │
│                                                                                 │
│  All MIT or Apache-2.0. All runnable on a single laptop.                       │
└────────────────────────────────────────────────────────────────────────────────┘
```

---

## Acknowledgments

- **Nous Research** for Hermes Agent — the best multi-profile agent framework
- **HKUDS** for OpenHarness — a beautiful formulation of what a harness should be.
  We learned from your dry-run, permissions model, and code clarity.
- The creators of each MCP server we use — this ecosystem is what makes everything possible
- Our partners **流金科技** and **浩瀚深度** for being the first validators

---

> *"The model learns. The MCPs act. The platform scales."*
>
> *"Others draw architectures. We build teams."*
>
> *"Others sell tools. We give you a CTO you can talk to."*
>
> — Jason Wang, QuanXun IoT
>
> Questions? [jason@quanxuniot.com](mailto:jason@quanxuniot.com)
