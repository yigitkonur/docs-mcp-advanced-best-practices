# CRUD: Combined Tool vs Separate Tools Decision

When you have multiple entity types each needing create/read/update/delete, you face a design choice: expose `create_user`, `read_user`, `update_user`, `delete_user` (×N entity types) — or combine them into `manage_user(action: "create|read|update|delete")`. The right answer depends on your tool count and approval requirements.

**Combine when:** you have many entity types (users, projects, teams, billing). Four CRUD operations × 8 entities = 32 tools. Most models degrade with 20+ tools in the prompt — they confuse similar tool names and pick the wrong one. Combining gives you 8 tools instead of 32, with the `action` enum guiding the model.

**Keep separate when:** you have few entity types (≤3), you need per-operation user approval (e.g., reads are auto-approved but deletes require confirmation), or your tools have very different parameter shapes per operation.

**Combined pattern:**
```python
@tool(description="Manage users. Actions: 'list' (filter by role/status), 'get' (by user_id), 'create' (name+email required), 'update' (user_id + fields to change), 'delete' (by user_id, irreversible).")
def manage_user(
    action: Literal["list", "get", "create", "update", "delete"],
    user_id: str | None = None,
    filters: dict | None = None,
    data: dict | None = None
) -> dict:
    match action:
        case "list":
            return {"users": db.users.find(filters or {})}
        case "get":
            return {"user": db.users.find_one(user_id)}
        case "create":
            return {"user": db.users.insert(data), "status": "created"}
        case "update":
            db.users.update(user_id, data)
            return {"user": db.users.find_one(user_id), "status": "updated"}
        case "delete":
            db.users.delete(user_id)
            return {"status": "deleted", "user_id": user_id}
```

**Decision matrix:**

| Factor | Combine | Separate |
|---|---|---|
| Entity types | >3 | ≤3 |
| Total tool count | Approaching 20+ | Under 15 |
| Approval granularity | Same for all ops | Different per operation |
| Parameter overlap | High | Low |

**Why it matters:** Tool count directly impacts model accuracy. With 30+ tools, models frequently pick the wrong tool or hallucinate parameters. The combined pattern keeps tool count manageable while preserving full CRUD functionality. But don't over-consolidate — if a user needs to approve deletes but not reads, separate tools give you that control.

**Source:** Reddit community consensus, r/mcp; Workato MCP documentation — "CRUD tool design patterns"
