# Use Tool Annotations to Signal Safety Properties

MCP spec supports tool annotations that tell the agent and host about a tool's safety characteristics. Use them to enable automatic permission decisions.

```python
@tool(
    annotations={
        "readOnlyHint": True,     # This tool doesn't modify anything
        "destructiveHint": False,  # Not destructive
        "idempotentHint": True,    # Safe to retry
        "openWorldHint": False     # Only accesses known, scoped resources
    }
)
def search_contacts(query: str) -> list:
    """Search for contacts by name or email."""
    return db.search(query)

@tool(
    annotations={
        "readOnlyHint": False,
        "destructiveHint": True,   # This deletes data
        "idempotentHint": False,   # Cannot be undone
        "openWorldHint": False
    }
)
def delete_project(project_id: str) -> dict:
    """Permanently delete a project and all its data."""
    return api.delete(project_id)
```

**How agents use these annotations:**
- `readOnlyHint: true` → Agent can auto-approve without human confirmation
- `destructiveHint: true` → Agent should require explicit human approval
- `idempotentHint: true` → Agent knows it's safe to retry on failure
- `openWorldHint: true` → Agent knows the tool accesses external/unbounded resources

**Practical impact:** Agents that respect these annotations can auto-approve read operations while always pausing for confirmation on destructive ones. This creates a smooth UX for safe operations without compromising security.

**Note:** Not all clients respect annotations yet, but setting them is forward-compatible and documents intent even for human readers.

**Source:** [MCP specification — Tools](https://modelcontextprotocol.io/specification/2025-11-25/server/tools); [Anthropic — Writing effective tools for AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents)
