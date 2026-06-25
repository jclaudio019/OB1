# Recommended Next Steps for Lumina

## Overview
Lumina should **use Open Brain directly via its MCP Adapter** in the short term, while **partially extracting** its agent-memory concepts into Lumina's native ORM in the long term. Lumina should **avoid rebuilding** the vector search layer entirely and instead delegate that to Open Brain.

## Recommendations

### 1. Short-Term: Use Directly via MCP
- **Action:** Build an MCP client in Lumina to connect to Open Brain's edge functions.
- **Why:** Immediate access to semantic search (`search_thoughts`) and robust memory storage (`capture_thought`) without any infrastructure overhead.
- **Priority:** High
- **Implementation Complexity:** Low
- **Strategic Value:** Extremely high. It gives Lumina instant long-term memory.

### 2. Mid-Term: Partially Extract Agent Memory Logic
- **Action:** Study `schemas/agent-memory/schema.sql` and `docs/safe-agent-memory-provenance.md`. Rebuild these concepts (provenance, evidence vs. instruction, review queues) natively inside Lumina's orchestration engine.
- **Why:** While Open Brain stores the data, Lumina (as the orchestrator) needs to *enforce* the rules around whether an agent is allowed to act on a memory. This governance logic belongs in Lumina.
- **Priority:** Medium
- **Implementation Complexity:** Medium
- **Strategic Value:** High. Prevents agent hallucination loops by enforcing strict use-policies on retrieved memory.

### 3. Long-Term: Avoid Full Internal Rebuild of Vector Infrastructure
- **Action:** Continue treating Open Brain (or an equivalent PostgreSQL/pgvector instance) as a delegated external microservice.
- **Why:** Embedding generation, cosine similarity math, HNSW index management, and Row Level Security are specialized domains. Rebuilding them inside a Node/Python orchestrator is an anti-pattern.
- **Priority:** Low (Active Avoidance)
- **Implementation Complexity:** N/A
- **Strategic Value:** High. Keeps Lumina focused on cognitive orchestration rather than database administration.

## Summary Checklist for Lumina Architects
1. [ ] Implement an MCP Client Adapter in Lumina.
2. [ ] Map Open Brain's `capture_thought` tool to Lumina's "Save Observation" agent action.
3. [ ] Map Open Brain's `search_thoughts` tool to Lumina's "Recall Context" agent action.
4. [ ] Design an internal Lumina Memory Manager that understands `evidence` vs `instruction` based on the Open Brain agent-memory spec.
