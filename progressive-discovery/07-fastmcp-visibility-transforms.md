# FastMCP Visibility Transforms

FastMCP 3.0 introduced a layered visibility system that lets you show or hide tools at the session level without re-registering anything. This is the production-ready way to implement progressive tool disclosure in Python MCP servers.

## Global Transforms (Apply to Every Session)

```python
from fastmcp import FastMCP
import mcp.types as types

mcp = FastMCP("my-server")

# Disable all tools tagged "advanced" by default
@mcp.global_transform()
async def hide_advanced_by_default(
    tool: types.Tool,
    context: dict
) -> types.Tool | None:
    if "advanced" in (tool.tags or []):
        return None  # None = hide this tool
    return tool
```

## Session Transforms (Per-Session Overrides)

```python
# In your session initialization handler:
async def on_session_start(session_id: str, user_role: str):
    if user_role == "admin":
        # Unlock advanced tools for this session
        await mcp.enable(tags=["advanced"], session_id=session_id)
    if user_role == "readonly":
        # Further restrict: only show read-prefixed tools
        await mcp.disable(
            filter=lambda t: not t.name.startswith("read_"),
            session_id=session_id
        )
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

**Source:** FastMCP 3.0 documentation (gofastmcp.com); dynamic-tools-patterns research file §3; r/mcp discussion on session-based tool visibility.
