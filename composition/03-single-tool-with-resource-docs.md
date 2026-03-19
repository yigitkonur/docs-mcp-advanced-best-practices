# Wrap Complex APIs in One Tool + Resource Documentation

For APIs with 30+ endpoints, expose a single flexible tool and use MCP resources to provide on-demand documentation. This keeps the tool count at 1 while still handling the full API surface.

```python
@mcp.tool(description="Execute an API operation. Use resources at tool://{client}/{method} to see available methods and parameters.")
def api_call(client: str, method: str, params: dict = {}) -> dict:
    """Single tool wrapping the entire API surface."""
    api_client = get_client(client)
    return api_client.call(method, **params)

@mcp.resource("tool://all")
def list_all_clients() -> str:
    """List all available API clients and their methods."""
    return json.dumps({c.name: list(c.methods.keys()) for c in clients})

@mcp.resource("tool://{client}/{method}")
def get_method_docs(client: str, method: str) -> str:
    """Detailed documentation for a specific API method."""
    return get_client(client).get_method_docs(method)

@mcp.resource("tool://{client}/{method}/{parameter}")
def get_parameter_docs(client: str, method: str, parameter: str) -> str:
    """Detailed docs for a specific parameter of a method."""
    return get_client(client).get_param_docs(method, parameter)
```

**How the interaction flows:**
1. Model calls `api_call` with a guess at the method
2. If wrong, error response says "See tool://all for available methods"
3. Model reads the resource to discover correct method name
4. Model reads `tool://{client}/{method}` for parameter documentation
5. Model calls `api_call` correctly

**Why this works:** Resources aren't preemptively pushed to context. The LLM reads them on-demand, so a seldom-used method with complex documentation doesn't waste tokens until it's actually needed.

**Trade-off:** Requires clients that support resource reading. Not all do. Test with your target client.

**Source:** u/Dipseth r/mcp - "I am using 1 tool that has 3 arguments... using prance to wrap an open API spec"
