# Return Task IDs for Long-Running Operations

When a tool call will take more than a few seconds, return a task ID immediately and let the model poll for results. This prevents blocking the conversation and avoids timeouts.

```python
from fastmcp import FastMCP
from fastmcp.server.tasks import TaskConfig

mcp = FastMCP("Report Server")

@mcp.tool(task=True)  # Decorator-based API — runs as a background task
async def generate_report(data_id: str) -> dict:
    """Generate a comprehensive data report.
    Runs as a background task — returns a task ID immediately.
    Use the task status endpoint to poll for completion."""
    # Long-running work happens here; FastMCP handles the task lifecycle
    report = await build_report(data_id)
    return {
        "status": "completed",
        "result": report,
        "message": f"Report for '{data_id}' is ready."
    }

# For more control, use TaskConfig:
# @mcp.tool(task=TaskConfig(mode="required"))
# async def heavy_analysis(data_id: str) -> dict: ...
```

**Why this matters:**
- Default tool call timeouts are typically 30-60 seconds
- Long-running operations block the conversation
- The model can do other work while waiting
- Progress updates keep the user informed

**Implementation options:**
- FastMCP's `@mcp.tool(task=True)` decorator or `@mcp.tool(task=TaskConfig(mode="required"))` for fine-grained control
- Simple async queue with Redis/SQLite backing
- Thread pool for CPU-bound work

**Source:** [FastMCP tasks docs](https://gofastmcp.com/servers/tasks); [FastMCP 3.0 — What's New](https://www.jlowin.dev/blog/fastmcp-3-whats-new)
