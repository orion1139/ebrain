<div align="center">

<img src="https://img.shields.io/badge/E--Brain-Persistent_Memory_for_AI_Agents-0066ff?style=for-the-badge&logo=rust&logoColor=white" alt="E-Brain">

# E-Brain

### The memory layer that makes AI agents remember everything.

*Across sessions. Across compactions. Across projects.*

[![Built with Rust](https://img.shields.io/badge/Rust-000000?style=flat-square&logo=rust&logoColor=white)](https://www.rust-lang.org/)
[![MCP Compatible](https://img.shields.io/badge/MCP-Compatible-7C3AED?style=flat-square)](https://modelcontextprotocol.io)
[![Status](https://img.shields.io/badge/Status-Active_Development-22c55e?style=flat-square)]()
[![License](https://img.shields.io/badge/License-Proprietary-ef4444?style=flat-square)]()
[![Memories](https://img.shields.io/badge/107K+-Memories_in_Production-f59e0b?style=flat-square)]()

[Blog Post](https://engram.hashnode.dev/memory-crisis-ai-agents) | [Discussions](https://github.com/orion1139/ebrain/discussions) | [Issues](https://github.com/orion1139/ebrain/issues)

---

</div>

## The Problem

Every AI coding agent has the same fatal flaw: **amnesia**.

- Context windows are finite. Your codebase is not.
- Auto-compaction silently destroys architectural decisions, bug history, and project context.
- Every new session starts from zero.
- Existing memory solutions (Mem0, Letta, Zep) are built for chatbot personas — not for the demands of real software engineering.

> If you've watched an AI agent forget a critical decision it made 20 minutes ago because the context window compacted, you understand why E-Brain exists.

## How E-Brain Works

E-Brain is a **server-side memory backend** designed specifically for AI coding agents. It ingests everything the agent sees and does, organizes it across multiple cognitive memory tiers, and serves precisely the right context back — fast enough to be invisible.

**Not a wrapper. Not a prompt hack. A dedicated memory architecture.**

```
                    ┌─────────────────────────────────┐
                    │         AI Coding Agent          │
                    │   (Claude, Cursor, Copilot...)   │
                    └──────────────┬──────────────────┘
                                   │ MCP / REST API
                    ┌──────────────▼──────────────────┐
                    │           E-Brain Server         │
                    │         (Rust, Axum, Tokio)      │
                    ├─────────────────────────────────┤
                    │                                  │
                    │  ┌──────────┐  ┌──────────────┐ │
                    │  │   Ring   │  │   Working    │ │
                    │  │  Buffer  │  │   Memory     │ │
                    │  └──────────┘  └──────────────┘ │
                    │  ┌──────────┐  ┌──────────────┐ │
                    │  │ Episodic │  │  Semantic    │ │
                    │  │  Memory  │  │   Memory     │ │
                    │  └──────────┘  └──────────────┘ │
                    │  ┌──────────┐  ┌──────────────┐ │
                    │  │Procedural│  │  Knowledge   │ │
                    │  │ Patterns │  │    Graph     │ │
                    │  └──────────┘  └──────────────┘ │
                    │                                  │
                    │  ┌────────────────────────────┐  │
                    │  │    Retrieval Pipeline       │  │
                    │  │  Noise → Category → MMR    │  │
                    │  │  → Temporal → Isolation     │  │
                    │  └────────────────────────────┘  │
                    └──────────────┬──────────────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                     │
     ┌────────▼───────┐  ┌────────▼───────┐  ┌─────────▼──────┐
     │  Vector Store  │  │   KV Store     │  │ Knowledge Graph│
     │   (Qdrant)     │  │  (Dragonfly)   │  │  (FalkorDB)    │
     └────────────────┘  └────────────────┘  └────────────────┘
```

## Key Capabilities

| Capability | Details |
|:-----------|:--------|
| **6 Memory Tiers** | Ring Buffer, Working, Episodic, Semantic, Procedural, Knowledge Graph |
| **Sub-100ms Recall** | p50: 72ms, p95: 88ms — fast enough to be invisible |
| **Noise-Aware Retrieval** | Filters debug noise, stale sessions, and irrelevant data before scoring |
| **MMR Diversity** | No more top-10 results that all say the same thing |
| **Category-Aware Scoring** | Architecture decisions rank higher than transient workflow notes |
| **Temporal Reasoning** | "What was I working on?" and "How does auth work?" get different strategies |
| **Project Isolation** | Multi-project, multi-tenant — memories never leak between contexts |
| **MCP Native** | 21 tools via Model Context Protocol, drop-in for Claude Code / Cursor |
| **85+ REST Endpoints** | Full API for any integration |
| **100K+ Scale** | 107K memories in production, tested without degradation |

## Memory Tiers

Different types of information have fundamentally different access patterns. E-Brain doesn't throw everything into a single vector database:

| Tier | Purpose | Access Pattern |
|:-----|:--------|:---------------|
| **Ring Buffer** | Current session context | Sub-millisecond, in-memory |
| **Working Memory** | Active observations and entities | Short-term, session-scoped |
| **Episodic Memory** | What happened, when, in what context | Auto-consolidates from working memory |
| **Semantic Memory** | Permanent facts and knowledge | Dual-indexed: BM25 + HNSW vectors |
| **Procedural Memory** | Learned patterns and workflows | Bayesian confidence, strengthens with use |
| **Knowledge Graph** | Entity relationships | Spreading activation, multi-hop reasoning |

## Retrieval Pipeline

Raw search results are garbage in every memory system. What matters is what happens *after* the search:

1. **Noise Filter** — Rejects debug data, stale artifacts, session noise before scoring
2. **Category-Aware Scoring** — Architecture and bug knowledge get priority over workflow notes
3. **Temporal Reasoning** — Detects recency vs. factual queries, adjusts ranking
4. **MMR Diversity** — Maximal Marginal Relevance eliminates near-duplicates
5. **Project Isolation** — Multi-project support without cross-contamination

## Benchmarks

### Search Latency

| System | p50 | p95 | Source |
|:-------|:---:|:---:|:-------|
| **E-Brain** | **0.072s** | **0.088s** | Measured |
| Mem0 | 0.148s | 0.200s | [Mem0 paper](https://arxiv.org/abs/2504.19413) |
| Mem0-Graph | 0.476s | 0.657s | Mem0 paper |
| Zep | 0.513s | 0.778s | Mem0 paper |
| LangMem | 17.99s | 59.82s | Mem0 paper |

### Retrieval Quality

| Metric | Score | What It Means |
|:-------|:-----:|:--------------|
| Noise Rejection | **100%** | Zero garbage in top-k results |
| Diversity (MMR) | **0.934** | No near-duplicate results |
| Temporal Awareness | **+0.182** | Correctly distinguishes recency vs. factual queries |

> **Honesty note:** We have not yet run the formal LoCoMo benchmark (industry standard). That's next. We publish only numbers we've actually measured.

## Built With

| Component | Technology | Why |
|:----------|:-----------|:----|
| Language | **Rust** | Zero-cost abstractions, no GC pauses, predictable latency |
| API | **Axum + Tokio** | Async, high-throughput HTTP |
| Vectors | **Qdrant** | HNSW dense embeddings |
| Full-Text | **Tantivy** | Rust-native BM25, sub-millisecond |
| KV Store | **Dragonfly** | Redis-compatible, multi-threaded |
| Graph | **FalkorDB** | Cypher queries, spreading activation |
| Protocol | **REST + MCP** | Direct agent integration |
| Embedding | **Configurable** | OpenAI, Ollama, or any compatible endpoint |

## Status

| Component | Status |
|:----------|:------:|
| Core memory tiers (6 tiers) | Production |
| REST API (85+ endpoints) | Production |
| MCP integration (21 tools) | Production |
| Retrieval pipeline (noise, diversity, category, temporal) | Production |
| Multi-tenant support | Production |
| Context monitoring (CCQ) | Production |
| Coding context system (3 MCP tools) | Production |
| Formal benchmarks (LoCoMo, LongMemEval) | Planned |
| Public API / hosted service | Planned |
| SDK (Python, TypeScript) | Planned |

## Why Not Open Source (Yet)?

E-Brain represents years of deep systems engineering work solving a problem the industry hasn't solved well. The existing open-source options treat memory as a feature to bolt on. E-Brain treats memory as the *core architecture*.

I'm building in public — sharing benchmarks, architecture decisions, and engineering insights openly. The implementation stays proprietary for now. If you're interested in early access, [open a discussion](https://github.com/orion1139/ebrain/discussions).

## Writing

- [The Memory Crisis in AI Agents — and What I'm Doing About It](https://engram.hashnode.dev/memory-crisis-ai-agents)

## Get Involved

- **Star this repo** to follow development
- **[Discussions](https://github.com/orion1139/ebrain/discussions)** — Ask questions, request features, share ideas
- **[Issues](https://github.com/orion1139/ebrain/issues)** — Report problems or suggest improvements

---

<div align="center">

*Built by [Engram](https://github.com/orion1139) — because AI agents deserve to remember.*

[![Twitter Follow](https://img.shields.io/badge/@engram__dev-Follow-000000?style=flat-square&logo=x&logoColor=white)](https://x.com/engram_dev)
[![Blog](https://img.shields.io/badge/Blog-Read-2962ff?style=flat-square&logo=hashnode&logoColor=white)](https://engram.hashnode.dev)

</div>
