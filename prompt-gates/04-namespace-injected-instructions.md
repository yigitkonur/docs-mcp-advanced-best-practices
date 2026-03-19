# Namespace Injected Instructions to Prevent Conflicts

When multiple tools inject instructions, they can contradict each other. Namespacing solves this — and prevents workflow hijacking.

## The Pattern

```json
{
  "result": {"data": "..."},
  "_agent_guidance": {
    "source": "search_tool",
    "instructions": "Cross-reference these results before presenting to the user.",
    "next_action": {
      "tool": "validate",
      "required": true
    },
    "allowed_next_tools": ["validate", "summarize", "export"]
  }
}
```

## Why Namespace

1. **Attribution** — `source` tells the model (and debuggers) which tool issued the instruction
2. **Conflict resolution** — When two tools disagree, the model can weigh by source
3. **Security** — `allowed_next_tools` prevents a compromised tool from redirecting the agent to unintended tools

## Preventing Workflow Hijacking

Without whitelisting, a compromised tool can redirect the agent arbitrarily. Defend with a server-side tool graph:

```python
TOOL_GRAPH = {
    "search":   {"allowed_next": ["validate", "search_broader"]},
    "validate": {"allowed_next": ["summarize", "search"]},
    "summarize": {"allowed_next": ["export", "refine"]},
}

def validate_next_tool(current_tool: str, requested_next: str) -> bool:
    allowed = TOOL_GRAPH.get(current_tool, {}).get("allowed_next", [])
    if requested_next not in allowed:
        raise ValueError(f"'{current_tool}' cannot invoke '{requested_next}'. Allowed: {allowed}")
    return True
```

Every tool response should include `_agent_guidance` with a consistent shape. The model learns the pattern quickly and follows it reliably.

---

**Source:** Deep research synthesis; security considerations from prompt injection literature and MCP server hardening.
