# Break Tools Over 40 Parameters

40 parameters is the hard ceiling. Even 10+ degrades accuracy to ~85%.

## Accuracy by Parameter Count

| Parameters | Accuracy |
|-----------|----------|
| 3–6       | ~98%     |
| 10–15     | ~85%     |
| 20–30     | ~75%     |
| 40+       | < 60%    |

## Strategy 1: Split by Workflow Stage

Break one mega-tool into sequential steps:

```typescript
// ❌ One tool with 30+ params
server.tool("deploy", { env, region, scaling, healthCheck, rollback, ... })

// ✅ Three focused tools
server.tool("deploy_configure", {
  env: z.enum(["staging", "production"]),
  region: z.string(),
});
server.tool("deploy_execute", {
  configId: z.string().describe("ID from deploy_configure"),
  strategy: z.enum(["rolling", "blue-green"]),
});
server.tool("deploy_verify", {
  deploymentId: z.string().describe("ID from deploy_execute"),
});
```

## Strategy 2: Action-Routed Facade

```typescript
server.tool("database", {
  action: z.enum(["query", "insert", "schema"]),
  table: z.string(),
  where: z.string().optional().describe("Only for 'query'"),
  data: z.string().optional().describe("Only for 'insert'"),
});
```

## Rules

1. **Under 6:** ideal
2. **6–15:** acceptable if flat
3. **15–40:** split into 2–3 tools
4. **40+:** mandatory split
---

**Source:** u/raghav-mcpjungle, r/mcp; deep research with measured thresholds
