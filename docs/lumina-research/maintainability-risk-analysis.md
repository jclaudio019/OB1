# Maintainability & Risk Analysis

## Repository Activity & Structure
The repository is highly structured, divided clearly into `schemas`, `recipes`, `primitives`, `integrations`, and the core `server`. The use of `AGENTS.md` and automated PR reviews via Claude indicates a highly governed and active repository. The design explicitly encourages community contributions through a modular plugin-like structure (extensions/recipes).

## Architectural Complexity
- **SQL Heavy:** The core business logic (matching, upserting, graph traversal, deduplication) is written in PL/pgSQL (PostgreSQL functions). While this is highly performant and secure (due to RLS), it creates a high barrier to entry for standard TypeScript/Python developers.
- **Micro-Architectures:** The `agent-memory` and `ob-graph` schemas add significant relational complexity. `agent-memory` alone introduces 8 new tables to manage reviews, audits, and traces.

## Dependency Risk
- **Supabase Lock-in:** The project is deeply intertwined with Supabase. RLS policies, Edge Functions (Deno), and specific PostgreSQL extensions (`pgvector`, `pgcrypto`) make migrating off Supabase a non-trivial task.
- **Provider Lock-in:** `server/index.ts` hardcodes OpenAI models via OpenRouter. While an API key change is easy, modifying the embedding logic requires direct code edits.
- **Frameworks:** Uses Deno, Hono, and the MCP SDK. These are modern and active, but Deno vs Node.js could pose integration challenges if Lumina intends to eventually merge the codebase directly.

## Maintainability
- **High Modular Maintainability:** Because features are added as "sidecar" schemas (like `agent-memories` linking to `thoughts`), the core `thoughts` table is rarely mutated, ensuring high stability.
- **Documentation:** The project has excellent documentation, clear READMEs, and explicit dependency mappings (via `metadata.json` for primitives).
- **Testing:** Much of the testing is done via automated agent reviews in CI rather than traditional unit test suites (e.g., Jest/Mocha).

## Upgrade Difficulty
- **Schema Migrations:** Upgrading the system involves running raw SQL files. There is no automated migration tool (like Prisma or Alembic) used globally; users manually execute SQL files in Supabase. This makes versioning and upgrading fragile.
- **Vector Dimension Changes:** Changing embedding models (e.g., from 1536 dimensions to 768) requires massive manual migrations of the `pgvector` columns.

## Long-Term Viability for Lumina
- As a standalone external service accessed via MCP, Open Brain is highly viable and low-maintenance for Lumina.
- If Lumina attempts to fork and maintain this as an internal core system, the SQL complexity and Supabase dependency will drastically increase Lumina's maintenance burden.
