# Transport Choice Makes or Breaks MCP Server Performance

Transport selection isn't a minor config decision — it determines whether your MCP server can handle production concurrency at all. Real benchmarks from Kubernetes load testing tell the story:

| Transport | Concurrency | Success % | Req/s | Avg Response Time |
|---|:---:|:---:|:---:|:---:|
| stdio | 20 | 4% | 0.64 | 20s |
| SSE | 20 | 100% | 7.23 | 18ms |
| Streamable HTTP (shared pool) | 20 | 100% | 48.4 | 5ms |
| Streamable HTTP (shared pool) | 200 | 100% | 299.85 | 622ms |

**Why stdio collapses under concurrency:**
stdio uses a single process with stdin/stdout pipes. Every request is serialized. At concurrency 20, requests queue behind each other, most time out, and only 4% succeed. It's designed for single-user local development — nothing more.

**Why Streamable HTTP wins:**
- Shared session pooling amortizes connection overhead
- Stateless request/response model maps naturally to HTTP load balancing
- Standard infrastructure (reverse proxies, Kubernetes ingress, health checks) works out of the box
- Scales linearly: 300 req/s at concurrency 200 with sub-second response times

**SSE works but is deprecated:**
SSE (Server-Sent Events) handles concurrency fine at small scale but has limitations — it's unidirectional, doesn't support resumability, and is being removed from the MCP spec in favor of Streamable HTTP.

**Production recommendation:**
```
Local dev, single user     → stdio is fine
Shared team server         → Streamable HTTP
Production / multi-tenant  → Streamable HTTP + session pooling + load balancer
```

**If you're building a new MCP server today**, start with Streamable HTTP transport. The migration cost from stdio to HTTP later is non-trivial — you'll need to rethink session management, add health endpoints, and handle concurrent state.

**Source:** Stacklok — "Performance Testing MCP Servers in Kubernetes"
