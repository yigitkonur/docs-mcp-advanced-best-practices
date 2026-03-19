# When to Use Resources vs Tools

Resources and tools are both data access mechanisms, but they serve different purposes. Picking the wrong one wastes tokens or limits functionality.

| Dimension | Resource | Tool |
|-----------|----------|------|
| **Trigger** | User/client-initiated | Model-initiated |
| **Mutability** | Read-only | Can read AND write |
| **Addressing** | URI-based (`file://docs/api.md`) | Name-based (`search_docs`) |
| **Parameters** | Only URI path variables | Full JSON Schema input |
| **Caching** | Supports ETags, subscriptions | Each call is independent |
| **Token cost** | Typically lower (focused data) | Higher (schema in context) |
| **Client support** | Inconsistent across clients | Universal |

**Use a resource when:**
- Data is read-only and doesn't change per-request
- Content is reused across multiple tool calls (e.g., documentation, config)
- You want URI-addressable content for cross-server references
- The data is >80% read-only (project files, schemas, templates)

**Use a tool when:**
- The operation has side effects (creates, updates, deletes)
- Parameters are complex (filters, pagination, sorting)
- The model needs to decide when to access the data
- You need arbitrary input validation beyond URI matching

**The hybrid pattern:** Expose data as a resource for browsing, but provide a tool for complex queries against the same data.
```python
# Resource: simple access by ID
@mcp.resource("customers://{customer_id}")
def get_customer(customer_id: str): ...

# Tool: complex search with filters
@mcp.tool
def search_customers(query: str, status: str = "active", limit: int = 10): ...
```

**Community reality check:** Most builders only use tools because client support for resources is inconsistent. Resources work well in Goose and some clients but Claude Desktop/Code support is limited. If portability matters, default to tools and add resources as a nice-to-have.

**Source:** [u/Dipseth on r/mcp](https://reddit.com/r/mcp) — uses resource templates for API documentation; [u/dankelleher on r/mcp](https://reddit.com/r/mcp) — "Does anyone use MCP prompts or resources?" thread
