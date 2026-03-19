# Dynamic Tool List Changes Nuke the KV/Prefix Cache

Changing the tool list dynamically invalidates the KV/prefix cache for both
Anthropic and OpenAI. The system prompt — which includes tool definitions — is
cached as a prefix. Every tool add/remove evicts the cache entry and forces
recomputation. **Dynamic pruning can be slower than a static list** if you
change tools too frequently.

## When to Change Tools

Only mutate the tool list at **clear session boundaries**:

- After an explicit `init_mode` or `set_context` call from the user
- When transitioning between workflow phases (e.g., "explore" → "edit")
- On initial connection setup

**Do not** change the tool list on every request or per-message heuristics.

## Rule of Thumb

```
If tool_list_change_frequency > 1 per ~20 requests:
    you're probably hurting more than helping.
If tool_list is stable for the session:
    prefix cache stays warm → faster TTFT, lower cost.
```

## Example

```typescript
// ✅ Good: change tools once at session boundaries
server.onRequest("set_mode", (params) => {
  if (params.mode === "admin") enableAdminTools(); // one invalidation
});

// ❌ Bad: change tools on every request
server.onRequest("tools/call", (params) => {
  pruneIrrelevantTools(params.context); // cache miss every time
});
```

Static tool lists are cache-friendly. Batch mutations at session boundaries.

---

**Source:** u/crewrelaychat, r/mcp
