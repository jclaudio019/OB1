# Adapter Recommendation

## Recommended Integration Strategy
Lumina should integrate with Open Brain via an **MCP (Model Context Protocol) Adapter**. Because Open Brain already exposes a mature, standard MCP server, Lumina does not need to write a custom REST wrapper or direct database connector.

### Recommended Interface Shape
Lumina should implement an `MCPClientProvider` that connects to the Open Brain MCP server over HTTP SSE (Server-Sent Events) or standard HTTP.

```typescript
interface OpenBrainAdapter {
  // Read-only tools exposed via MCP
  searchThoughts(query: string, limit?: number): Promise<MemoryResult[]>;
  listRecentThoughts(filters?: MemoryFilter): Promise<MemoryResult[]>;

  // Write tools
  captureThought(content: string): Promise<CaptureResult>;

  // Direct Agent Memory interactions (if using the agent-memory-api)
  recordMemoryTrace(trace: RecallTrace): Promise<void>;
  submitAgentMemory(memoryPayload: AgentMemoryPayload): Promise<void>;
}
```

## Minimal Integration Surface
1. **MCP Connection Setup:** Lumina configures an MCP Client pointing to the Open Brain edge function URL, passing `x-brain-key` in the headers.
2. **Tool Discovery:** Lumina calls MCP's standard `tools/list` to dynamically discover `search_thoughts` and `capture_thought`.
3. **Agent Delegation:** When Lumina's orchestrator needs memory, it simply hands the Open Brain MCP tools to the active agent, letting the agent query memory directly.

## Alternative: Direct API Wrapper
If Lumina wishes to bypass MCP for tighter orchestration control (especially for the `agent-memory` workflow), an API wrapper around `integrations/agent-memory-api` is recommended. This allows Lumina to explicitly pass `workspace_id`, `project_id`, and `flow_id` to track memory provenance precisely.

## Risks
- **Network Latency:** Communicating with a Supabase Edge Function over MCP introduces network hops compared to a local in-memory store.
- **Authentication:** Must securely manage and pass `MCP_ACCESS_KEY`.
- **Schema Divergence:** If Open Brain updates its MCP tool inputs (e.g., adding mandatory scope parameters), the Lumina adapter will need to adjust.

## Maintenance Concerns
- Since Open Brain relies heavily on Supabase RPCs, Lumina cannot easily mock the backend locally without running a full PostgreSQL instance.
- The MCP spec is evolving; Lumina must ensure its MCP client stays compatible with the `@modelcontextprotocol/sdk` version used in `server/index.ts`.
