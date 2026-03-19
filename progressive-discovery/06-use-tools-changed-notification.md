# Use `notifications/tools/list_changed` for Dynamic Tooling

When your server adds, removes, or modifies tools at runtime, emit the
`notifications/tools/list_changed` event. This is the **official MCP protocol
mechanism** for dynamic tooling. The client receives the notification,
re-fetches `tools/list`, and sees the updated set. Without it, clients use
stale definitions — calling removed tools or missing new ones.

## SDK Support: `RegisteredTool`

The TypeScript SDK's `RegisteredTool` handles notifications automatically.
Each mutation triggers `sendToolListChanged()` internally:

```typescript
const tool = server.registerTool("query_db", {
  description: "Run a read-only SQL query",
  inputSchema: { query: z.string() },
}, async ({ query }) => {
  return { content: [{ type: "text", text: await db.query(query) }] };
});

// Each triggers notifications/tools/list_changed automatically:
tool.disable();  // hides from tools/list
tool.enable();   // makes visible again
tool.update({ description: "Read-only SQL (max 1000 rows)" });
tool.remove();   // permanently removes
```

## Manual Notification

If you manage tools directly, emit the notification yourself:

```typescript
await server.notification({
  method: "notifications/tools/list_changed",
});
```

Use this **at session boundaries** (see tip 05) to avoid prefix cache
thrashing. The notification is cheap — the cache invalidation it triggers
on the LLM side is not.

---

**Source:** u/hasmcp, r/mcp; modelcontextprotocol/typescript-sdk source
