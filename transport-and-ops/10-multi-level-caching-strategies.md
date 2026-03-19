# Multi-Level Caching for MCP Responses

Tool calls that read data frequently return the same result within a session. Add a three-level cache hierarchy to eliminate redundant upstream calls without serving stale data.

```typescript
import { LRUCache } from "lru-cache";
import Redis from "ioredis";

interface CacheEntry { value: unknown; expiresAt: number; }

class MCPCache {
  // L1: in-process memory — fastest, session-local
  private l1 = new LRUCache<string, CacheEntry>({ max: 500, ttl: 60_000 });

  // L2: Redis — shared across instances, survives restarts
  private l2: Redis | null = null;

  // L3: persistent DB cache for expensive computations (e.g., embeddings)
  private l3Ttl = 86_400_000; // 24 hours

  constructor(redisUrl?: string) {
    if (redisUrl) this.l2 = new Redis(redisUrl);
  }

  async get(key: string): Promise<unknown | null> {
    // L1 hit
    const l1 = this.l1.get(key);
    if (l1 && l1.expiresAt > Date.now()) return l1.value;

    // L2 hit
    if (this.l2) {
      const raw = await this.l2.get(key);
      if (raw) {
        const parsed = JSON.parse(raw);
        this.l1.set(key, parsed); // warm L1
        return parsed.value;
      }
    }

    return null;
  }

  async set(key: string, value: unknown, ttlMs: number): Promise<void> {
    const entry = { value, expiresAt: Date.now() + ttlMs };
    this.l1.set(key, entry);
    if (this.l2) {
      await this.l2.set(key, JSON.stringify(entry), "PX", ttlMs);
    }
  }
}

const cache = new MCPCache(process.env.REDIS_URL);

// Wrap any tool with caching
async function cachedToolHandler(cacheKey: string, ttlMs: number, fn: () => Promise<unknown>) {
  const cached = await cache.get(cacheKey);
  if (cached) return cached;
  const result = await fn();
  await cache.set(cacheKey, result, ttlMs);
  return result;
}
```

## Cache TTL Strategy by Data Type

| Data Type | L1 TTL | L2 TTL | Notes |
|---|---|---|---|
| Repo file tree | 30s | 5min | Changes infrequently mid-session |
| Search results | 60s | 10min | Queries repeat in agentic loops |
| User profile | 5min | 1hr | Rarely changes |
| Embeddings / AI analysis | N/A (skip L1) | 24hr | Expensive to recompute |
| Live metrics / status | 5s | 30s | Must be near-real-time |

## Cache Key Design

Include version/ETag in the key when available:
```typescript
const key = `github:issues:${owner}/${repo}:${etag}`;
```

**Why it matters:** Agentic loops repeatedly call the same tools with identical params. Without caching, a 10-step agent workflow that queries file listings on each step can generate 10× the upstream API calls needed.

**Source:** [r/mcp](https://reddit.com/r/mcp) discussion on GitHub MCP server rate limit mitigation; [Docker Blog — MCP Server Best Practices](https://www.docker.com/blog/mcp-server-best-practices/)
