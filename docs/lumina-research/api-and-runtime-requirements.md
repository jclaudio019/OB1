# API and Runtime Requirements

## Required APIs
1. **Supabase / PostgreSQL:** The system requires a PostgreSQL database with the `pgvector` and `pgcrypto` extensions enabled. It heavily utilizes PostgreSQL RPCs (`match_thoughts`, `upsert_thought`) and Row Level Security (RLS).
2. **OpenRouter API:** Requires `OPENROUTER_API_KEY` to access specific models.
   - Embeddings: Hardcoded to use `openai/text-embedding-3-small`.
   - Metadata Extraction: Hardcoded to use `openai/gpt-4o-mini`.
3. **Model Context Protocol (MCP):** The server implements the MCP SDK via `@modelcontextprotocol/sdk/server/mcp.js`.

## Optional APIs
- Slack / Discord APIs (for integration pipelines and auto-capture recipes).
- Telegram APIs (used in specific recipes like `vercel-neon-telegram`).

## Local-Only Capability Support
- The core PostgreSQL schema and SQL functions can run completely locally on any Docker container that supports `pgvector` (e.g., standard PostgreSQL + pgvector).
- The repo includes an integration (`integrations/kubernetes-deployment/`) to self-host entirely without Supabase, which enables a local-only database.
- However, embeddings and metadata extraction are hardcoded to hit OpenRouter (cloud API) in the `server/index.ts` file. Running 100% offline would require modifying `getEmbedding` and `extractMetadata` in `server/index.ts` to use local models via Ollama (there is a recipe `recipes/local-ollama-embeddings/` that proves this is possible).

## Docker/Runtime Requirements
- **Deno Edge Functions:** The primary API surface (`server/index.ts` and `integrations/agent-memory-api/index.ts`) is designed to run on Supabase Edge Functions (which use Deno).
- **Hono Web Framework:** Used to serve the Deno APIs.

## GPU/Model Requirements
- If run natively out-of-the-box, no local GPU is required since inference hits OpenRouter APIs.
- If adapted to local (e.g., using `recipes/local-ollama-embeddings/`), a local GPU capable of running embedding models (like `nomic-embed-text`) is recommended for latency.

## Cloud Dependencies
- OpenRouter (for OpenAI models)
- Supabase (unless deployed locally via Docker/Kubernetes)

## Authentication Requirements
- **Supabase Service Role Key:** Required for Edge Functions to bypass RLS (`SUPABASE_SERVICE_ROLE_KEY`).
- **MCP Access Key:** A custom `MCP_ACCESS_KEY` is required in the headers (`x-brain-key`) or query string to authenticate MCP requests securely.
