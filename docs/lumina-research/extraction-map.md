# Extraction Map

This document highlights the exact files and modules within Open Brain that are most useful for Lumina to study, adapt, or extract.

## 1. Orchestration and Governance Logic Worth Studying
- `schemas/agent-memory/schema.sql`
  - **Why:** This is the most crucial file for Lumina. It defines how agent memory should be governed. Instead of agents just saving raw text, this schema enforces provenance (`observed`, `inferred`, `generated`), use-policies (`can_use_as_instruction`, `can_use_as_evidence`), and review queues (`pending`, `confirmed`, `rejected`).
  - **Extraction Value:** Lumina should extract the *concepts* of this schema into its own ORM/database structure.
- `schemas/agent-memory/README.md`
  - **Why:** Explains the philosophy behind the "evidence-only by default" approach for agent memory.

## 2. Memory Structures and Deduplication Worth Studying
- `docs/drafts/agent-memory-staging-base.sql` (and core setup docs)
  - **Why:** Contains the core `thoughts` table with the `vector(1536)` HNSW index.
  - **Extraction Value:** Shows the exact optimal `pgvector` index setup (`vector_cosine_ops`) for rapid semantic retrieval.
- `recipes/content-fingerprint-dedup/` or the `upsert_thought` RPC.
  - **Why:** Demonstrates how to use SHA-256 hashing on lowercase, whitespace-normalized content (`content_fingerprint`) to prevent agents from saving duplicate thoughts.

## 3. Graph Architecture
- `recipes/ob-graph/schema.sql`
  - **Why:** Demonstrates how to overlay a graph database (nodes and edges) on top of the vector store using PostgreSQL recursive CTEs.
  - **Extraction Value:** If Lumina needs to map relationships between memories (e.g., "Agent A depends on Lesson B"), this file provides a highly efficient SQL approach (`traverse_graph`, `find_shortest_path`).

## 4. Reusable Abstractions (API and Adapter level)
- `server/index.ts`
  - **Why:** Contains the complete Model Context Protocol (MCP) server implementation for Open Brain.
  - **Extraction Value:** If Lumina builds an MCP adapter, this file shows exactly how the tools (`search`, `fetch`, `search_thoughts`, `capture_thought`) map to underlying database operations.
- `integrations/agent-memory-api/index.ts`
  - **Why:** A REST API over Deno/Hono that handles agent write-backs and recall traces.
  - **Extraction Value:** Shows how to process and validate complex workflow memory schemas coming from an orchestrator.

## Low-Dependency Components
Most of the codebase is heavily dependent on PostgreSQL. However, the MCP tool definitions in `server/index.ts` (using Zod schemas for inputs) are abstract enough to be extracted and ported to a different backend architecture (e.g., MongoDB, SQLite).
