# Engram vs. The Ecosystem — Deep Assessment

> A thorough comparison of engram against similar persistent memory tools for AI coding agents.

---

## The Problem Space

Every AI coding agent forgets everything when the session ends. A growing ecosystem of tools tries to solve this, each making different tradeoffs around storage, search, architecture, and agent compatibility.

This document compares engram against the most relevant alternatives as of early 2026, covering:

1. [**claude-mem**](#1-claude-mem-thedotmackclaude-mem) — The original, Claude-only pioneer (~28K stars)
2. [**Mem0 / OpenMemory MCP**](#2-mem0--openmemory-mcp-mem0aimem0) — The AI memory platform (~35K stars)
3. [**Anthropic Knowledge Graph Memory**](#3-anthropic-knowledge-graph-memory-modelcontextprotocolserversmemory) — The official reference server
4. [**MegaMemory**](#4-megamemory-0xk3vinmegamemory) — Knowledge graph with in-process embeddings
5. [**agent-recall**](#5-agent-recall-mnardtagent-recall) — Python SQLite knowledge graph

---

## TL;DR Comparison Table

| | **engram** | **claude-mem** | **Mem0 / OpenMemory** | **MCP Memory** | **MegaMemory** |
|---|---|---|---|---|---|
| **Language** | Go | TypeScript | Python | TypeScript | TypeScript |
| **Stars** | ~1K (growing) | ~28K | ~35K | ~80K (monorepo) | ~60 |
| **Single binary** | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Runtime deps** | None | Node.js + Bun + uv | Python + Docker | Node.js | Node.js |
| **Agent-agnostic** | ✅ | ❌ Claude only | ✅ | ✅ | ✅ |
| **Search type** | FTS5 full-text | ChromaDB vector | Qdrant/LanceDB vector | Graph traversal | In-process embeddings |
| **Requires GPU/API for search** | ❌ | ❌ | ✅ (embedding API) | ❌ | ✅ (embedding API) |
| **Storage** | SQLite | SQLite + ChromaDB | Vector DB + metadata | JSON file | SQLite |
| **Session lifecycle** | ✅ | ✅ | ❌ | ❌ | ❌ |
| **Privacy tags** | ✅ (2 layers) | ✅ | ❌ | ❌ | ❌ |
| **Git sync** | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Cloud sync** | ✅ (self-host) | ❌ | ✅ (SaaS + self-host) | ❌ | ❌ |
| **Web dashboard** | ✅ (cloud mode) | ✅ (localhost:37777) | ✅ | ❌ | ❌ |
| **TUI** | ✅ | ❌ | ❌ | ❌ | ❌ |
| **MCP tools** | 13 | 13 | 6 | 9 | ~8 |
| **Auto-capture** | ❌ (agent-driven) | ✅ (all tool calls) | ❌ | ❌ | ❌ |
| **License** | MIT | AGPL-3.0 | Apache-2.0 | MIT | MIT |
| **Windows native** | ✅ | ❌ (WSL2 needed) | ✅ | ✅ | ✅ |
| **Homebrew install** | ✅ | ❌ | ❌ | ❌ | ❌ |

---

## 1. claude-mem ([thedotmack/claude-mem](https://github.com/thedotmack/claude-mem))

**~28K stars · TypeScript · AGPL-3.0**

The original project that inspired engram. Claude-mem captures *everything* Claude does during a session (file reads, edits, shell commands), compresses it with an AI pipeline using Claude's agent-sdk, and injects relevant context into future sessions.

### What it does well

- **Passive capture** — zero effort for the agent; every action is recorded automatically
- **AI compression pipeline** — high-quality semantic summaries via a dedicated compression pass
- **Web viewer** — a real-time session timeline visible at `localhost:37777`
- **Smart explore tools** — AST-powered code navigation for token-efficient context delivery
- **Mature ecosystem** — 28K stars, extensive documentation, large community

### Where it falls short

| Concern | Details |
|---|---|
| **Claude only** | Hard-locked to Claude Code's plugin system. Won't work with OpenCode, Gemini CLI, Codex, VS Code Copilot, Cursor, or Windsurf. |
| **Heavy dependencies** | Requires Node.js 18+, Bun, uv, Python, AND ChromaDB as a separate vector database process. On Windows it requires WSL2. |
| **Multiple processes** | Worker service on port 37777 + ChromaDB process. Not a single binary. |
| **AGPL-3.0** | Copyleft license creates complications for proprietary workflows or embedding in commercial tooling. |
| **Raw capture noise** | Storing every `edit: {file: "foo.go"}` and `bash: {command: "go build"}` pollutes FTS search until the compression pass runs. |
| **Separate compression API cost** | Every session requires additional Claude API calls for the compression pipeline, adding latency and cost beyond what the agent already spends. |
| **ChromaDB overhead** | A vector database process for what is fundamentally a coding session log is significant infrastructure for modest recall improvements over FTS5. |

### Key design difference

claude-mem's philosophy: *capture everything, compress with AI later.*  
Engram's philosophy: *let the agent decide what's worth keeping.*

The agent already has the LLM, the context, and understands what just happened. A separate compression pipeline is redundant — and expensive.

---

## 2. Mem0 / OpenMemory MCP ([mem0ai/mem0](https://github.com/mem0ai/mem0))

**~35K stars · Python · Apache-2.0**

Mem0 is a general-purpose AI memory platform. OpenMemory MCP is its local-first MCP server for coding agents. It uses vector databases (Qdrant or LanceDB) for semantic similarity search and supports multi-level memory (user, session, agent, project).

### What it does well

- **Semantic search** — vector similarity finds relevant memories even with different wording
- **Memory types** — organizes memories with rich metadata (topics, timestamps, emotional context)
- **Multi-tool support** — works across Cursor, VS Code, Claude Desktop, JetBrains, and more
- **Web dashboard** — built-in UI for reviewing and managing stored memories
- **Large community** — 35K stars, active development, well-funded company behind it
- **Benchmarks** — claims +26% accuracy over OpenAI's memory system on standard benchmarks

### Where it falls short

| Concern | Details |
|---|---|
| **Python + Docker** | Requires `pip install mem0ai` or a Docker container. No single binary, no Go runtime. |
| **Embedding API dependency** | Semantic search requires a running embedding model (local or API). Every search = an embedding API call or local inference. |
| **No session lifecycle** | No concept of coding sessions with start/end tracking, session summaries, or session-scoped context injection. |
| **No Git sync** | No mechanism to share memories through a git repository across team members. |
| **Cloud = SaaS** | The hosted Mem0 cloud is a third-party SaaS product with pricing tiers. Self-hosting is possible but complex (Qdrant + application server). |
| **General-purpose, not coding-focused** | Designed for all AI agents, not specifically optimized for the coding agent workflow (bugfixes, architecture decisions, code patterns). |
| **No privacy tags** | No built-in mechanism to exclude sensitive content from persistent storage. |

### When Mem0/OpenMemory is the better choice

- You need **semantic similarity search** (e.g., "find memories about authentication even if they weren't tagged that way")
- You're building a **general AI assistant** (not specifically a coding agent)
- You want **existing infrastructure** backed by a funded company
- You need memories to **span different tools** (IDE + chatbot + CLI)

---

## 3. Anthropic Knowledge Graph Memory ([modelcontextprotocol/servers/memory](https://github.com/modelcontextprotocol/servers/tree/main/src/memory))

**~80K stars (whole servers monorepo) · TypeScript · MIT**

The official Anthropic reference implementation. Provides a knowledge graph-based memory using a local JSON file. Entities, relations, and observations modeled as nodes and edges.

### What it does well

- **Official/reference status** — maintained by Anthropic, good docs, widely cited
- **Simple knowledge graph** — entities and relations model complex interconnections well
- **Zero cost** — stores everything in a flat JSON file, no database process required
- **MIT license** — maximum permissive usage
- **Broad agent support** — works with any MCP client

### Where it falls short

| Concern | Details |
|---|---|
| **No sessions** | No concept of coding sessions, session summaries, or session lifecycle. Just raw entities and relations. |
| **JSON file storage** | Large knowledge graphs in JSON files are slow to query and problematic to update atomically. No FTS or indexing. |
| **No full-text search** | Can only query by entity name or traverse relations. No keyword search across all stored observations. |
| **Node.js runtime** | Requires Node.js to run (`npx -y @modelcontextprotocol/server-memory`). |
| **No team sync** | No mechanism to share the knowledge graph across machines or team members. |
| **Manual graph management** | The agent must explicitly create entities and relations. No session summary or context injection workflows. |
| **No privacy** | No mechanism to exclude sensitive data. |

### When this is the better choice

- You want the **simplest possible** MCP memory (just a JSON file)
- You need a **reference implementation** to understand MCP memory patterns
- You're building a **knowledge graph** specifically (entities + relations, not session summaries)

---

## 4. MegaMemory ([0xK3vin/MegaMemory](https://github.com/0xK3vin/MegaMemory))

**~60 stars · TypeScript · MIT**

A TypeScript MCP server that combines a knowledge graph with in-process embeddings for semantic search. Designed specifically for coding agents with support for Claude Code and OpenCode.

### What it does well

- **In-process embeddings** — semantic search without a separate vector database process
- **Knowledge graph** — entities and relations for structured memory
- **Coding-agent focus** — designed with similar use cases to engram
- **SQLite storage** — familiar, reliable storage backend

### Where it falls short

| Concern | Details |
|---|---|
| **Node.js runtime** | Requires Node.js. No single binary distribution. |
| **Embedding dependency** | Semantic search still requires an embedding API call or local model at query time. |
| **No session lifecycle** | No start/end session tracking, no session summaries. |
| **No Git sync** | No mechanism to share memories through git. |
| **No cloud sync** | No multi-device synchronization. |
| **Early stage** | ~60 stars, relatively new project with smaller community and fewer battle-tested deployments. |
| **No TUI** | Terminal interface requires external tools. |

---

## 5. agent-recall ([mnardit/agent-recall](https://github.com/mnardit/agent-recall))

**~6 stars · Python · MIT**

Python-based MCP server with a SQLite knowledge graph. Designed with scope hierarchy (projects, clients, roles) and LLM-based context summarization at session start.

### What it does well

- **Scope hierarchy** — nested project/client/role context is a thoughtful design
- **Bitemporal history** — full history tracking (not just current state)
- **Structured knowledge** — graph-based storage for complex relationships
- **LLM summarization** — context is summarized at session start for token efficiency

### Where it falls short

| Concern | Details |
|---|---|
| **Python runtime** | No single binary. Requires Python installation and `pip install`. |
| **LLM required for context** | Context summarization requires a live LLM API call at session start, adding latency. |
| **Very early stage** | ~6 stars, limited documentation and community. |
| **No Git sync** | No mechanism to share memories through git. |
| **No cloud sync** | No multi-device synchronization. |
| **No TUI** | No terminal UI for browsing memories. |

---

## Deep Comparison: Search Strategy

This is the most consequential architectural decision — how stored memories are found.

| Approach | Used by | Pros | Cons |
|---|---|---|---|
| **SQLite FTS5** (BM25 ranking) | engram | Zero deps, fast, works offline, handles code tokens well | No semantic similarity, requires keyword overlap |
| **Vector embeddings** | Mem0, MegaMemory | Finds semantically similar content even with different wording | Requires embedding API or local model, slower, costs money |
| **ChromaDB** | claude-mem | Mature vector DB, full semantic search | Separate process, heavyweight for session memory |
| **JSON graph traversal** | MCP Memory | Simple, no deps | Slow at scale, no keyword search |
| **Postgres tsvector** | engram (cloud) | Weighted full-text search, production-ready | Requires Postgres for cloud mode |

**Engram's bet**: For coding session memory — bugfixes, architecture decisions, patterns — keywords matter. A search for "JWT auth middleware" will find the memory titled "Fixed JWT auth middleware" just as well as a vector search would, at zero cost and zero latency. Semantic search shines when queries are vague natural language ("how did we handle that auth thing?") — and the agent, with its understanding of context, can reformulate queries into keywords before calling `mem_search`.

---

## Deep Comparison: Installation Complexity

```
engram:
  brew install gentleman-programming/tap/engram
  → done. One binary, no other processes.

claude-mem:
  /plugin marketplace add thedotmack/claude-mem
  /plugin install claude-mem
  + needs: Node.js 18+ (manual), Bun (auto-installed), uv (auto-installed)
  + Windows: needs WSL2 first
  → 2-3 background processes running

Mem0 / OpenMemory:
  pip install mem0ai
  + needs: Python 3.9+, Docker (for Qdrant), embedding API key
  → Qdrant vector DB process running

MCP Memory (Anthropic):
  npx -y @modelcontextprotocol/server-memory
  + needs: Node.js
  → simple, but no persistence guarantees

MegaMemory:
  npm install (from source)
  + needs: Node.js, embedding API key
  → Node.js process
```

---

## Deep Comparison: Agent Support Matrix

| Agent | engram | claude-mem | Mem0 MCP | MCP Memory | MegaMemory |
|---|---|---|---|---|---|
| Claude Code | ✅ (plugin) | ✅ (native) | ✅ | ✅ | ✅ |
| OpenCode | ✅ (plugin) | ❌ | ✅ | ✅ | ✅ |
| Gemini CLI | ✅ | ❌ | ✅ | ✅ | ✅ |
| Codex | ✅ | ❌ | ✅ | ✅ | ✅ |
| VS Code Copilot | ✅ | ❌ | ✅ | ✅ | ✅ |
| Cursor | ✅ | ❌ | ✅ | ✅ | ✅ |
| Windsurf | ✅ | ❌ | ✅ | ✅ | ✅ |
| Antigravity | ✅ | ❌ | ✅ | ✅ | ✅ |
| Any MCP client | ✅ | ❌ | ✅ | ✅ | ✅ |

---

## When to Use Each Tool

### Choose **engram** when:
- You want **one binary with zero runtime dependencies** — no Node.js, no Python, no Docker
- You switch between **multiple agents** (not locked to Claude Code)
- You need **Git sync** to share memories with your team through version control
- You want a **self-hosted cloud sync** option with a Postgres backend
- You prefer **agent-curated summaries** over raw tool-call capture (cleaner data)
- You're on **Windows natively** (no WSL2 required)
- You want a **TUI** for browsing memories from the terminal
- You need the **MIT license** (vs AGPL-3.0 for claude-mem)
- You want **privacy tags** enforced at both plugin and storage layers

### Choose **claude-mem** when:
- You **only use Claude Code** and don't plan to switch
- You want **zero-effort passive capture** (every action recorded automatically without telling the agent to save)
- You prefer a **web viewer** over a terminal TUI
- You value the **large community** (28K stars, extensive docs, active support)
- You want **AST-powered code exploration** tools for deep codebase navigation

### Choose **Mem0 / OpenMemory** when:
- You need **semantic/vector search** for fuzzy recall ("find memories about authentication")
- You want memories that **span multiple tools** (IDE + chatbot + other AI tools)
- You're building a **general AI assistant** (not a coding-specific agent)
- You're comfortable with **Python + Docker** in your setup
- You want a **SaaS option** with managed infrastructure

### Choose **Anthropic MCP Memory** when:
- You want the **simplest possible** MCP memory implementation
- You need a **knowledge graph** (entities + relations) not session-based memory
- You're building a prototype or **learning about MCP**
- You want **official Anthropic backing** and reference implementation status

---

## Unique Engram Capabilities

These features exist in engram but have no equivalent in the other tools compared here:

| Feature | Description |
|---|---|
| **Git sync** | Compressed gzipped JSONL chunks committed to your repo — share memories across machines with `git push/pull`, no extra server |
| **Cloud sync with self-hosted Postgres** | Full mutation-based auto-sync with JWT auth, project-scoped enrollment, and admin dashboard — host it yourself |
| **Session lifecycle tracking** | Full session start/end, session summaries, context injection, compaction recovery — not just raw observation storage |
| **Progressive disclosure (3-layer)** | `mem_search` → `mem_timeline` → `mem_get_observation` — token-efficient retrieval without dumping everything |
| **`mem_suggest_topic_key`** | Suggests stable canonical keys for evolving topics so `mem_save` can upsert instead of creating duplicates |
| **Homebrew install** | `brew install gentleman-programming/tap/engram` — native package manager install on macOS/Linux |
| **Windows native binary** | True Windows executable — no WSL2, no emulation layer |
| **Cloud dashboard (templ + htmx)** | Server-rendered web UI embedded in the binary — browse observations, sessions, contributors, admin panel |
| **`<private>` tags at 2 layers** | Stripped in the agent plugin before HTTP transmission AND again in the Go store layer before any DB write |

---

## Summary

The persistent memory space for AI coding agents is young and fast-moving. Every tool here solves a real problem, and the "best" choice depends heavily on your workflow.

**Engram's positioning**: the *infrastructure-light* choice for developers who want powerful session memory without managing Node.js workers, Python environments, or vector databases — across any agent, on any platform.

The tradeoff is intentional: FTS5 over vector search (simpler, faster, offline), agent-driven saves over passive capture (cleaner data, no noise), and a single binary over a microservice architecture (operational simplicity).

As the ecosystem matures, watch for:
- Vector search becoming viable at zero marginal cost (local embeddings getting faster)
- More agents adopting MCP natively (growing agent-agnostic ecosystem)
- Semantic search potentially complementing FTS5 (not replacing it) for vague natural-language queries

---

*Last updated: March 2026. Star counts are approximate and change rapidly in this space.*
