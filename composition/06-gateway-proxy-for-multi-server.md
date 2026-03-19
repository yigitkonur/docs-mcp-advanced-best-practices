# Use a Gateway/Proxy for Multi-Server Orchestration

Running multiple MCP servers creates real operational problems: tool name collisions, resource waste from idle servers, no unified discovery, and cascading failures when one server goes down. A gateway proxy solves all of these by sitting between the client and your server fleet.

**What a gateway handles:** server-prefixed naming (`web___search`, `db___query`) to avoid collisions, on-demand server lifecycle, circuit breaking for unreachable servers, and unified `tools/list` across all servers.

```typescript
// Gateway configuration — declare servers, let the gateway manage lifecycle
const gateway = new MCPGateway({
  servers: [
    {
      name: 'web',
      command: 'npx',
      args: ['-y', '@anthropic/web-search-mcp'],
      lazy: true,           // only start when a tool is called
      idleTimeout: 120_000, // stop after 2min of inactivity
      healthCheck: { interval: 30_000, timeout: 5_000 }
    },
    {
      name: 'db',
      command: 'npx',
      args: ['-y', 'postgres-mcp-server'],
      lazy: true,
      idleTimeout: 300_000,
      maxRetries: 3  // circuit breaker after 3 failures
    }
  ],
  naming: 'prefixed'  // web___search, db___query — prevents collisions
});

// Client sees one server with all tools
await gateway.start();
```

**Existing solutions:**
- **MCPJungle** — Multi-server management with prefixed naming and health checks
- **MCPX** — Dynamic server allocation with ephemeral containers
- **MetaMCP** — Control plane for managing MCP server fleets

**When you need this:**
- 3+ MCP servers running simultaneously
- Tool names conflict across servers (multiple servers expose `search` or `query`)
- Resource-constrained environments where idle servers waste memory
- Production deployments needing circuit breaking and monitoring

**When you don't:** If you have 1-2 servers with no name conflicts, direct connections are simpler. Don't add a gateway for the sake of architecture.

> **Source:** u/Rotemy-x10, r/mcp (269 upvotes) on gateway patterns; u/EconomicsDangerous44, r/mcp on multi-server orchestration; community discussion of MCPJungle, MCPX, and MetaMCP.
