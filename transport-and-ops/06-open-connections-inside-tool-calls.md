# Open External Connections Inside Tool Calls, Not at Startup

Open database connections, API clients, and external service handles **inside each tool call**, not at server startup. Accept the slight latency hit for dramatically higher reliability.

**Why this matters:**

When you connect at startup, a misconfigured database URL or a temporarily unavailable API prevents the server from starting at all. The MCP client can't even list available tools — the server is completely invisible. The user sees "failed to connect to MCP server" with no context about what's wrong.

**Wrong — connection at startup blocks everything:**
```typescript
// Server fails to start if DB is down
const db = new Pool({ connectionString: process.env.DATABASE_URL });
await db.connect(); // Throws → server never registers tools

const server = new McpServer({ name: "analytics" });
server.tool("query_metrics", schema, async (params) => {
  const result = await db.query(params.sql);
  return { content: [{ type: "text", text: JSON.stringify(result.rows) }] };
});
```

**Right — connect per tool call:**
```typescript
const server = new McpServer({ name: "analytics" });

server.tool("query_metrics", schema, async (params) => {
  let db;
  try {
    db = new Pool({ connectionString: process.env.DATABASE_URL });
    const result = await db.query(params.sql);
    return { content: [{ type: "text", text: JSON.stringify(result.rows) }] };
  } catch (err) {
    return {
      content: [{ type: "text", text: `Database error: ${err.message}. Check DATABASE_URL.` }],
      isError: true,
    };
  } finally {
    await db?.end();
  }
});
```

**The trade-off is worth it:**
- Tool listing always works, even when external services are down
- Connection errors surface as structured tool errors the agent can reason about
- Each tool is independently resilient — a broken database doesn't kill your file tools
- Connection pooling can still be used; just initialize the pool lazily on first tool call

**For high-frequency tools**, use a lazy singleton pattern: initialize the connection on first use, cache it, and handle reconnection inside the tool if it goes stale.

**Source:** Docker Blog — "MCP Server Best Practices"
