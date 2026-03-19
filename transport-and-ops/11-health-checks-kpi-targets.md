# Health Checks with KPI Targets

Implement structured health checks that return per-component status and KPI metrics. This enables infrastructure monitoring, load-balancer routing, and agent-driven diagnostics with a single tool.

```typescript
interface ComponentHealth {
  status: "healthy" | "degraded" | "unhealthy";
  latency_ms?: number;
  error?: string;
  details?: Record<string, unknown>;
}

server.tool("health_check", "Check server health and KPIs. Returns per-component status.", {
  include_metrics: z.boolean().default(false)
    .describe("Set true to include request-rate and latency percentile metrics"),
}, async ({ include_metrics }) => {
  const [dbHealth, cacheHealth, upstreamHealth] = await Promise.allSettled([
    checkDatabase(),
    checkRedis(),
    checkUpstreamAPI(),
  ]);

  const components: Record<string, ComponentHealth> = {
    database: resolveHealth(dbHealth),
    cache: resolveHealth(cacheHealth),
    upstream_api: resolveHealth(upstreamHealth),
  };

  const allHealthy = Object.values(components).every(c => c.status === "healthy");
  const anyUnhealthy = Object.values(components).some(c => c.status === "unhealthy");

  const result: Record<string, unknown> = {
    status: anyUnhealthy ? "unhealthy" : allHealthy ? "healthy" : "degraded",
    components,
    uptime_seconds: process.uptime(),
  };

  if (include_metrics) {
    result.metrics = {
      requests_per_second: metrics.rps,
      p50_latency_ms: metrics.p50,
      p95_latency_ms: metrics.p95,
      p99_latency_ms: metrics.p99,
      error_rate_percent: metrics.errorRate,
    };
    // KPI targets for alerting
    result.kpi_targets = {
      target_rps: "> 1000",
      target_p95_ms: "< 100",
      target_error_rate: "< 0.1%",
      current_vs_target: {
        rps: metrics.rps >= 1000 ? "✅ passing" : "❌ below target",
        p95: metrics.p95 < 100 ? "✅ passing" : "❌ above target",
        error_rate: metrics.errorRate < 0.1 ? "✅ passing" : "❌ above target",
      }
    };
  }

  return { content: [{ type: "text", text: JSON.stringify(result, null, 2) }] };
});
```

## Production KPI Targets

| KPI | Target | Why |
|---|---|---|
| Throughput | > 1,000 req/s | Handles bursty agentic loops |
| P95 latency | < 100ms | Keeps agentic sessions responsive |
| P99 latency | < 500ms | Prevents cascading timeouts |
| Error rate | < 0.1% | LLM context isn't wasted on error recovery |
| Tool success rate | > 99% | Low failure rate means fewer retries |

**Why it matters:** Agents that call tools blindly during a degraded server state waste context budget on retries. A structured health tool lets orchestrators skip to a fallback server or pause the session until degradation resolves.

**Source:** Advanced MCP patterns research file (production deployment section); modelcontextprotocol.io observability guidance.
