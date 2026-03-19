# Use Content Annotations to Separate User-Facing and Assistant-Facing Data

MCP supports content annotations with `audience` and `priority` fields on every content block. Use them to embed debug info, telemetry, and internal context that the model can reason about without cluttering the user's view.

**Without annotations — everything goes to the user:**
```
Found 15 matching records.
Cache hit ratio: 0.95, latency: 150ms, query plan: seq_scan on users_idx.
3 results filtered by permission check. Auth token expires in 240s.
```

The user sees implementation details they don't care about.

**With annotations — layered content:**

```typescript
server.registerTool("search_records", {
  inputSchema: { query: z.string() },
}, async ({ query }) => {
  const { results, meta } = await search(query);

  return {
    content: [
      {
        type: "text",
        text: `Found ${results.length} matching records.`,
        annotations: {
          priority: 1.0,
          audience: ["user", "assistant"],
        },
      },
      {
        type: "text",
        text: [
          `Debug: cache_hit=${meta.cacheHit}, latency=${meta.latencyMs}ms`,
          `Query plan: ${meta.queryPlan}`,
          `Filtered: ${meta.filteredCount} by permissions`,
          `Auth token TTL: ${meta.tokenTTL}s`,
        ].join("\n"),
        annotations: {
          priority: 0.3,
          audience: ["assistant"],
        },
      },
    ],
  };
});
```

The model sees everything and can use the debug data to optimize subsequent calls (e.g., "latency is high, let me add a filter to narrow the query"). The user only sees the clean summary.

**Priority levels to use:**
- `1.0` — Critical info the user asked for
- `0.7` — Supplementary context (pagination hints, next steps)
- `0.3` — Debug/telemetry data for the model only
- `0.1` — Verbose trace data, only relevant if something fails

**Note:** Client support for annotations varies. Well-behaved clients will respect `audience` and hide assistant-only content. Clients that ignore annotations will show everything — so keep assistant-only content informative but not confusing if a user happens to see it.

**Source:** modelcontextprotocol/servers — everything server reference implementation; MCP specification for content annotations.
