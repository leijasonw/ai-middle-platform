# Build Your Own AI Middle Platform

## How we run 10 AI agents 24/7 on a single laptop — and how you can too.

> **Author**: Jason Wang (王磊), Founder & Architect @ QuanXun IoT (FusionConnect)
> **License**: MIT | **Date**: 2026-07-18
> **GitHub**: [github.com/quanxun/ai-middle-platform-playbook](https://github.com/quanxun/ai-middle-platform-playbook)

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

### Phase 1: Exploration (2024-2025)

We tried everything: LangChain (too abstract), AutoGPT (too unreliable),
Dify (too heavy for a single laptop), OpenAI Assistants API (vendor lock-in
concerns). Each taught us something, but none stuck.

### Phase 2: The breakthrough — Hermes Agent (Dec 2025)

We discovered [Hermes Agent](https://hermes-agent.nousresearch.com) by
Nous Research. Three things stood out:

1. **Multi-profile by design** — each agent is a separate profile with its
   own system prompt, model config, and tool set
2. **MCP-native** — Model Context Protocol support out of the box, meaning
   any MCP-compatible tool plugs right in
3. **Lightweight** — runs on a single machine, no Docker required

This was the foundation we'd been looking for.

### Phase 3: The 10-agent ramp (Apr-Jun 2026)

Once we had the foundation, each new agent took about 2-3 days to configure
and tune:

- **default**: General assistant, knowledge base queries
- **caiwuzhushou** (财务): Financial analysis, FP&A, bookkeeping
- **zhuli** (助理): Meeting minutes, document generation
- **stockexpert**: Investment research, market analysis
- **manager**: Business strategy, executive decision support
- **productexpert**: UX research, competitive analysis
- **engineeringdir**: Code review, architecture, infrastructure
- **marketingdir**: Content creation, social media strategy
- **ecommerceops**: E-commerce operations, ASO
- **operationsdir**: Supply chain management

Each agent runs as a separate Hermes profile, connected to Feishu as a
distinct bot identity — so users talk to "财务助手" or "市场总监" as if
they're different people.

### Phase 4: Production validation (1 month+)

The system has been running 24/7 for over a month, handling real business
requests from 8 AM to 10 PM daily. The uptime: essentially 100%
(the laptop hasn't crashed once).

---

## 3. Architecture: The six-layer model

After building this, we can describe the architecture in six clean layers:

```text
╔══════════════════════════════════════════════════════════════════════════════════╗
║                AI Middle Platform · Six-Layer Architecture                       ║
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
│                                                                                  │
│    Ports 13000-13009 → Each maps to one agent profile                           │
│    Framework: Hermes Gateway v0.18.2                                            │
│    Isolation: Each gateway runs as an independent process                        │
└─────────────────────────────────────────────────────────────────────────────────┘
           │
           ▼
┌─ Layer 3: Agent Layer ──────────────────────────────────────────────────────────┐
│                                                                                  │
│    10 profiles × 43+ agency roles × 188 global agencies                          │
│    Each profile: system prompt + model config + tool grant                       │
└─────────────────────────────────────────────────────────────────────────────────┘
           │
           ▼
┌─ Layer 4: MCP Tool Layer ───────────────────────────────────────────────────────┐
│                                                                                  │
│    11 MCP Servers · 66+ tools                                                    │
│    AnySearch(web), Scrapling(crawl), Qdrant(search), Feishu(docs),               │
│    GitHub(code), Tushare(finance), Draw.io(diagrams), OpenMontage(video),        │
│    Agent-Reach(social), AI Berkshire(invest), codebase-memory(code intel)        │
└─────────────────────────────────────────────────────────────────────────────────┘
           │
           ▼
┌─ Layer 5: Knowledge & Data Layer ───────────────────────────────────────────────┐
│                                                                                  │
│    Qdrant vector DB · 109,305 entries · 2560-dim embeddings                     │
│    Local Ollama (qwen3-embedding:4b) · 4 automated ingestion pipelines          │
│    Two-stage review: candidate → published                                      │
└─────────────────────────────────────────────────────────────────────────────────┘
           │
           ▼
┌─ Layer 6: Model Layer ──────────────────────────────────────────────────────────┐
│                                                                                  │
│    DeepSeek Flash → Daily tasks, 3 RPM free tier                                 │
│    DeepSeek Pro   → Complex analysis, 60 RPM                                    │
│    GLM-5.2        → Chinese content, marketing (Zhipu API)                      │
│    Total monthly API cost: ~$50-100                                              │
└─────────────────────────────────────────────────────────────────────────────────┘
```

This is not the only architecture. It's the one that emerged from our constraint
(one laptop, zero cloud, minimal cost). Different constraints would produce
different architectures. The important thing is the **layering principle**:
each layer has a clear responsibility and can be swapped independently.

### Key architectural decisions (and why)

**ADR-1: Hermes Agent over Dify/Coze/OpenClaw**
- Needed: multi-profile isolation, MCP protocol, open source
- Hermes: ✓ Profile-native ✓ MCP-native ✓ Lightweight
- Dify: Docker-heavy, single-tenant
- Coze: Closed source, no self-hosting
- OpenClaw: Security concerns (341 malicious plugins found in ClawHub)

**ADR-2: Multiple single-agent Gateways over shared Gateway**
- Each agent gets its own port (13000-13009)
- Trade-off: More ports, but complete isolation
- A crash in one agent never affects another
- Win: 1-month uptime, zero cascading failures

**ADR-3: Qdrant over Milvus/Weaviate**
- Qdrant runs on a laptop (single binary, no JVM)
- Milvus needs Docker + etcd + minio — overkill
- Weaviate needs Docker — adds complexity
- Qdrant: `docker run qdrant/qdrant` and done

**ADR-4: Dual model strategy (DeepSeek + GLM)**
- DeepSeek Flash for 80% of daily work (cheap, fast)
- DeepSeek Pro for complex analysis
- GLM-5.2 for Chinese-language marketing content
- Total monthly API bill: ~$50-100

**ADR-5: All tools through MCP protocol**
- Every MCP server is a standard interface
- Tools are swappable without changing agent code
- Adding a new tool = adding a new MCP server

---

## 4. What we learned

### 4.1 The tools matter more than the model

When we started, we thought the LLM was the differentiator. Wrong. The tools
are what make an agent useful.

```text
  Model without tools = expensive text generator
  Model with tools    = autonomous worker
  
  Our most used tools, ranked by call frequency:
  1. Scrapling (web crawling) — every agent uses it
  2. Feishu (document operations) — 4 agents core dependency
  3. Tushare (financial data) — 3 agents core dependency
  4. AnySearch (web search) — universal fallback
  5. Qdrant (knowledge retrieval) — default agent primary
```

**Advice**: Pick your MCP servers carefully. A $0 tool (free API) that saves
10 minutes/day is more valuable than a $100/month model with slightly
better reasoning.

### 4.2 One agent per domain, not one agent for everything

The single most impactful decision was giving each business domain its own
dedicated agent with its own Feishu bot identity.

Why this works:

```text
  General assistant:       Tries to do everything, masters nothing.
  Dedicated agents:        Each one has a narrow focus and deep expertise.
  
  Users naturally adapt:   They know to ask "财务助手" for expenses,
                           "市场总监" for content, "ceo" for strategy.
                           
  No prompt bloat:         Each system prompt stays under 2000 tokens.
                           A general-purpose agent would need 10,000+.
```

**Advice**: Fight the urge to build one super-agent. Build many small ones.
The coordination overhead is negligible; the focus gain is enormous.

### 4.3 Most of the cost is in API calls, not hardware

```text
  Monthly costs:
    ThinkPad laptop:           $0 (already owned)
    DeepSeek Flash API:       $30-60
    DeepSeek Pro API:         $10-20  (sporadic use)
    GLM-5.2 API:              $10-20  (marketing only)
    Tushare Pro (finance):    $3-7
    Electricity:               $5
    ─────────────────────────────────
    Total:                    ~$58-112/month
```

The hardware cost is a rounding error. The operating cost is the API calls.
And even at peak usage, it's 95-98% cheaper than hiring humans.

**Advice**: Optimize for token usage, not compute. Use the cheapest model
that gets the job done. Reserve expensive models for tasks that genuinely
need them.

### 4.4 The knowledge base is more important than the agent

We have 109,305 knowledge entries. They cover:

- Product documentation
- Technical specifications
- Customer conversation history
- Industry reports
- Competitor analysis
- Internal SOPs
- Financial records

When a new hire (or a new agent) joins, their effectiveness is directly
proportional to the knowledge they can access. We spent more time building
the ingestion pipeline (4 automated sources, 30-second indexing) than
tuning any single agent.

**Advice**: Start building your knowledge base before you write a single
line of agent code. The agent is the interface; the knowledge is the value.

### 4.5 Fallback chains are non-negotiable

```text
  Primary: DeepSeek Flash
    ↓ (HTTP 429 / timeout)
  Fallback: GLM-5.2
    ↓ (still fails)
  Response: "Service temporarily unavailable"
  
  Every step: logged, monitored, alerted.
```

In one month of production, we've hit the primary model rate limit about
5-10 times. Each time, the fallback kicked in silently. Users never noticed.

**Advice**: Configure fallback chains on day one, not after the first outage.

### 4.6 You don't need Kubernetes (and probably don't need Docker either)

This is the most controversial take. Our entire system runs:

- On WSL (Windows Subsystem for Linux) on a laptop
- Started by a VBS script on Windows boot
- 10 gateway processes, 7 background services
- Zero containerization

Docker would add complexity without value for a single-machine deployment.
We'll add Docker Compose when we productize for multi-machine deployments,
but for now, raw processes are simpler and more debuggable.

**Advice**: Don't let infrastructure complexity eat your innovation budget.
Add architectural overhead only when your constraints demand it.

---

## 5. The proof: BOSS system rewrite (33 days)

The most concrete validation of this approach: rebuilding our entire IoT
operations platform (BOSS system).

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
└────────────────────────────────┴────────────────────────────────────────────────┘
```

**Key stat**: 11 months → 33 days. Not because AI writes code faster (it does),
but because AI agents removed coordination overhead. No standups. No spec
reviews. No pull request waiting. The architect describes the task, the agents
execute, and the review loop is hours instead of days.

---

## 6. Comparison: How we stack against the alternatives

### 6.1 OpenHarness (HKUDS, 14.5K⭐)

[OpenHarness](https://github.com/HKUDS/OpenHarness) is excellent. It's what
we'd use if we were starting today and only needed a single-agent system.
Their "The model is the agent; the code is the harness" is a beautiful
formulation.

```text
                  OpenHarness                        QuanXun
                  ─────────────────────────────────────────────────
  Focus:          Single-agent harness                Multi-agent platform
  Codebase:       11K lines Python                    ~500K lines (Hermes)
  Agents:         1 + subagent spawn                  10 independent profiles  
  Knowledge:      MEMORY.md (text file)               Qdrant (109K vectors)
  Multi-channel:  ohmo (Feishu/Slack/TG/DC)           Feishu native (10 bots)
  Permissions:    3-level + path rules                App-level (Feishu)
  Dry-run:        ✅ oh --dry-run                     ❌ (planned)
  Productized:    ❌ (open source only)                ✅ ¥9,800-58,000
```

**What we admire**: The dry-run preview. The clean permissions model. The
code readability (11K lines!).

**What we do differently**: Multi-agent independence, enterprise knowledge
management, professional MCP tools, product packaging.

### 6.2 Dify / Coze

```text
                  Dify / Coze                         QuanXun
                  ─────────────────────────────────────────────────
  Philosophy:     Workflow builder                    Agent team builder
  Architecture:   LLM app platform                    Multi-agent gateway
  Multi-agent:    Workflow DAG                        Independent profiles
  Knowledge:     RAG pipeline                         Vector DB + pipelines
  Deployment:    Docker Compose                       Bare metal (WSL)
  Cost:           ¥3,000-5,000/mo SaaS                $50-100/mo API
```

Dify and Coze are great for building LLM-powered workflows. We use a
fundamentally different model: instead of building workflows, we hire
agents. Each agent is autonomous, has its own identity, and grows with
use.

### 6.3 Claude Code / Cursor

These are interactive coding assistants — powerful but single-user and
single-session. Our agents run 24/7, interact through chat, and handle
non-coding domains (finance, marketing, operations).

---

## 7. The playbook: How to replicate this

### Step 1: Choose your agent framework

We use **Hermes Agent** by Nous Research. Alternatives worth evaluating:

| Framework | When to choose |
|-----------|---------------|
| Hermes Agent | Multi-profile, MCP-native, lightweight |
| OpenHarness | Single-agent, hackable, Python |
| Claude Code | Best model (Claude), but single-user |
| Dify | Workflow-heavy, visual builder |

### Step 2: Set up your knowledge base

This is the foundation. You need three things:

1. **A vector database**: We use Qdrant. Install is `docker run qdrant/qdrant`
2. **An embedding model**: We use qwen3-embedding:4b via local Ollama
3. **An ingestion pipeline**: Auto-process new documents into the vector DB

**Don't skip the ingestion pipeline.** Manual upload is a dead end.

### Step 3: Define your agent profiles

For each business function, define:
- **Agent profile**: System prompt, role description
- **Model**: Cheapest model that works for this domain
- **Tools**: MCP servers this agent needs
- **Knowledge filters**: What subset of the knowledge base to search

Start with 3 agents. Add more only after the first 3 are stable.

### Step 4: Connect to your communication platform

We use **Feishu (Lark)** because we're in China. If you're elsewhere:
- Telegram for technical teams
- Slack for enterprise
- Discord for community

The key is: **one bot identity per agent**. Users should feel like they're
talking to different people, not a single chatbot with multiple personalities.

### Step 5: Monitor and iterate

Track:
- **Cost per agent** (API tokens used)
- **Response time** (how fast does the agent reply)
- **Fallback rate** (how often does the primary model fail)
- **User satisfaction** (do people actually use it)

Iterate on the agents people use most. Kill the ones nobody uses.

---

## 8. What's next

We're productizing this into a deliverable:

```text
  AI Middle Platform Base:
    Standard ($9,800)   — 3 agents, basic MCP, 5K knowledge
    Professional ($28K) — 8 agents, full MCP, 50K knowledge
    Enterprise ($58K)   — Unlimited agents, custom MCP, unlimited knowledge
```

But honestly, that's just our business. The real reason we're publishing this
is: **we think this model works for many teams**. If you're a small company
with one good engineer and a laptop, you can build what we built. The only
question is whether you start today.

---

## Appendix: The MCP tool ecosystem we use

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
│  Draw.io ✏️          ~3 tools   Architecture diagrams, circuit diagrams        │
│  OpenMontage 🎬      52 tools   AI video production (script→素材→剪辑→成片)    │
│  Agent-Reach 🌐      ~10 tools  Social media scraping (13+ platforms)          │
│  AI Berkshire 🏦     19 Skills  Value investing research (Buffett+Munger)      │
│  codebase-memory 🧠  15 tools   Code intelligence (semantic search, trace)     │
│                                                                                 │
└────────────────────────────────────────────────────────────────────────────────┘
```

All MIT or Apache-2.0 licensed. All runnable on a single laptop.

---

## Acknowledgments

- **Nous Research** for Hermes Agent — the best multi-profile agent framework
- **HKUDS** for OpenHarness — a beautiful formulation of what a harness should be
- The creators of each MCP server we use — this ecosystem is what makes this possible
- Our partners **流金科技** and **浩瀚深度** for being the first to validate this approach

---

> *"Others draw architectures. We build teams."*
>
> *"Others go to the cloud. We use what's on our desk."*
>
> *"Others run demos. We run production."*
>
> — Jason Wang, QuanXun IoT
>
> Questions? [jason@quanxuniot.com](mailto:jason@quanxuniot.com)
