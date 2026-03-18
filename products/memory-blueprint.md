# Memory Blueprint — Complete Memory Architecture for AI Agents

**Version:** 1.0 | **Level:** Advanced | **Setup Time:** 60-90 minutes

The most comprehensive review of memory solutions for OpenClaw multi-agent setups — covering every option from native file-based memory to vector databases, ranked and benchmarked from real production use.

---

## What's In This Package

- Ranked tier list of memory solutions (native, vector, external DB, hybrid)
- Multi-agent architecture patterns (single workspace, shared memory, broadcast)
- Memory configuration templates (MEMORY.md, memory/*.md, memorySearch config)
- OpenClaw native memory system deep-dive with exact config options
- Vector search setup (OpenAI embeddings + BM25 hybrid)
- External memory options (Supabase, Mem0, custom solutions)
- Cross-agent memory sharing patterns
- Session memory compaction guide (what survives vs. what doesn't)
- Performance benchmarks from real deployment

---

# Memory Solutions & Multi-Agent Architecture Compendium

**Compiled:** 2026-03-14
**Source:** Daily Catch 🦞 chat history, Zigelbot session logs, web research
**Purpose:** Comprehensive review of memory solutions for a fresh OpenClaw multi-agent install

---

## Part 1: Memory Solutions (Ranked)

### Tier 1 — Built-In / Native OpenClaw (Start Here)

**1. OpenClaw Native Memory System** ⭐⭐⭐⭐⭐
The filesystem-based memory that ships with OpenClaw. MEMORY.md, memory/*.md, AGENTS.md bootstrap files.
- **What:** Markdown files on disk, loaded at session start, survive compaction
- **Strengths:** Zero dependencies, full agent control, works out of the box
- **Weaknesses:** No automatic extraction, relies on agent discipline to write things down, no cross-agent sharing without shared workspace
- **Key config:** `memorySearch`, `compaction.memoryFlush`, `contextPruning`
- **Link:** https://docs.openclaw.ai/concepts/memory
- **Verdict:** The foundation. Everything else builds on top of this.

**2. OpenClaw Memory Search (Hybrid Vector + BM25)** ⭐⭐⭐⭐⭐
Built-in semantic search over memory files using embeddings.
- **What:** OpenAI or local embeddings index all .md files, hybrid BM25+vector retrieval
- **Config:** `memorySearch.provider: "openai"`, `model: "text-embedding-3-large"`, `hybrid: { vectorWeight: 0.7, textWeight: 0.3 }`
- **Strengths:** Native integration, searches memory + sessions + library, no external DB
- **Weaknesses:** Depends on embedding API quota (we hit OpenAI 429s), SQLite-backed so not distributed
- **Pro tip:** Enable `experimental.sessionMemory: true` to make past conversations searchable
- **Link:** https://docs.openclaw.ai/concepts/memory
- **Verdict:** Essential. Enable session memory search — it's the single biggest win.

**3. Pre-Compaction Memory Flush** ⭐⭐⭐⭐⭐
OpenClaw's built-in safety net that fires a silent agent turn before compaction.
- **What:** When context approaches the limit, agent gets a chance to write critical info to files
- **Config:** `compaction.memoryFlush.enabled: true`, `softThresholdTokens: 12000`
- **Strengths:** Automatic, no external deps, customizable prompt
- **Key insight:** Increase softThresholdTokens (default 4000 → 8000-12000) for more breathing room
- **From VelvetShark masterclass:** "Most people never check this is working or give it enough room to fire"
- **Verdict:** Non-negotiable. Customize the prompt to be specific about what to save.

**4. VelvetShark Memory Masterclass** ⭐⭐⭐⭐⭐
The most authoritative community guide on OpenClaw memory, written by a codebase maintainer.
- **What:** Complete guide covering the 4-layer memory model (bootstrap files → session transcript → LLM context → retrieval index)
- **Key takeaway:** "Instructions typed in conversation don't survive compaction. Put durable rules in files."
- **Three things that matter most:** (1) Put rules in files not chat, (2) Verify memory flush is enabled, (3) Make retrieval mandatory via AGENTS.md
- **Link:** https://velvetshark.com/openclaw-memory-masterclass
- **Verdict:** Required reading for any new install. Follow these 3 rules first.

---

### Tier 2 — Enhanced Memory Layers (Add When Native Isn't Enough)

**5. QMD (Quick Memory Database)** ⭐⭐⭐⭐
Lightweight BM25 search index for structured facts.
- **What:** Local CLI tool that indexes markdown files into a BM25 searchable database
- **Strengths:** Fast, local, no API costs, great for structured entity data
- **Weaknesses:** No semantic search (keyword only), needs manual `qmd update` calls
- **We use it:** For knowledge graph indexing (`life/` entities)
- **Install:** `bun install` (already in our setup)
- **Verdict:** Good complement to vector search. Use for structured data where exact keyword match matters.

**6. Mem0** ⭐⭐⭐⭐
Managed memory layer with automatic extraction and deduplication.
- **What:** API service that automatically extracts, categorizes, and deduplicates memories from conversations
- **Strengths:** Automatic memory extraction (no manual writes needed), deduplication, entity resolution, graph relationships
- **Weaknesses:** External API dependency, cost, potential privacy concerns, another service to manage
- **MCP server available:** Can integrate via MCP protocol
- **Link:** https://mem0.ai / https://github.com/mem0ai/mem0
- **Best for:** Teams who want "set and forget" memory without disciplined file writes
- **Verdict:** Strongest option if you want automatic memory. Consider self-hosted for privacy.

**7. Andy Nguyen's Memory Skill (26K+ Downloads)** ⭐⭐⭐⭐
Community-built memory skill that addresses OpenClaw's native memory pain points.
- **What:** ClawHub skill that improves memory capture, recall, and deduplication
- **Claim:** "OpenClaw's default memory system is broken — requires curating massive MEMORY.md files, duplicate-heavy generation, burns tokens"
- **Downloads:** 26K+ in first week — clearly hits a nerve
- **Install:** ClawHub (`clawdhub install` — search for memory skill by kevinnguyendn)
- **Link:** LinkedIn announcement by Andy Nguyen
- **Verdict:** Worth evaluating. High adoption suggests it solves real pain. Vet before installing (per our security policy).

**8. Gigabrain (Memory OS by @Legendaryyy)** ⭐⭐⭐
Long-term memory layer with typed memory capture.
- **What:** Typed memory capture (facts, decisions, preferences), recall, deduplication, native markdown sync
- **Claims "6 things native OpenClaw memory doesn't have"**
- **GitHub:** ~35 stars, 1 contributor — small project
- **Verdict:** Interesting ideas worth stealing, but too small/unproven for production. Read the README for patterns.

---

### Tier 3 — Graph & Vector Databases (Heavy Infrastructure)

**9. Graphiti (by Zep)** ⭐⭐⭐
Temporal knowledge graph for agent memory.
- **What:** Builds a knowledge graph from conversations with temporal awareness — knows WHEN facts were true
- **Strengths:** Temporal reasoning ("what did the user prefer last month vs now?"), entity relationships, graph queries
- **Weaknesses:** Requires Neo4j, significant infrastructure, overkill for most setups
- **Our assessment (2026-02-01):** "Graphiti/Neo4j overkill for now" — session memory search was the bigger win
- **Link:** https://github.com/getzep/graphiti
- **Best for:** Complex multi-agent systems where temporal entity relationships matter
- **Verdict:** Revisit when you have 6+ agents that need shared understanding of evolving entity states.

**10. Cognee** ⭐⭐⭐
Knowledge graph + vector search memory engine.
- **What:** Builds knowledge graphs from documents and conversations, combines with vector retrieval
- **Strengths:** Automatic knowledge extraction, graph + vector hybrid, good for document-heavy workflows
- **Weaknesses:** Another Python service to run, early stage
- **Link:** https://github.com/topoteretes/cognee
- **Mentioned in Agent Native article** alongside QMD, Mem0, and Obsidian as top options
- **Verdict:** Worth watching. More mature than Gigabrain but less proven than Mem0.

**11. Letta (formerly MemGPT)** ⭐⭐⭐
The OG AI memory architecture — self-editing memory with hierarchical storage.
- **What:** Core memory (in-context, self-editing) + conversational memory (searchable) + archival memory (long-term)
- **Key insight from benchmarking:** "Agents using simple filesystem operations (grep, search_files) achieved 74% accuracy — beating specialized memory tools like Mem0 (68.5%)"
- **Strengths:** Proven architecture, deep research backing, self-editing memory concept
- **Weaknesses:** Heavyweight, designed as its own agent framework not a plugin
- **Link:** https://github.com/letta-ai/letta
- **Verdict:** The research is invaluable (we already use its patterns). As a runtime, it's competing with OpenClaw rather than complementing it.

**12. Qdrant / ChromaDB / Pinecone / Weaviate** ⭐⭐
Standalone vector databases.
- **What:** Dedicated vector stores for embedding-based retrieval
- **Qdrant:** Self-hosted, Rust-based, fast. Best self-hosted option
- **ChromaDB:** Python-native, easy to embed, good for prototyping
- **Pinecone:** Managed service, scales well, costs money
- **Weaviate:** Hybrid search (vector + keyword), self-hosted or cloud
- **Verdict:** OpenClaw already has built-in vector search via SQLite. Only use these if you need: (a) distributed search across machines, (b) massive scale (100K+ documents), or (c) real-time cross-agent sharing.

---

### Tier 4 — Emerging / Just Announced

**13. Memori Labs OpenClaw Plugin** ⭐⭐⭐ (NEW — 2026-03-13)
Just announced yesterday — persistent memory plugin specifically for OpenClaw.
- **What:** "Structured storage, advanced augmentation, intelligent recall, and production observability"
- **Target:** Multi-agent environments specifically
- **Link:** https://nationaltoday.com/us/ca/san-francisco/news/2026/03/13/memori-labs-launches-openclaw-plugin-bringing-persistent-ai-memory-to-multi-agent-gateways/
- **Verdict:** Brand new, unvetted. Watch closely — if it works as advertised, it could be the missing piece for multi-agent memory sharing. Needs security audit before installing.

**14. Obsidian Integration** ⭐⭐⭐
Use Obsidian vault as agent memory backend.
- **What:** Agent reads/writes to an Obsidian vault, leveraging Obsidian's linking, graph view, and plugins
- **Strengths:** Human-readable, great UI for reviewing agent memory, rich plugin ecosystem
- **Weaknesses:** Another app to run, file-locking concerns with concurrent agents
- **Mentioned in Agent Native article** as a viable substrate
- **Verdict:** Great if you already use Obsidian. Otherwise, markdown files in workspace accomplish the same thing.

**15. Lossless Claw (LCM) by Martian Engineering** ⭐⭐⭐⭐ (NEW — trending)
Lossless Context Management — prevents information loss during compaction.
- **What:** Plugin that preserves full context fidelity through compaction cycles instead of lossy summarization
- **Shared by:** steipete (OpenClaw creator) — "If you are annoyed that your crustacean is forgetful after compaction, give this a try!"
- **Engagement:** 146 reposts, 1690 likes, 2710 bookmarks, 108K views — massive traction
- **Link:** https://github.com/martian-engineering/lossless-claw
- **Tweet:** https://x.com/steipete/status/2032861327967072671
- **Verdict:** Directly addresses THE core pain point (compaction amnesia). steipete's endorsement carries weight. High priority to evaluate.

**16. MuninnDB** ⭐⭐⭐⭐ (NEW — field-tested by Suede/Orchard)
Local graph memory with MCP server and web UI.
- **What:** Persistent graph-based memory layer with built-in MCP server, web UI for browsing, multiple vault support
- **Key features:** MCP server (any MCP-compatible tool can query it), web UI for visual browsing, multiple vaults (work/personal/project), runs locally on port 8475-8750, OpenAI embedder for semantic search
- **Setup:** `muninn init --tool openclaw --yes` auto-configures the MCP connection in openclaw.json. Restart gateway and it works
- **Version:** 0.4.1-alpha (early but functional)
- **Field report (Suede, 2026-03-14):** "OpenClaw still handles session memory, bootstrap files, and compaction flushes — that didn't change. But MuninnDB sits alongside as the persistent graph layer that any MCP-compatible tool can query."
- **Link:** https://github.com/scrypster/muninndb
- **Verdict:** The MCP angle is compelling — makes agent memory queryable by any tool in your stack, not just OpenClaw. Real-world tested by group member. Worth trying.

**17. SynaBun** ⭐⭐⭐ (NEW — mentioned in steipete thread)
Local SQLite + vector embeddings as MCP server for Claude Code.
- **What:** Semantic recall instead of flat file search, designed for vibe coding workflows
- **Claim:** "The compaction problem is solved"
- **Link:** https://synabun.ai
- **Tweet:** https://x.com/SynabunAI/status/2032876952986771882
- **Verdict:** Interesting but more focused on Claude Code than OpenClaw. Worth noting if you use both.

**18. muratcankoylan/memory-systems (SkillsMP)** ⭐⭐
Agent skill for designing memory architectures.
- **What:** Templates for short-term, long-term, and graph-based memory architectures
- **Link:** https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering/tree/main/skills/memory-systems
- **Verdict:** More of a reference/template than a solution. Good for learning patterns.

---

## Part 2: Key Patterns & Insights

### The "Write It Down" Discipline (Most Important Pattern)
From our own research (memory/memory-improvement-research.md, 2026-01-31):
- **Letta's benchmarking proved:** simple file operations beat specialized memory tools
- **The 30-Second Rule:** If info would be painful to lose, write it within 30 seconds
- **Write-On-Receive triggers:** credentials, decisions, setup completions, preferences, state changes
- **Cross-reference on write:** daily log + TOOLS.md + MEMORY.md + relevant project file

### The Four Memory Layers (VelvetShark Model)
1. **Bootstrap files** (SOUL.md, AGENTS.md, etc.) — permanent, reloaded every session
2. **Session transcript** (JSONL) — semi-permanent, can be compacted
3. **LLM context window** — temporary, fixed size, overflows
4. **Retrieval index** (memory_search) — permanent, rebuilt from files

### Memory Contract (Agent Native Article)
Force the agent to:
1. **Search before acting** — mandatory memory_search before answering from "memory"
2. **Persist constraints** — write decisions/rules to files, never rely on chat context
3. **Checkpoint at flush** — custom flush prompt that saves specific categories of info

---

## Part 3: Multi-Agent Architecture Recommendations

### For a Fresh OpenClaw Install with Multiple Agents

**Recommended Architecture:**

```
┌─────────────────────────────────────────────┐
│                 SHARED LAYER                 │
│  ~/.openclaw/workspace/                      │
│  ├── MEMORY.md (global shared memory)        │
│  ├── BULLETIN.md (cross-agent announcements) │
│  ├── memory/ (daily logs, entity files)      │
│  ├── life/ (knowledge graph)                 │
│  └── shared/ (inter-agent message drops)     │
└─────────────────────────────────────────────┘
         ↕              ↕              ↕
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│  Agent: Main │ │ Agent: Dev   │ │ Agent: Ops   │
│  Opus 4.6    │ │ Codex/Sonnet │ │ Haiku/Sonnet │
│  Telegram DM │ │ Discord/GH   │ │ Cron/Infra   │
│  Personal    │ │ Coding tasks │ │ Monitoring   │
│  assistant   │ │ PR reviews   │ │ Health       │
└──────────────┘ └──────────────┘ └──────────────┘
```

### Agent Configuration Tiers

**Agent 1: Main (Personal Assistant)**
- Model: Claude Opus 4.6
- Channels: Telegram (DM + groups)
- Memory: Full MEMORY.md access, daily logs, knowledge graph
- Skills: All skills, full tool access
- Role: Primary interface, decision-making, personal tasks

**Agent 2: Dev (Coding & Engineering)**
- Model: Claude Sonnet 4.6 or Codex
- Channels: Discord, GitHub webhooks
- Memory: Shared workspace (read), own agent memory (write)
- Skills: coding-agent, github, gh-issues
- Role: PR reviews, feature development, bug fixes

**Agent 3: Ops (Infrastructure & Monitoring)**
- Model: Claude Haiku (cost-efficient for routine checks)
- Channels: Cron jobs, alerting
- Memory: Shared workspace (read), BULLETIN.md (write)
- Skills: healthcheck, cloudflare, monitoring
- Role: System health, security checks, update management

**Agent 4: Research (Optional)**
- Model: Gemini 2.5 Pro (long context) or Opus
- Channels: Triggered by main agent
- Memory: Library access, research output files
- Skills: web_search, web_fetch, pdf analysis
- Role: Deep research tasks, document analysis

### Memory Strategy for Multi-Agent

1. **Shared workspace** — All agents read the same workspace directory
2. **BULLETIN.md** — Cross-agent announcements (timestamped entries, max ~10)
3. **Per-agent memory directories** — Each agent gets `memory/agent-{name}/` for private context
4. **Entity files** — Shared `memory/entities/` for people, projects, services
5. **Memory search** — All agents use the same embedding index (built from shared files)
6. **Flush prompts** — Each agent has a customized flush prompt for its domain

### ⚠️ Known Pitfalls

- **Shared workspace race condition:** `agents delete` can trash shared workspace (GitHub issue #39701). Use separate workspaceDirs or careful agent isolation.
- **Memory flush timing:** Sub-agents may not get flush prompts. Save critical info explicitly.
- **Token burn:** Running Opus on cron jobs is expensive. Use Haiku for mechanical tasks.
- **Cross-agent context:** Agents can't read each other's session history. Use files for coordination.

---

## Part 4: What I'd Recommend (Zigelbot's Take)

### If Starting Fresh Today:

1. **Start with native OpenClaw memory** — it's genuinely good
2. **Follow VelvetShark's 3 rules** — files not chat, verify flush works, mandatory retrieval
3. **Enable session memory search** — biggest single improvement
4. **Try Lossless Claw** — steipete-endorsed, directly fixes compaction amnesia (the #1 pain point)
5. **Consider MuninnDB** as a persistent graph layer — MCP server means any tool in your stack can query agent memory
6. **Add Mem0 as a layer** (self-hosted) — for automatic extraction when you don't want to rely on agent discipline
7. **Skip Graphiti/Neo4j** unless you have 5+ agents that need temporal entity tracking
8. **Watch the Memori Labs plugin** — if it delivers on cross-agent memory, it solves the hardest problem
9. **Use QMD** for structured entity/project data alongside vector search

### The 80/20:
- 80% of memory quality comes from **discipline** (writing things down)
- 20% comes from **infrastructure** (vector search, graphs, specialized tools)

Don't over-engineer the 20% before nailing the 80%.

---

## References

- OpenClaw Memory Docs: https://docs.openclaw.ai/concepts/memory
- VelvetShark Masterclass: https://velvetshark.com/openclaw-memory-masterclass
- Agent Native "Memory Systems That Don't Forget": https://agentnativedev.medium.com/openclaw-memory-systems-that-dont-forget-qmd-mem0-cognee-obsidian-4ad96c02c9cc
- Multi-Agent Orchestration Guide: https://zenvanriel.com/ai-engineer-blog/openclaw-multi-agent-orchestration-guide/
- Multi-Agent Team Setup: https://www.mejba.me/blog/openclaw-agent-team-configuration
- Multiple Concurrent Agents: https://www.answeroverflow.com/m/1471453972932984956
- OpenClaw Security Architecture: https://nebius.com/blog/posts/openclaw-security
- Memori Labs Plugin: https://nationaltoday.com/us/ca/san-francisco/news/2026/03/13/memori-labs-launches-openclaw-plugin-bringing-persistent-ai-memory-to-multi-agent-gateways/
- Mem0: https://github.com/mem0ai/mem0
- Graphiti: https://github.com/getzep/graphiti
- Cognee: https://github.com/topoteretes/cognee
- Letta: https://github.com/letta-ai/letta
- Our internal research: memory/memory-improvement-research.md
