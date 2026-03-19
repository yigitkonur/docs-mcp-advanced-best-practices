# Conversation Compaction Priorities

Long MCP sessions accumulate messages until they overflow the context window.
Naive truncation (drop oldest) loses critical schema context. Use priority-aware
compaction instead.

## Priority Categories

```
| Priority    | What It Is                        | Policy            |
|-------------|-----------------------------------|-------------------|
| Anchor      | Schema definitions, structural    | Always keep       |
| Important   | Substantial query results, data   | Keep              |
| Contextual  | Useful background, explanations   | Summarize if full |
| Routine     | Ordinary dialogue, confirmations  | May drop          |
| Transient   | Acks, "ok", status pings          | Drop first        |
```

Compact bottom-up: drop Transient → Routine → summarize Contextual.
Anchor and Important survive until session ends.

## Cache Compaction with Content Hash

```typescript
import { createHash } from "crypto";

async function getCompactedContext(messages: Message[]): Promise<string> {
  const hash = createHash("sha256")
    .update(messages.map(m => m.content).join("|"))
    .digest("hex");
  const cached = await redis.get(`compaction:${hash}`);
  if (cached) return cached; // O(1) reuse
  const compacted = await summarize(messages);
  await redis.setex(`compaction:${hash}`, 3600, compacted);
  return compacted;
}
```

## Classify at Creation Time

```typescript
function classifyMessage(msg: McpMessage): Priority {
  if (msg.type === "schema" || msg.type === "tool_definition") return "anchor";
  if (msg.type === "tool_result" && msg.content.length > 500)  return "important";
  if (msg.type === "tool_result")                              return "contextual";
  if (msg.role === "assistant" && msg.content.length < 20)     return "transient";
  return "routine";
}
```

Classify upfront, compact bottom-up, cache results for O(1) reuse.

---

**Source:** [pgEdge — Lessons Learned Writing an MCP Server for PostgreSQL](https://www.pgedge.com/blog/lessons-learned-writing-an-mcp-server-for-postgresql)
