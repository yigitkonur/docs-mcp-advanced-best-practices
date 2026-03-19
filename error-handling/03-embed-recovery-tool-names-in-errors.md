# Embed Recovery Tool Names Directly in Error Messages

When a tool call fails, don't just say what went wrong - tell the model exactly which tool to call next to fix it.

```json
{
  "content": [{
    "type": "text",
    "text": "You can't terminate an instance while it is running. Call stop_instance(instance_id='i-abc123') first, then retry terminate_instance()."
  }],
  "isError": true
}
```

**Pattern: State-change prerequisite errors**
```python
@tool
def terminate_instance(instance_id: str):
    instance = get_instance(instance_id)
    if instance.state == "running":
        return {
            "content": [{
                "type": "text",
                "text": (
                    f"Instance '{instance_id}' is currently running. "
                    f"Call stop_instance(instance_id='{instance_id}') first, "
                    f"wait for it to reach 'stopped' state, then retry "
                    f"terminate_instance(instance_id='{instance_id}')."
                )
            }],
            "isError": True
        }
    # ... proceed with termination
```

**Pattern: Validation with corrected suggestions**
```json
{
  "content": [{
    "type": "text",
    "text": "The requested travel date cannot be in the past. You asked for July 31 2024 but today is July 25 2025. Did you mean July 31 2025?"
  }],
  "isError": true
}
```

**Key principle:** Include the actual parameter values the model should use, not just the tool name. Don't make the model figure out what arguments to pass - spell them out.

**Source:** alpic.ai/blog - "Better MCP tool call error responses: help your AI recover gracefully"
