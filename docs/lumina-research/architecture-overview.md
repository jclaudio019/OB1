# Architecture Overview

## High-Level Architecture
Open Brain is structured as an infrastructure layer meant to act as a unified, persistent memory and data gateway for any AI agent or interface. It does not implement complex orchestration directly but provides the primitives for storage, semantic retrieval, and governed interaction.

The system relies on three core layers:
1. **Persistent Store:** A PostgreSQL database (typically hosted via Supabase) that serves as the single source of truth. It uses `pgvector` for embedding storage and similarity search.
2. **AI Gateway & Transport:** Deno-based Edge Functions (using Hono) that expose a Model Context Protocol (MCP) server or REST APIs, allowing AI agents to search, list, summarize, and capture memory.
3. **Extensions & Primitives:** A modular ecosystem of schemas, SQL functions, and recipes (e.g., `agent-memory`, `ob-graph`) that extend the core `thoughts` table with specialized capabilities like governed agent memory, graphs, deduplication, and workflow states.

## Execution Model
Open Brain itself is runtime-agnostic and relies on external AI systems (e.g., Claude, ChatGPT, OpenClaw, Codex) for execution.
- It acts purely as a memory backend and retrieval service.
- The execution model is "bring your own runtime" where external agents query the Open Brain MCP server during their own execution loops to retrieve context, fetch thoughts, and store outputs via write-backs.

## Memory Model
The memory model is highly robust, built on a relational database augmented with vector embeddings.
- **Core Unit:** The `thoughts` table stores unstructured text alongside JSONB metadata, content fingerprints (for deduplication), and `vector(1536)` embeddings.
- **Semantic Retrieval:** Uses `pgvector` with HNSW indices to perform cosine similarity searches on embeddings generated via OpenAI models (accessed through OpenRouter).
- **Agent Memory:** The `schemas/agent-memory` schema introduces governed operational memory. It implements provenance tracking, review queues, use policies (instruction-grade vs. evidence-only), and recall traces to manage and audit memory written by agents.
- **Graph Layer:** The `recipes/ob-graph` schema provides an optional node and edge pattern (with recursive CTEs for traversal) to overlay graph relationships on top of the linear thought store.

## Planning & Orchestration Structure
Open Brain delegates planning and orchestration to external systems (such as OpenClaw). It supports these systems by providing structured schemas for continuity:
- **Traceability:** It logs "recall traces" and "audit events" so that orchestration systems can debug their memory context.
- **Governed Write-backs:** Orchestration agents write their intermediate decisions, lessons, or outputs back to the database as "evidence." A human or trusted policy can promote this to "instruction" for future plans.

## Dependency Graph
- **Database:** PostgreSQL with `pgvector` and `pgcrypto` extensions (Supabase ecosystem is the primary target).
- **Runtime:** Deno Edge Functions.
- **Frameworks:** Hono (web framework), Model Context Protocol (MCP) SDK.
- **AI Models:** OpenRouter API is required for generating embeddings (`openai/text-embedding-3-small`) and extracting metadata (`openai/gpt-4o-mini`).
- **Clients/Consumers:** Any MCP-compatible client (Claude Desktop, Cursor, etc.).

## Key Subsystems
1. **Core Database (SQL):** `public.thoughts` table and `match_thoughts` RPC for semantic matching.
2. **MCP Server (Deno/Hono):** Located in `server/index.ts`, exposing tools like `search`, `fetch`, `search_thoughts`, `list_thoughts`, `thought_stats`, and `capture_thought`.
3. **Agent Memory Integration:** Located in `integrations/agent-memory-api/index.ts` and `schemas/agent-memory/schema.sql`. Manages the lifecycle, review, and recall of operational agent memory.
4. **Data Ingestion Recipes:** Various scripts in `recipes/` and `integrations/` (like Slack and Discord capture) to push data into the persistent store.
