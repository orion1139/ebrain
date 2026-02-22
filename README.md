<div align="center">

# E-Brain

### Persistent External Brain for AI Coding Agents

*The memory layer that makes AI agents remember everything — across sessions, across compactions, across projects.*

[![Built with Rust](https://img.shields.io/badge/Built_with-Rust-dea584?style=flat-square&logo=rust)](https://www.rust-lang.org/)
[![License](https://img.shields.io/badge/License-Proprietary-red?style=flat-square)]()
[![Status](https://img.shields.io/badge/Status-Active_Development-green?style=flat-square)]()
[![MCP](https://img.shields.io/badge/MCP-Compatible-purple?style=flat-square)]()

---

</div>

## The Problem

Every AI coding agent has the same fatal flaw: **amnesia**.

- Context windows are finite (128K-200K tokens). Your codebase is not.
- Auto-compaction silently destroys architectural decisions, bug history, and project context.
- Every new session starts from zero. The agent re-discovers what it already knew.
- Existing memory solutions (Mem0, Letta, Zep) are built for chatbot personas — not for the demands of real software engineering.

If you've ever watched an AI agent forget a critical decision it made 20 minutes ago because the context window compacted, you understand why E-Brain exists.

## What E-Brain Does

E-Brain is a **server-side memory backend** that sits between your AI agent and permanent storage. It ingests everything the agent sees and does, organizes it across multiple memory tiers, and serves precisely the right context back — fast enough to be invisible.

**Not a wrapper. Not a prompt hack. A dedicated memory architecture.**

### Key Capabilities

| Capability | What It Means |
|-----------|---------------|
| **Multi-tier memory** | Different memory types for different needs — not everything goes in one vector store |
| **Sub-100ms recall** | Fast enough that the agent never waits for its memory |
| **Noise-aware retrieval** | Filters debug noise, stale session data, and irrelevant results before they reach the agent |
| **Diversity enforcement** | No more top-10 results that all say the same thing in different words |
| **Category-aware scoring** | Architectural decisions rank higher than transient workflow notes |
| **Temporal reasoning** | "What was I working on?" and "How does auth work?" get different retrieval strategies |
| **Project isolation** | Multi-project, multi-tenant — memories don't leak between contexts |
| **MCP native** | Model Context Protocol integration out of the box |
| **100K+ scale** | Tested and tuned for massive memory pools without degradation |

## Architecture (High-Level)

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
                    │  │  Retrieval Pipeline        │  │
                    │  │  Noise Filter → Category   │  │
                    │  │  Scoring → MMR Diversity   │  │
                    │  │  → Temporal Ranking        │  │
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

### Memory Tiers

E-Brain doesn't throw everything into a single vector database. Different types of information have fundamentally different access patterns:

- **Ring Buffer** — Ultra-fast in-memory cache for the current session's most recent context. Sub-millisecond access.
- **Working Memory** — Short-term observations, entities, and relationships detected in the current work session.
- **Episodic Memory** — Longer-term records of what happened, when, and in what context. Consolidates from working memory.
- **Semantic Memory** — Permanent facts and knowledge. Dual-indexed: full-text search (Tantivy/BM25) + dense vectors (Qdrant/HNSW).
- **Procedural Memory** — Learned patterns and workflows. Bayesian confidence scoring that strengthens with repeated use.
- **Knowledge Graph** — Entity relationships with spreading activation for multi-hop reasoning.

### Retrieval Pipeline

Raw search results are garbage in every memory system. What matters is what happens *after* the search:

1. **Noise Filter** — Rejects debug data, stale session artifacts, and irrelevant content before scoring
2. **Category-Aware Scoring** — Architectural decisions and bug knowledge get priority over transient workflow notes
3. **Temporal Reasoning** — Detects whether you're asking about recent activity or permanent facts, adjusts ranking accordingly
4. **MMR Diversity** — Maximal Marginal Relevance ensures the top results are actually *different* from each other
5. **Project Isolation** — Multi-project support without cross-contamination

## Benchmarks

Measured against published numbers from competing memory systems:

### Search Latency

| System | p50 | p95 | Source |
|--------|-----|-----|--------|
| **E-Brain** | **0.072s** | **0.088s** | Measured (this project) |
| Mem0 | 0.148s | 0.200s | [Mem0 paper](https://arxiv.org/abs/2504.19413) |
| Mem0-Graph | 0.476s | 0.657s | Mem0 paper |
| Zep | 0.513s | 0.778s | Mem0 paper |
| LangMem | 17.99s | 59.82s | Mem0 paper |

### Retrieval Quality

| Metric | Score | Assessment |
|--------|-------|-----------|
| Noise Rejection | 100% | Zero garbage in top-k results |
| Diversity (MMR) | 0.934 | No near-duplicate results |
| Temporal Awareness | +0.182 | Correctly distinguishes recency vs factual queries |

> **Note on LoCoMo:** We have not yet run the formal LoCoMo benchmark (the industry standard for memory system comparison). This is planned. We believe in publishing only numbers we've actually measured, not estimates. See the [benchmarks](docs/benchmarks.md) directory for full methodology and reproducible scripts.

## How It's Built

- **Language**: Rust (zero-cost abstractions, no GC pauses, predictable latency)
- **API Framework**: Axum + Tokio (async, high-throughput HTTP)
- **Vector Search**: Qdrant (HNSW, dense embeddings)
- **Full-Text Search**: Tantivy (Rust-native BM25, sub-millisecond)
- **KV Store**: Dragonfly (Redis-compatible, multi-threaded)
- **Graph**: FalkorDB (Cypher queries, spreading activation)
- **Protocol**: REST API + MCP (Model Context Protocol) for direct agent integration
- **Embedding**: Configurable — supports OpenAI, local Ollama, or any OpenAI-compatible endpoint

## Status

E-Brain is in **active development**. It is not yet open source.

| Component | Status |
|-----------|--------|
| Core memory tiers (6 tiers) | Production |
| REST API (85+ endpoints) | Production |
| MCP integration (21 tools) | Production |
| Retrieval pipeline (noise, diversity, category, temporal) | Production |
| Multi-tenant support | Production |
| Context monitoring (CCQ) | Production |
| Formal benchmarks (LoCoMo, LongMemEval) | Planned |
| Public API / hosted service | Planned |
| SDK (Python, TypeScript) | Planned |

## Why Not Open Source (Yet)?

E-Brain represents years of deep engineering work solving a problem that the industry hasn't solved well. The existing open-source options treat memory as a feature to bolt on. E-Brain treats memory as the *core architecture*.

I'm building in public — sharing benchmarks, architecture decisions, and engineering insights — while keeping the implementation proprietary for now. If you're interested in early access, open a discussion.

## Writing

I write about AI memory engineering, the problems I'm solving, and honest benchmarks:

- *Coming soon: "The Memory Crisis in AI Agents — and What I'm Doing About It"*

## Get Involved

- **Star this repo** to follow development
- **[Discussions](https://github.com/orion1139/ebrain/discussions)** — Ask questions, request features, share ideas
- **[Issues](https://github.com/orion1139/ebrain/issues)** — Report problems or suggest improvements to docs/benchmarks

---

<div align="center">

*Built by [Engram](https://github.com/orion1139) — because AI agents deserve to remember.*

</div>
