# Use a Meta-Server for Cross-Cutting Concerns

When running multiple MCP servers, don't duplicate auth, rate limiting, and logging in each one. Build a lightweight meta-server (gateway) that handles cross-cutting concerns and delegates to domain-specific servers.

```python
class MCPGateway:
    def __init__(self):
        self.servers = {}  # name -> MCP server connection
        self.middleware = [
            AuthMiddleware(),
            RateLimitMiddleware(max_requests=100, window_seconds=60),
            AuditLogMiddleware(),
        ]

    def register(self, name: str, server_config: dict):
        self.servers[name] = connect_mcp_server(server_config)

    async def handle_tool_call(self, tool_name: str, args: dict, ctx: Context):
        # Run middleware chain
        for mw in self.middleware:
            await mw.before(tool_name, args, ctx)

        # Route to correct server based on namespace
        server_name = tool_name.split("_")[0]  # e.g., "github" from "github_create_pr"
        server = self.servers[server_name]

        result = await server.call_tool(tool_name, args)

        for mw in reversed(self.middleware):
            result = await mw.after(tool_name, result, ctx)

        return result
```

**What the gateway handles:**
- **Authentication**: Verify user tokens before forwarding
- **Authorization**: Check per-tool permissions
- **Rate limiting**: Prevent abuse across all backend servers
- **Audit logging**: Single log stream for all tool calls
- **Response transformation**: Namespace tool names, filter sensitive fields

**FastMCP 3.0 approach:**
```python
mcp = FastMCP("Gateway")

# Mount sub-servers with namespace transforms
mcp.mount(github_server, namespace="github")
mcp.mount(jira_server, namespace="jira")
mcp.mount(slack_server, namespace="slack")

# Apply cross-cutting middleware
mcp.add_transform(AuthMiddleware(tag="all", scopes={"authenticated"}))
```

**The gateway pattern also enables:**
- Lazy loading of backend servers (only connect when first tool is called)
- Failover between redundant server instances
- Version routing (send 20% of traffic to v2)
- Tool visibility control per user/session

**Source:** [NearForm — Implementing MCP](https://nearform.com/digital-community/implementing-model-context-protocol-mcp-tips-tricks-and-pitfalls/); [FastMCP 3.0 blog](https://jlowin.dev/blog/fastmcp-3); [MCP specification](https://modelcontextprotocol.io)
