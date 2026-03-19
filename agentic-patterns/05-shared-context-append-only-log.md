# Shared Context: Append-Only Event Log for Multi-Agent Coordination

When multiple agents share an MCP server, they need a coordination mechanism that avoids last-write-wins conflicts and preserves the full history of what each agent has done. An append-only event log is the simplest primitive that gives you both.

## Why Not Shared Mutable State?

Two agents reading-then-writing to the same record will silently clobber each other's work. Locks help but introduce deadlock risk. The append-only log sidesteps both: agents only write new entries, never update old ones, so concurrent writes are safe.

## Implementation

```typescript
interface LogEntry {
  id: string;
  agent_id: string;
  timestamp: string;          // ISO 8601
  type: "observation" | "decision" | "action" | "result" | "handoff";
  payload: unknown;
  confidence?: number;        // 0.0–1.0, optional
  supersedes?: string;        // ID of a previous entry this revises
}

// In-memory log (replace with Redis or Postgres for persistence)
const eventLog: LogEntry[] = [];

server.tool("log_event", "Append an event to the shared coordination log", {
  agent_id: z.string(),
  type: z.enum(["observation", "decision", "action", "result", "handoff"]),
  payload: z.unknown(),
  confidence: z.number().min(0).max(1).optional(),
  supersedes: z.string().optional().describe("ID of a prior log entry this entry revises"),
}, async ({ agent_id, type, payload, confidence, supersedes }) => {
  const entry: LogEntry = {
    id: crypto.randomUUID(),
    agent_id,
    timestamp: new Date().toISOString(),
    type,
    payload,
    confidence,
    supersedes,
  };
  eventLog.push(entry);
  return { content: [{ type: "text", text: `Logged event ${entry.id}` }] };
});

server.tool("read_log", "Read shared coordination log, optionally filtered", {
  since: z.string().optional().describe("ISO timestamp — only return entries after this time"),
  agent_id: z.string().optional().describe("Filter to a specific agent's entries"),
  type: z.string().optional().describe("Filter to a specific event type"),
  limit: z.number().default(50),
}, async ({ since, agent_id, type, limit }) => {
  let entries = eventLog;
  if (since) entries = entries.filter(e => e.timestamp > since);
  if (agent_id) entries = entries.filter(e => e.agent_id === agent_id);
  if (type) entries = entries.filter(e => e.type === type);
  return { content: [{ type: "text", text: JSON.stringify(entries.slice(-limit), null, 2) }] };
});
```

## Coordination Pattern

Each agent should `read_log` at session start to understand what others have done, and `log_event` (type: `handoff`) when passing work to another agent. The `supersedes` field allows a correction entry without erasing history.

**Why it matters:** Multi-agent pipelines that use simple shared variables inevitably produce race conditions. The append-only log pattern makes agent coordination safe without requiring locks or transactions.

**Source:** Multi-agent coordination patterns in r/mcp; IBM Context Forge multi-agent architecture docs; steering-patterns research file §5.
