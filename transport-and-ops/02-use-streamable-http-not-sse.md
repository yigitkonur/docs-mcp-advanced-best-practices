# Use Streamable HTTP, Not Deprecated SSE

The SSE (Server-Sent Events) transport is deprecated. New MCP servers should use Streamable HTTP for remote connections and stdio for local ones.

**Transport selection guide:**

| Scenario | Transport | Why |
|----------|-----------|-----|
| Local tool (same machine) | `stdio` | Simplest, no network overhead |
| Remote server | Streamable HTTP | Bidirectional, supports streaming, future-proof |
| Browser client | Streamable HTTP | Works with standard HTTP infrastructure |
| Legacy integration | SSE (if forced) | Only if the client doesn't support Streamable HTTP yet |

**Streamable HTTP advantages over SSE:**
- Bidirectional communication (SSE is server-to-client only)
- Works with standard HTTP load balancers and proxies
- Supports proper session management
- Compatible with OAuth authentication flows
- Better suited for production deployment behind CDNs

**Implementation:**
```python
from fastmcp import FastMCP

mcp = FastMCP("My Server")

# For local development
if __name__ == "__main__":
    mcp.run(transport="stdio")

# For remote deployment
# mcp.run(transport="http", host="0.0.0.0", port=8000)
```

**When deploying remotely:**
- Put behind a reverse proxy (nginx, Caddy)
- Enable HTTPS (required for production)
- Set appropriate CORS headers for browser clients
- Configure session affinity in load balancers

**Source:** [NearForm — Implementing MCP](https://nearform.com/digital-community/implementing-model-context-protocol-mcp-tips-tricks-and-pitfalls/); [MCP specification — transport](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports)
