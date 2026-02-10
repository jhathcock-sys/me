---
title: "Semantic Memory: Claude Memory System 3.0"
date: 2026-02-09
draft: false
---

# 🧠 Semantic Memory: Claude Memory System 3.0

**Part 3 of the Claude Code Memory System series**

This article documents the evolution from file-based context to semantic vector search. If you haven't read the previous parts, start with:
- [Part 1: Claude Code Memory System](../claude-memory/) - XML-based context and directives
- [Part 2: Memory System 2.0](../claude-memory/#evolution-memory-system-20-february-2026) - Symlink architecture and sync automation

---

## 🔄 Evolution: Memory System 3.0 (February 2026)

### The Limits of Files

Memory Systems 1.0 and 2.0 solved real problems. XML-based CLAUDE.md files gave Claude structured context, and the symlink architecture kept everything in sync across projects. But as the homelab grew, cracks started to show.

The fundamental issue was **retrieval**. Every conversation started with Claude loading whatever CLAUDE.md and MEMORY.md files were in scope, parsing the XML, and working from there. This worked fine when the homelab was a handful of services. But with dozens of containers, multiple network segments, architecture decisions scattered across months of work, and runbooks for everything from Wazuh tuning to Jellyfin transcoding -- the file-based approach started hitting walls:

- **Context window limits are real.** You can only stuff so much XML into a conversation before you are burning tokens on context that may not be relevant to the current question.
- **Keyword matching is brittle.** If I documented a service under "SIEM stack" but later asked about "security monitoring," the XML structure had no way to bridge that semantic gap.
- **Manual maintenance does not scale.** Every new service meant updating XML files by hand. Every architecture decision needed to be slotted into the right node. The overhead was starting to feel like the problem I was trying to solve.

The question became: what if Claude could *search* for what it needs, the same way a human would search documentation, rather than having everything pre-loaded and hoping the right context is there?

### The Solution: Vector Database + Semantic Search

The answer turned out to be a technology I had been reading about in the context of RAG (Retrieval-Augmented Generation) applications: **vector databases**. The core idea is straightforward, even if the math underneath is not.

Instead of storing documents as plain text to be keyword-matched, a vector database converts each document into an **embedding** -- a high-dimensional numerical representation that captures the *meaning* of the text, not just the words. When you query the database, your question gets converted into the same vector space, and the database finds documents that are semantically close to your query.

In practice, this means asking "What IP is my SIEM on?" can find a document that says "Wazuh is deployed at 192.168.1.7" -- even though the query never mentions "Wazuh" or the specific IP. The vectors for "SIEM" and "Wazuh" are close in embedding space because they are semantically related.

The specific tool: **ChromaDB**, a lightweight, open-source vector database that runs locally and has a clean Python API. No cloud dependencies, no API keys for the database itself, and it is simple enough to get running in an afternoon.

### Architecture: Three Collections

Rather than dumping everything into a single bucket, the ChromaDB instance is organized into three purpose-built collections:

| Collection | Purpose | Example Documents |
|---|---|---|
| `infrastructure` | Network topology, IP assignments, services, stack locations | "Wazuh SIEM is deployed at 192.168.1.7 on the Docker host" |
| `documentation` | How-to guides, runbooks, troubleshooting notes | "To restart the monitoring stack, run docker compose up -d from /opt/monitoring/" |
| `decisions` | Architecture decisions, rationale, lessons learned | "Chose Wazuh over ELK because of built-in HIDS and lower resource usage" |

This separation matters. When Claude needs to answer "What IP is Wazuh on?" it can target the `infrastructure` collection. When the question is "Why did we pick Wazuh?" it hits `decisions`. The collection structure acts as a coarse filter before the semantic search even begins, improving both speed and relevance.

Each document is stored with metadata -- source file, date added, tags -- so results can be filtered further when needed.

### MCP Integration: Making Memory Transparent

The piece that ties this all together is the **Model Context Protocol (MCP)**. MCP is a standard that lets AI assistants access external tools and data sources through a consistent interface. In this case, an MCP server wraps the ChromaDB instance and exposes it as a set of tools that Claude can call directly during a conversation.

The MCP server provides operations like:

```
chroma_query_documents   -- semantic search across a collection
chroma_add_documents     -- store new knowledge
chroma_get_documents     -- retrieve specific documents
chroma_list_collections  -- see what collections exist
```

From Claude's perspective, querying the memory database is no different from calling any other tool. There is no special prompt engineering, no manual "please check the database" instructions. The tools are simply available, and Claude uses them when they are relevant.

Here is what the MCP server configuration looks like in practice:

```json
{
  "mcpServers": {
    "homelab-memory": {
      "command": "uvx",
      "args": ["chroma-mcp", "--chroma-db-path", "/home/cib/.chroma-db"]
    }
  }
}
```

That is the entire integration. A locally running MCP server pointing at a ChromaDB directory on disk. No cloud services, no complex infrastructure -- just a Python process and a database directory.

### Real-World Example

Here is what this looks like in practice. I ask a simple question during a conversation:

> "What IP is Wazuh on?"

Behind the scenes, Claude calls the `chroma_query_documents` tool against the `infrastructure` collection with the query text "Wazuh IP address." ChromaDB converts that query into a vector, searches for the nearest documents, and returns something like:

```json
{
  "ids": [["infra-wazuh-001"]],
  "documents": [["Wazuh SIEM stack is deployed on the Docker host at 192.168.1.7.
                  The stack includes wazuh-manager, wazuh-indexer, and wazuh-dashboard.
                  Dashboard is accessible on port 443."]],
  "metadatas": [[{"source": "infrastructure-inventory", "updated": "2026-01-15"}]],
  "distances": [[0.23]]
}
```

Claude gets the document, sees the IP, and responds naturally: "Wazuh is at 192.168.1.7, with the dashboard on port 443."

The critical thing is what did *not* happen. I did not need to remember which file that information was in. I did not need to tell Claude to check a specific MEMORY.md. I did not need to use the exact keyword that was in the document. The semantic search bridged the gap between my question and the stored knowledge automatically.

### Benefits Over Previous Systems

The evolution across all three systems tells a clear story about scaling knowledge management:

| | System 1.0 | System 2.0 | System 3.0 |
|---|---|---|---|
| **Storage** | Single CLAUDE.md per project | Synced files via symlinks | ChromaDB vector database |
| **Retrieval** | Loaded at conversation start | Loaded at conversation start | Queried on demand |
| **Search** | Manual / keyword | Manual / keyword | Semantic similarity |
| **Scaling** | Limited by context window | Limited by context window | Grows independently of context |
| **Maintenance** | Manual XML editing | Manual with sync | Add documents programmatically |
| **Cross-project** | Copy/paste or symlinks | Symlinks | Single database, all projects |

The most significant shift is from **pre-loaded context** to **on-demand retrieval**. Systems 1.0 and 2.0 relied on front-loading everything Claude might need. System 3.0 lets Claude pull exactly what is relevant, when it is relevant. That is a fundamentally different approach to AI memory.

### Migration: Complementary, Not Replacement

An important design decision: System 3.0 does not replace Systems 1.0 and 2.0. It complements them.

The CLAUDE.md files still serve their purpose. They hold stable, structural information -- persona directives, interaction preferences, network ranges, project-level configuration. This is context that is relevant to *every* conversation and benefits from being loaded upfront.

ChromaDB handles the long tail: the growing body of documentation, decisions, and infrastructure details that are individually relevant to specific questions but collectively too large to load every time.

The migration has been gradual. As I work on projects and generate new documentation, it gets added to the appropriate ChromaDB collection. The existing XML files remain in place. Both systems coexist, and Claude draws from whichever is appropriate for the current question.

### Lessons Learned

**Vector databases are not just for enterprise RAG.** The tutorials and blog posts about vector databases tend to focus on massive document corpora and production AI applications. But the same technology works remarkably well for personal knowledge management. A homelab with thirty services and six months of architecture decisions is exactly the kind of growing, loosely-structured knowledge base that benefits from semantic search.

**MCP changes the integration model.** Before MCP, giving an AI access to external data meant carefully crafting prompts, pasting content, or building custom tooling. MCP makes external tools feel native. Claude does not treat the ChromaDB query as something special -- it is just another tool, like reading a file or running a command. That transparency is what makes the system practical rather than gimmicky.

**Semantic search changes how you document.** With keyword-based retrieval, you had to be disciplined about structure, naming, and tagging. With semantic search, you can write naturally and trust that the meaning will be captured. A casually written note about "switching from Elastic to Wazuh because of memory usage" will still surface when someone asks about SIEM architecture decisions months later. The pressure to maintain rigid structure drops significantly.

**Different knowledge types still want different storage.** XML is excellent for stable, hierarchical configuration. Vector databases are excellent for growing, queryable documentation. Trying to force everything into one system is a mistake. The best architecture uses each tool where it fits.

### What's Next

System 3.0 is functional but early. The roadmap includes:

- **Broader document ingestion.** Changelog entries, git commit summaries, and project notes are all candidates for collection storage. The more knowledge in the database, the more useful semantic search becomes.
- **Embedding model experimentation.** The default embedding model works well, but different models have different strengths. Testing alternatives for homelab-specific technical content could improve retrieval quality.
- **Automated collection updates.** Right now, documents are added manually or during conversations. The next step is building lightweight automation -- a script that watches for new documentation files and indexes them into the appropriate collection automatically.
- **Multi-modal memory.** Network diagrams, architecture sketches, dashboard screenshots. Vector databases can handle image embeddings alongside text. Being able to ask "Show me the network topology" and get back an actual diagram is a compelling future state.

The bigger picture is this: AI memory is not a solved problem. Each iteration of this system has taught me something about how humans and AI assistants can share context effectively. The shift from "load everything upfront" to "search for what you need" feels like the right architectural direction. The question now is how far that paradigm can scale -- and what new capabilities emerge when an AI assistant has genuine, searchable memory of every decision, every configuration, and every lesson learned across an entire infrastructure.

---

## 🔗 Series Navigation

- [← Part 1: Claude Code Memory System](../claude-memory/) - The foundation with XML context
- [← Part 2: Memory System 2.0](../claude-memory/#evolution-memory-system-20-february-2026) - Symlink architecture

---

## 🔗 Related Projects

- [HomeLab Infrastructure](../home-lab/) - The infrastructure this memory system documents
- [GitOps Workflow](../gitops/) - Docker Compose management with version control
