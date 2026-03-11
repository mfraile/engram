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

Jump to: [Deep comparisons](#deep-comparison-search-strategy) · [VS Code + Copilot + Governance assessment](#vs-code--github-copilot-deep-assessment-for-governance-constrained-repositories) · [When to choose each tool](#when-to-use-each-tool)

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

## VS Code + GitHub Copilot: Deep Assessment for Governance-Constrained Repositories

> This section specifically analyses which persistent memory tool best fits a **VS Code + GitHub Copilot** workflow operating under enterprise governance and security constraints — the kind enforced in repositories like **panaegis-sentinel**.

### About panaegis-sentinel

> **Note**: `panaegis-sentinel` is a private repository that is not publicly accessible at the time of writing. The analysis below is based on the repository name's implied purpose (security/governance tooling), standard enterprise governance patterns for such projects, and the author's known context. If the actual repository has different constraints, the scoring and recommendations should be adjusted accordingly. This section is therefore best read as *an illustrative governance scenario for a security-focused repository using VS Code + Copilot* — one that panaegis-sentinel appears to represent.

`panaegis-sentinel` is a private security/governance repository by `mfraile`. The name itself signals intent: *panaegis* (pan + aegis — comprehensive protection) combined with *sentinel* (guard, monitor, observer) describes a project that monitors, audits, and enforces governance rules over an environment. Working in such a repository implies specific constraints on what tools an AI coding agent may use:

- **Sensitive code and data must never leave the machine** (security policies, threat models, vulnerability research, API tokens, audit findings)
- **License compliance is mandatory** — copyleft licenses like AGPL-3.0 can create complications for governance/security tooling that may be embedded in or distributed with other software
- **No unapproved external cloud services** — data residency and sovereignty rules prevent third-party SaaS memory stores
- **Enterprise Windows compatibility** — most security/compliance environments run corporate-managed Windows machines without WSL2
- **Minimal attack surface** — every additional process or runtime is a potential vulnerability vector
- **Auditability of AI behaviour** — what the agent saves and retrieves must be inspectable

---

### How VS Code Copilot MCP Works (Technical Foundation)

VS Code natively supports MCP servers since **v1.102 (GA)**. Memory tools are wired in via `.vscode/mcp.json`:

```json
{
  "servers": {
    "engram": {
      "command": "engram",
      "args": ["mcp"]
    }
  }
}
```

**Enterprise-specific constraints from Microsoft's own docs:**

| Constraint | Detail |
|---|---|
| **MCP disabled by default** | Org admins must explicitly enable "MCP servers in Copilot" policy for teams |
| **GitHub Enterprise Server** | Only **local** MCP servers are allowed — remote HTTP MCP is blocked |
| **Group Policy precedence** | If either Copilot config or group policy disables MCP, it stays off |
| **Workspace trust** | VS Code must trust the workspace before agent mode activates |
| **MCP registry allowlist** | Enterprises can enforce a registry of approved MCP servers — only listed servers are discoverable |

This matters for tool selection: **any tool requiring a running network service, Python daemon, or Docker container will likely fail enterprise IT review**. Only tools that run as a simple local `stdio` process pass cleanly.

---

### Behavioural Guidance: copilot-instructions.md + Memory Protocol

For VS Code + Copilot to use memory intelligently, two files work together:

1. **`.github/copilot-instructions.md`** — repository-level rules for Copilot behaviour (enforced by GitHub's enterprise policy system)
2. **`~/.config/Code/User/prompts/engram-memory.instructions.md`** — user-level Memory Protocol that teaches Copilot *when* to save and search

In a governance repository like panaegis-sentinel, the `copilot-instructions.md` would typically enforce:
- "Never include secrets, tokens, or credentials in any memory save"
- "Tag all security findings with `type: discovery`"
- "Always use `<private>` tags around CVE identifiers, API endpoints, and infrastructure details"
- "Call `mem_search` before implementing any security control to check for prior art in this repo"

**This integration only works reliably with engram**, because:
- The Memory Protocol maps directly to engram's 13 MCP tools
- The `<private>` tag system is enforced at two layers (plugin + store), not just in instructions
- No other tool in this comparison has an equivalent two-layer privacy guarantee

---

### Tool-by-Tool Scoring for VS Code + Governance Context

Evaluation criteria — each scored 0–3 (0 = fails outright, 3 = fully meets requirement):

| Criterion | **engram** | **claude-mem** | **Mem0/OpenMemory** | **MCP Memory** | **MegaMemory** |
|---|---|---|---|---|---|
| **Works as local stdio MCP** | 3 | 0 | 2 | 3 | 2 |
| **Zero runtime dependencies** | 3 | 0 | 0 | 1 | 1 |
| **MIT/permissive license** | 3 | 0 (AGPL) | 2 (Apache-2) | 3 | 3 |
| **Native Windows (no WSL2)** | 3 | 0 | 2 | 2 | 2 |
| **All data stays local by default** | 3 | 2 | 2 | 3 | 3 |
| **No embedding API required** | 3 | 3 | 0 | 3 | 0 |
| **Privacy/redaction feature** | 3 | 2 | 0 | 0 | 0 |
| **Copilot-instructions compatible** | 3 | 0 | 1 | 1 | 1 |
| **Committable to repo (Git sync)** | 3 | 0 | 0 | 0 | 0 |
| **Structured security-focused saves** | 3 | 1 | 1 | 1 | 1 |
| **Enterprise MCP policy compliant** | 3 | 0 | 1 | 3 | 2 |
| **Inspectable/auditable storage** | 3 | 2 | 2 | 2 | 2 |
| **TOTAL** | **36/36** | **10/36** | **11/36** | **22/36** | **17/36** |

> **Scoring notes**: claude-mem scores 0 on "Works as local stdio MCP" because it requires the Claude Code plugin system and will not register as an MCP server in VS Code Copilot's agent mode. Mem0/OpenMemory scores 0 on "No embedding API required" because semantic search always requires an embedding call. Both MCP Memory and engram score 3 on "All data stays local by default".

---

### Why claude-mem Is Not Viable Here

claude-mem is **incompatible with VS Code + Copilot** at a fundamental level:

- It is a Claude Code **plugin**, not a standalone MCP server. VS Code Copilot does not use Claude Code plugins.
- It requires Node.js 18+, Bun, and uv — three runtimes that enterprise IT teams typically need to approve separately
- On Windows (common in corporate environments), it **requires WSL2** — which is blocked or requires admin escalation on most corporate laptops
- Its AGPL-3.0 license means that any software incorporating or distributing it inherits the copyleft requirement — a legal problem for a security tool that may be packaged or deployed internally
- It has no `.vscode/mcp.json` integration path

**Verdict for panaegis-sentinel**: ❌ Incompatible.

---

### Why Mem0/OpenMemory Falls Short Here

Mem0 OpenMemory is conceptually aligned (local-first, MCP-compatible, MIT-adjacent Apache-2.0), but:

- **Requires Python + Docker** — Docker Desktop requires a paid license in enterprises with >250 employees or >$10M revenue (Docker Business), and Python must be a separately approved runtime
- **Embedding API dependency** — semantic search calls an external embedding service. In an air-gapped or internet-restricted environment (common for security tooling), this breaks completely
- **No structured security memory types** — Mem0's memory schema is general-purpose and doesn't have the `decision`, `discovery`, `bugfix`, `config` types that map to security work
- **No `<private>` tag system** — for a security repository, the absence of any built-in sensitive-content redaction is a significant gap
- **No Git sync** — findings and governance decisions cannot be versioned alongside the code they describe

**Verdict for panaegis-sentinel**: ⚠️ Possible but has meaningful gaps; requires Docker approval and embedding API access.

---

### Why Anthropic MCP Memory Is Not Enough

The official Anthropic memory server (MCP Memory) is the cleanest option after engram in the governance scoring, but it is underpowered for real security/governance work:

- **JSON file storage** — a single flat JSON file with no indexing or full-text search is impractical once hundreds of security observations accumulate
- **No structured types** — can't distinguish a threat model decision from a CVE patch from an architecture choice
- **No session lifecycle** — security work is session-oriented: "Thursday's penetration test session" should be retrievable as a unit
- **No Git sync** — security findings need to live in version control with the code they protect
- **No privacy redaction** — no mechanism to strip sensitive data before storage
- **Manual graph management** — the agent must explicitly create entities and relations, adding friction to already-complex security workflows

**Verdict for panaegis-sentinel**: ⚠️ Works for basic memory, but insufficient for structured security knowledge management.

---

### Engram's Specific Advantages for This Use Case

For a security/governance repository like panaegis-sentinel, engram uniquely provides:

**1. Two-layer `<private>` tag enforcement**
```
Discovered API key: <private>sk-prod-abc123</private> in legacy config
→ Stored as: "Discovered API key: [REDACTED] in legacy config"
```
Sensitive data is stripped *before* the HTTP call to engram (plugin layer) AND again inside `AddObservation()` in Go (store layer). No other tool in this comparison strips at both layers.

**2. Security-native memory types**
Engram's built-in types (`decision`, `architecture`, `bugfix`, `pattern`, `config`, `discovery`) map directly to security work:
- `discovery` — new vulnerability, misconfiguration, threat vector
- `decision` — architectural security decision (e.g., "chose JWT over sessions for stateless auth")
- `config` — environment change, security policy update
- `pattern` — reusable security pattern, encoding convention

**3. Git sync for audit trails**
```bash
engram sync
git add .engram/ && git commit -m "sync: week 11 threat model sessions"
```
Security findings, architectural decisions, and vulnerability discoveries are committed alongside the code they protect. This creates a tamper-evident, version-controlled audit trail that satisfies most governance requirements.

**4. VS Code Copilot integration path**

Add to `.vscode/mcp.json` (committable to panaegis-sentinel):
```json
{
  "servers": {
    "engram": {
      "command": "engram",
      "args": ["mcp"]
    }
  }
}
```

Add Memory Protocol to user prompts file (`~/.config/Code/User/prompts/engram-memory.instructions.md`) or merge into `.github/copilot-instructions.md`:
```markdown
## AI Memory Protocol (engram)
- After every security finding: call mem_save with type=discovery
- After every architectural decision: call mem_save with type=decision
- Wrap all sensitive values in <private> tags before calling mem_save
- Before implementing a security control: call mem_search to check for prior art
- At session end: call mem_session_summary with what was found and what remains
```

**5. Zero infrastructure overhead**
A security repository is not the place to run a Docker container, a Python daemon, a vector database, or a Node.js worker. Engram is a single Go binary — one file, one process, no ports beyond the optional HTTP API on localhost.

**6. Enterprise MCP policy compatibility**
Engram's `stdio` transport runs as a child process of VS Code — exactly the model that enterprise MCP policies are designed for. It never opens a network socket by default. No firewall rules, no proxy configuration, no IT ticket needed.

---

### Recommended Setup for panaegis-sentinel + VS Code Copilot

```bash
# 1. Install (macOS/Linux)
brew install gentleman-programming/tap/engram

# 1. Install (Windows — native, no WSL2)
# Download engram_<version>_windows_amd64.zip from GitHub Releases
# Extract engram.exe to a folder in PATH

# 2. Register with VS Code (one-time, sets up .vscode/mcp.json)
code --add-mcp '{"name":"engram","command":"engram","args":["mcp"]}'

# 3. Verify
engram stats  # should show empty db on first run
```

Then commit `.vscode/mcp.json` to panaegis-sentinel so every team member gets the same memory server configuration automatically.

Add to `.github/copilot-instructions.md`:
```markdown
## Persistent Memory (engram)
Engram MCP is active in this workspace. Use it:
- ALWAYS wrap secrets, CVEs, tokens, and IPs in <private> tags before mem_save
- Use type=discovery for vulnerabilities and misconfigs
- Use type=decision for security architecture choices
- Call mem_search before re-implementing any security pattern
- End every session with mem_session_summary
```

---

### Final Verdict for VS Code + Copilot + panaegis-sentinel

| Tool | Verdict | Reason |
|---|---|---|
| **engram** | ✅ **Recommended** | Single binary, MIT, local-first, privacy tags at 2 layers, Git sync, VS Code MCP native, no external dependencies |
| **Anthropic MCP Memory** | ⚠️ Acceptable fallback | MIT, local, zero deps — but lacks FTS, session lifecycle, typed observations, and privacy redaction |
| **Mem0 / OpenMemory** | ⚠️ Use with caution | Requires Python + Docker; embedding API calls may violate data residency rules; no privacy tags |
| **MegaMemory** | ⚠️ Limited | Node.js + embedding API dependency; no Git sync; no privacy redaction |
| **claude-mem** | ❌ Not viable | Claude-only plugin system; AGPL-3.0 license; requires WSL2 on Windows; incompatible with VS Code Copilot MCP |

**Engram is the only tool in this ecosystem that satisfies all governance constraints simultaneously**: permissive license, local-only data by default, zero external runtime dependencies, native Windows binary, two-layer privacy redaction, Git-committable memory, and VS Code Copilot MCP integration.

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
