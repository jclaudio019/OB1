# Lumina Integration Analysis

## Capabilities Lumina Should Use Directly
- **Semantic Retrieval over MCP:** Open Brain acts as a standard Model Context Protocol (MCP) server. Lumina can directly connect to its edge functions via the MCP `StreamableHTTPTransport` to access `search_thoughts` and `capture_thought` without reinventing vector indexing infrastructure.
- **Agent Memory Governance:** The concepts in the `schemas/agent-memory/schema.sql` (provenance, confidence, review queues, use policy) are extremely valuable for a cognitive orchestrator. Lumina can use the logic directly or adopt these exact lifecycle and trust patterns to ensure agents don't hallucinate "instruction-grade" rules from unchecked logs.

## Capabilities That Should Remain External
- **Database Hosting & PostgreSQL Extensibility:** The heavy reliance on Supabase, `pgvector`, and Row Level Security (RLS) is highly robust but represents significant external infrastructure. Lumina should keep the database and edge function deployments external and interface over MCP or REST, rather than trying to embed PostgreSQL and HNSW indices directly into its core orchestration engine.
- **Third-party Integrations (Recipes/Scrapers):** Scripts that import data from ChatGPT, Slack, X/Twitter, or Obsidian (found in `recipes/` and `integrations/`) are valuable but should remain externally managed. Lumina does not need to own the ingestion pipelines.

## Capabilities That Should Eventually Be Extracted Internally
- **Memory Schemas & Deduplication Abstractions:** Lumina may eventually want to adopt the core conceptual schemas (e.g., `thoughts` structure, `content_fingerprint` for SHA-256 deduplication, and the graph node/edge pattern in `recipes/ob-graph/schema.sql`) into its own native data structures to create a more tightly integrated persistence layer.
- **Recall Trace & Audit Mechanisms:** The structures found in `agent_memory_recall_traces` and `agent_memory_audit_events` are critical for debugging complex agent chains. Lumina might extract this logic to build an internal dashboard for memory resolution traces.

## What is Tightly Coupled and Risky
- **Supabase Edge Function / RLS Coupling:** Open Brain relies heavily on Supabase Row Level Security (RLS), custom RPC functions (`match_thoughts`, `upsert_thought`), and Deno-based Edge Functions. Trying to migrate this specific implementation to a non-PostgreSQL environment (or self-hosted Node/Python without Supabase) would be a massive rewrite. It is risky to couple Lumina's core to Supabase-specific SQL extensions.
- **OpenRouter/OpenAI Embedding Hardcoding:** `server/index.ts` hardcodes `openai/text-embedding-3-small` and `openai/gpt-4o-mini` for embeddings and metadata extraction. This represents a vendor lock-in risk if Lumina prefers local LLMs or different providers.

## What Appears Stable and Reusable
- **The MCP Interface:** The Deno/Hono server exposing the MCP protocol (`search`, `fetch`, `search_thoughts`, `list_thoughts`, `thought_stats`, `capture_thought`) is a highly stable, standard boundary.
- **SQL Data Definitions (Schemas):** The raw schema definitions (`schema.sql` across multiple folders) provide a very stable relational architecture for managing vector data and agent continuity.
