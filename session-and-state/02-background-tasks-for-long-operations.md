# Return Task IDs for Long-Running Operations

When a tool call will take more than a few seconds, return a task ID immediately and let the model poll for results. This prevents blocking the conversation and avoids timeouts.

```python
@mcp.tool
async def generate_report(ctx: Context, data_id: str) -> dict:
    """Generate a comprehensive data report. Returns immediately with a task ID.
    Use check_task_status(task_id) to poll for completion."""
    task_id = await ctx.start_background_task("report_worker", data_id=data_id)
    return {
        "status": "queued",
        "task_id": task_id,
        "message": (
            f"Report generation started for data '{data_id}'. "
            f"Use check_task_status(task_id='{task_id}') to check progress. "
            f"Typical completion time: 30-60 seconds."
        )
    }

@mcp.tool
async def check_task_status(task_id: str) -> dict:
    """Check the status of a background task."""
    task = await get_task(task_id)
    if task.status == "completed":
        return {
            "status": "completed",
            "result": task.result,
            "message": "Report is ready."
        }
    elif task.status == "failed":
        return {
            "content": [{"type": "text", "text": f"Task failed: {task.error}"}],
            "isError": True
        }
    else:
        return {
            "status": "in_progress",
            "progress": f"{task.percent_complete}%",
            "message": f"Still processing ({task.percent_complete}% complete). Check again in 10 seconds."
        }
```

**Why this matters:**
- Default tool call timeouts are typically 30-60 seconds
- Long-running operations block the conversation
- The model can do other work while waiting
- Progress updates keep the user informed

**Implementation options:**
- FastMCP's `ctx.start_background_task()` (SEP-1686 / Docket integration)
- Simple async queue with Redis/SQLite backing
- Thread pool for CPU-bound work

**Source:** FastMCP 3.0 blog - SEP-1686 and Docket integration; modelcontextprotocol.info best practices - AsyncTaskQueue pattern
