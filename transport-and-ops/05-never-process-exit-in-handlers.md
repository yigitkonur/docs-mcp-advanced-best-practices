# Never Call process.exit() or sys.exit() Inside Tool Handlers

Calling `process.exit()` (Node) or `sys.exit()` (Python) inside a tool handler kills the entire MCP server process. The agent doesn't get a tool error — it gets a transport-level disconnection. It can't retry, can't fall back, can't even tell the user what went wrong. Every other tool on that server dies with it.

**What happens from the agent's perspective:**
```
Agent → call tool "deploy" → transport error: connection reset
Agent → retry? Server is gone. No tool listing available.
Agent → "I'm unable to reach the MCP server."
```

**Wrong — kills the server:**
```python
@server.tool("validate_config")
async def validate_config(path: str) -> list[TextContent]:
    config = load_config(path)
    if not config:
        print("Fatal: invalid config", file=sys.stderr)
        sys.exit(1)  # Server dies. All tools gone.
```

**Right — structured error, server stays alive:**
```python
@server.tool("validate_config")
async def validate_config(path: str) -> list[TextContent]:
    config = load_config(path)
    if not config:
        return [TextContent(
            type="text",
            text=f"Config at '{path}' is invalid or missing. "
                 f"Expected YAML with keys: host, port, db_name."
        )]
        # isError: true is set via raise or return convention
```

**The principle:** If a tool can't do its job, return `isError: true` with actionable guidance. Keep the server alive so other tools remain available. Design every handler for graceful degradation — the agent can work around one broken tool, but not a dead server.

**Also catch unhandled exceptions** in your tool handlers. A top-level try/except that converts crashes into structured errors prevents a single bad tool from taking down the entire server.

```python
async def safe_handler(func, **kwargs):
    try:
        return await func(**kwargs)
    except Exception as e:
        return [TextContent(type="text", text=f"Internal error: {type(e).__name__}: {e}")]
```

**Source:** u/rhuanbarreto, r/softwarearchitecture
