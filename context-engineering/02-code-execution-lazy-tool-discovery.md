# The Code Execution Pattern Saves 98% of Tool-Related Tokens

Instead of registering every tool definition upfront, expose a single `execute_code()` tool that runs scripts inside a sandbox. The model calls `list_servers()` to get names only, then `list_tools(server)` for names plus one-line descriptions, then `get_tool_schema(server, tool)` only when it actually needs a specific tool. Full schemas are fetched on demand and discarded after the call rather than injected permanently into the context.

Benchmarks from the Anthropic Engineering blog ("Code Execution with MCP") show the savings are dramatic at scale. A session with 1,000 tool definitions and a 2-hour transcript drops from 150k tokens to 2k — a 98.7% reduction. A 10,000-row spreadsheet filter task falls from 10k to 0.5k (95%). Loop-based Slack polling that previously paid the full context cost per iteration compresses to a single script call (80% reduction). The pattern trades cold-start latency and Docker overhead for massive token efficiency.

The main production caveat (u/DurinClash, 215 upvotes) is transport: this pattern works cleanly with stdio-based local servers, but most production deployments use remote HTTP/SSE servers where OAuth flows and session management make dynamic discovery significantly more complex. Evaluate whether your deployment context supports it before committing to the architecture.

```python
# Single gateway tool registered in the MCP server
@server.tool()
async def execute_code(code: str, language: str = "python") -> str:
    """
    Execute code in sandbox. Use list_servers(), list_tools(server),
    get_tool_schema(server, tool) to discover capabilities on demand.
    """
    sandbox_globals = {
        "list_servers": lambda: ["filesystem", "github", "slack"],
        "list_tools": _list_tools_brief,       # returns name + 1-line desc only
        "get_tool_schema": _get_schema_on_demand,  # full JSON schema on request
        "call_tool": _call_tool,
    }
    exec(compile(code, "<sandbox>", "exec"), sandbox_globals)
    return sandbox_globals.get("__result__", "")

async def _list_tools_brief(server: str) -> list[dict]:
    # Returns minimal shape — no full schemas injected into context
    tools = await registry.get_tools(server)
    return [{"name": t.name, "summary": t.description[:80]} for t in tools]

async def _get_schema_on_demand(server: str, tool: str) -> dict:
    # Full schema fetched only when the model explicitly requests it
    return await registry.get_full_schema(server, tool)
```

```python
# Model usage pattern — lazy discovery in practice
code = """
servers = list_servers()                     # → ["filesystem", "github"]
tools   = list_tools("github")               # → [{"name": "create_pr", "summary": "..."}]
schema  = get_tool_schema("github", "create_pr")  # → full JSON schema
result  = call_tool("github", "create_pr", {"title": "Fix bug", "body": "..."})
__result__ = result
"""
```

**Why it matters:** Paying the token cost for 1,000 tool definitions you might never call is like loading an entire library into RAM for a single book lookup — the code execution pattern turns that into a targeted fetch.

**Source:** Anthropic Engineering blog "Code Execution with MCP"; github.com/elusznik/mcp-server-code-execution-mode; u/DurinClash r/ClaudeAI (215 upvotes).
