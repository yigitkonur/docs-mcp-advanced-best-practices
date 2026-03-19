# The Toolhost/Facade Pattern for Many Related Operations

When your MCP server exposes 20+ closely related operations — think a full admin API with list, get, create, update, delete, search, export, import across multiple entities — you hit the tool count ceiling. The Toolhost pattern solves this: one dispatcher tool that routes to internal handlers via an `operation` parameter. This is literally the Gang of Four Facade pattern applied to MCP.

Instead of polluting the tool list with dozens of entries, you expose a single entry point. The model picks the operation from a well-documented enum, and the server dispatches internally. Shared logic (auth, logging, error handling) lives in the facade, not duplicated across 25 tool definitions.

**Implementation:**
```python
OPERATIONS = {
    "list_users":    {"handler": list_users,    "args": ["filters", "page"]},
    "get_user":      {"handler": get_user,      "args": ["user_id"]},
    "create_user":   {"handler": create_user,   "args": ["name", "email", "role"]},
    "list_projects": {"handler": list_projects,  "args": ["owner_id", "status"]},
    "get_project":   {"handler": get_project,    "args": ["project_id"]},
    "export_data":   {"handler": export_data,    "args": ["entity", "format"]},
}

@tool(description=f"Admin API gateway. Operations: {', '.join(OPERATIONS.keys())}. Pass the operation name and its arguments.")
def admin_toolhost(
    operation: str,
    args: dict = {}
) -> dict:
    if operation not in OPERATIONS:
        return {"error": f"Unknown operation. Available: {list(OPERATIONS.keys())}"}

    op = OPERATIONS[operation]
    missing = [a for a in op["args"] if a not in args and a not in OPTIONAL_ARGS]
    if missing:
        return {"error": f"Missing required args for {operation}: {missing}"}

    try:
        result = op["handler"](**args)
        return {"operation": operation, "status": "success", "result": result}
    except Exception as e:
        return {"operation": operation, "status": "error", "error": str(e)}
```

**When to use the Toolhost pattern:**
- 20+ operations that share common auth/validation/error-handling code
- Operations are closely related (same domain, same API backend)
- You want a single place to add logging, rate limiting, or retry logic

**When to avoid it:**
- Fewer than 5 operations — the indirection adds complexity for no benefit
- Operations need individual user-approval flows (the facade hides which operation runs)
- Operations have radically different parameter shapes that don't fit a generic `args` dict

**Why it matters:** The Toolhost pattern lets you scale to 50+ operations without hitting model confusion limits. The model only needs to understand one tool signature. The tradeoff is discoverability — the model must know which operations exist, so your description string must be comprehensive.

**Source:** glassBead, "Design Patterns in MCP: Toolhost Pattern"; u/glassBeadCheney, r/mcp
