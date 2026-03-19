# Session Pooling for 10x Throughput

A shared pool of 10 sessions delivers ~10x higher throughput vs
unique-session-per-request. Benchmarks from Stacklok's Kubernetes testing:

```
| Configuration         | Req/s   | Avg Response |
|-----------------------|---------|-------------|
| Unique sessions       | 30-36   | 500ms+      |
| Shared pool (10)      | 290-300 | 5ms         |
| stdio (single)        | 0.64    | 20s         |
```

The bottleneck is connection setup overhead — TLS handshake, protocol
negotiation, state initialization. Pooling amortizes this.

## Implementation

```typescript
import { createPool } from "generic-pool";

const sessionPool = createPool({
  create: async () => {
    const session = await mcpClient.connect(serverUrl);
    await session.initialize();
    return session;
  },
  destroy: async (session) => await session.close(),
}, { min: 2, max: 10, idleTimeoutMs: 60_000 });

async function callTool(name: string, args: Record<string, unknown>) {
  const session = await sessionPool.acquire();
  try {
    return await session.callTool({ name, arguments: args });
  } finally {
    sessionPool.release(session);
  }
}
```

Externalize session state so any pooled connection can serve any request:

```typescript
await redis.setex(`session:${sid}`, 3600, JSON.stringify(state));
const state = JSON.parse(await redis.get(`session:${sid}`));
```

Pool connections, externalize state, let the pool handle lifecycle.

---

**Source:** Stacklok — "Performance Testing MCP Servers in Kubernetes"
