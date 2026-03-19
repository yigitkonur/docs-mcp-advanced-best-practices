# Parameter Dependency Chain

Force tools to be called in the correct sequence by requiring outputs from earlier tools as mandatory inputs to later ones. This turns implicit workflow ordering into a structural constraint enforced by the schema, not just by documentation.

Without this pattern, an agent may skip steps or call `deploy_service` before `run_tests`. With it, the server issues a schema validation error if the prerequisite token is absent — making the shortcut impossible.

## Implementation

```typescript
// Step 1: run_tests returns a signed token
server.tool("run_tests", "Run test suite. Returns test_token required for deployment.", {
  suite: z.string(),
}, async ({ suite }) => {
  const results = await runTestSuite(suite);
  if (results.failed > 0) {
    return { content: [{ type: "text", text: `Tests failed: ${results.failed} failures` }], isError: true };
  }
  // Sign a token proving tests passed
  const testToken = sign({ suite, passed: true, ts: Date.now() }, process.env.TOKEN_SECRET!);
  return {
    content: [{ type: "text", text: `All ${results.passed} tests passed.\ntest_token: ${testToken}\n\nNow call deploy_service with this test_token.` }]
  };
});

// Step 2: deploy_service REQUIRES the token from step 1
server.tool("deploy_service", "Deploy to production. Requires test_token from run_tests.", {
  service_name: z.string(),
  environment: z.enum(["staging", "production"]),
  test_token: z.string().describe("Token returned by run_tests. Required."),
}, async ({ service_name, environment, test_token }) => {
  const payload = verify(test_token, process.env.TOKEN_SECRET!);
  if (!payload?.passed) {
    return { content: [{ type: "text", text: "Invalid test_token. Run run_tests first." }], isError: true };
  }
  await deployService(service_name, environment);
  return { content: [{ type: "text", text: `Deployed ${service_name} to ${environment}.` }] };
});
```

## Why It's Better Than Documentation

A description saying "call run_tests before deploy" relies on the model reading and following instructions. A required `test_token` parameter makes skipping structurally impossible — the call will fail schema validation before your handler even runs.

**Why it matters:** Agentic workflows that modify production systems need hard ordering guarantees. Structural dependency chains are the only reliable way to enforce them across all LLMs.

**Source:** [MCP specification](https://modelcontextprotocol.io/specification/2025-11-25/server/tools) — tool design guidance; community patterns from [r/mcp](https://reddit.com/r/mcp)
