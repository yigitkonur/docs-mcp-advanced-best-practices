# Generate MCP Servers from OpenAPI Specs

If you already have a well-documented REST API with an OpenAPI spec, you don't need to hand-code each MCP tool. Generate them. Notion's MCP server uses exactly this pattern — it loads the OpenAPI spec and creates an `MCPProxy` that exposes every endpoint as a tool automatically.

**How it works:**

```typescript
import { MCPProxy } from './proxy';

// Load spec and create proxy — each endpoint becomes a tool
export async function initProxy(specPath: string, baseUrl?: string) {
  const openApiSpec = await loadOpenApiSpec(specPath, baseUrl);
  const proxy = new MCPProxy('Notion API', openApiSpec);
  return proxy;
}

// The proxy translates MCP tool calls into HTTP requests:
// tool("createPage", { parent: {...}, properties: {...} })
//   → POST /v1/pages { parent: {...}, properties: {...} }
//   → returns structured MCP response
```

The proxy handles parameter mapping, request construction, and response translation. You get instant MCP coverage for your entire API surface.

**When to use this:**
- You have 20+ API endpoints and writing individual tools is tedious
- Your OpenAPI spec is accurate and well-maintained
- You want quick MCP access while iterating on the API itself

**Caveats:**
- Auto-generated tool descriptions inherit whatever's in your spec — often too terse or too technical for LLM consumption
- One-to-one endpoint mapping creates tool sprawl. Consolidate related endpoints (e.g., merge `GET /users/{id}`, `PATCH /users/{id}`, `DELETE /users/{id}` into a single `manage_user` tool)
- Missing: semantic grouping, smart defaults, and context-aware parameter descriptions

**Recommended approach:** Use the proxy as a starting point, then iteratively refine the highest-traffic tools with hand-crafted descriptions, better parameter schemas, and consolidated operations.

```typescript
// After auto-generation, override specific tools for better LLM experience
proxy.overrideTool('search_pages', {
  description: 'Search Notion pages by title or content. Returns page IDs and titles.',
  parameters: {
    query: { type: 'string', description: 'Search text — matches titles and page content' },
    limit: { type: 'number', description: 'Max results (default: 10, max: 100)' }
  }
});
```

> **Source:** [makenotion/notion-mcp-server](https://github.com/makenotion/notion-mcp-server) — production example of OpenAPI-to-MCP proxy generation.
