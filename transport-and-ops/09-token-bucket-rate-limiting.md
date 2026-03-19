# Token Bucket Rate Limiting per Tool Category

Don't apply a single global rate limit to your MCP server. Different tool categories have radically different cost profiles: a read that queries a local cache is cheap; an AI inference call is expensive. Use per-category token buckets.

```typescript
interface BucketConfig {
  capacity: number;    // Max tokens in bucket
  refillRate: number;  // Tokens added per second
}

class TokenBucket {
  private tokens: number;
  private lastRefill: number;

  constructor(private config: BucketConfig) {
    this.tokens = config.capacity;
    this.lastRefill = Date.now();
  }

  consume(cost = 1): boolean {
    const now = Date.now();
    const elapsed = (now - this.lastRefill) / 1000;
    this.tokens = Math.min(this.config.capacity, this.tokens + elapsed * this.config.refillRate);
    this.lastRefill = now;

    if (this.tokens < cost) return false;
    this.tokens -= cost;
    return true;
  }

  get retryAfterMs(): number {
    return Math.ceil(((1 - this.tokens) / this.config.refillRate) * 1000);
  }
}

const rateLimiters: Record<string, TokenBucket> = {
  read:  new TokenBucket({ capacity: 120, refillRate: 10 }), // generous: cheap reads
  write: new TokenBucket({ capacity: 30,  refillRate: 2  }), // moderate: writes have side effects
  ai:    new TokenBucket({ capacity: 10,  refillRate: 0.5 }), // tight: AI calls are expensive
};

const toolCategory: Record<string, keyof typeof rateLimiters> = {
  search_files:   "read",
  list_resources: "read",
  update_record:  "write",
  create_issue:   "write",
  analyze_code:   "ai",
  summarize:      "ai",
};

function withRateLimit(category: string, handler: Function) {
  return async (...args: any[]) => {
    const bucket = rateLimiters[category];
    if (bucket && !bucket.consume()) {
      return {
        content: [{ type: "text", text: JSON.stringify({
          error_category: "RATE_LIMITED",
          message: `Rate limit for ${category} tools exceeded.`,
          retryable: true,
          retry_after_ms: bucket.retryAfterMs,
          suggested_actions: ["wait_and_retry"],
        }) }],
        isError: true,
      };
    }
    return handler(...args);
  };
}
```

## Recommended Starting Limits

| Category | Capacity | Refill Rate | Rationale |
|---|---|---|---|
| read | 120 | 10/s | Local cache; near-free |
| write | 30 | 2/s | DB writes; moderate cost |
| ai | 10 | 0.5/s | LLM inference; expensive |
| external_api | 20 | 1/s | Third-party rate limits |

**Why it matters:** Flat global limits either over-restrict cheap reads or under-protect expensive AI calls. Categorical buckets let you tune each independently and surface informative retry-after values to the agent.

**Source:** Advanced MCP patterns research; community discussion on r/mcp regarding production server throttling.
