# Session-Based Progressive Tool Unlocking

Hide sensitive or advanced tools by default and reveal them only after authorization or a prerequisite action within the session.

```python
from fastmcp import FastMCP, Context
from fastmcp.server.auth import require_scopes

mcp = FastMCP("Enterprise Server")

# 1. Mount admin tools from a directory
admin_provider = FileSystemProvider("./admin_tools")
mcp.mount(admin_provider)

# 2. Hide them by default
mcp.disable(tags={"admin"})

# 3. Gatekeeper tool that unlocks admin tools for this session
@mcp.tool(auth=require_scopes("super-user"))
async def unlock_admin_mode(ctx: Context):
    """Authenticate and unlock administrative tools for this session.
    Requires super-user OAuth scope."""
    await ctx.enable_components(tags={"admin"})
    return "Admin mode unlocked. The following tools are now available: [list of admin tools]"
```

**How it works:**
1. The agent sees only safe, read-only tools initially
2. When a task requires admin capabilities, the agent calls `unlock_admin_mode`
3. The server verifies authorization and reveals the admin tools for this session only
4. Other sessions remain unaffected

**This pattern solves multiple problems:**
- **Security**: Destructive tools aren't visible until explicitly authenticated
- **Context efficiency**: The agent's initial tool list stays small
- **Progressive complexity**: Users start simple and escalate when needed

**Implementation via FastMCP 3.0:**
- `Visibility Transforms` control which components are visible per session
- `AuthMiddleware` gates groups of components by tag
- Session state tracks what each user has unlocked
- The unlock tool's response should list what was just made available

**Source:** FastMCP 3.0 blog (jlowin.dev/blog/fastmcp-3); deep research on progressive disclosure patterns
