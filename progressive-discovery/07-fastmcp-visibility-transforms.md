# FastMCP Visibility Transforms

FastMCP 3.0 introduced a layered visibility system that lets you show or hide tools at the session level without re-registering anything. This is the production-ready way to implement progressive tool disclosure in Python MCP servers.

## Disable by Tag (Apply to Every Session)

```python
from fastmcp import FastMCP

mcp = FastMCP("my-server")

# Disable all tools tagged "advanced" by default
mcp.disable(tags={"advanced"})
```

## Per-Session Overrides (Inside a Tool Handler)

```python
from fastmcp import FastMCP, Context

@mcp.tool(tags=["core"])
async def unlock_admin_mode(ctx: Context, role: str) -> str:
    """Unlock tools based on user role."""
    if role == "admin":
        # Enable advanced tools for this session via context
        await ctx.enable_components(tags={"advanced"})
        return "Admin tools unlocked."
    return "No additional tools available for this role."
```

You can also disable specific tools by name:

```python
mcp.disable(names={"dangerous_tool", "debug_tool"})
```

## Dynamic Unlock via Context Tool

```python
@mcp.tool(tags=["core"])
async def enable_advanced_mode(ctx: Context, access_code: str) -> str:
    """Unlock advanced tools for this session. Requires a valid access code."""
    if not verify_access_code(access_code):
        return "Invalid access code."
    
    # Enable advanced tools for this session only
    await ctx.enable_components(tags=["advanced"])
    return "Advanced tools unlocked. Available: analyze_deep, export_bulk, admin_override."
```

## Layer Priority

Global transforms → session transforms → tool-level annotations. Later layers override earlier ones, so a session transform can re-enable a tool that a global transform hid.

**Why it matters:** Without session-scoped visibility, you must re-register tools (which nukes the prefix cache) or maintain separate server instances per user tier. FastMCP transforms let you do progressive disclosure cleanly.

**Source:** [FastMCP 3.0 docs — visibility](https://gofastmcp.com/servers/visibility); [FastMCP context docs](https://gofastmcp.com/python-sdk/fastmcp-server-context)
