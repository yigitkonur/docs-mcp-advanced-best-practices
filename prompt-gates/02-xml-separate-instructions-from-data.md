# Use XML Tags to Separate Instructions from Data

LLMs can distinguish behavioral instructions from raw data — but only when they're **structurally separated**. Mixing instructions inline with data dramatically reduces compliance. XML tags solve this cleanly.

## The Pattern

```xml
<instructions>
Analyze these results step-by-step.
If fewer than 3 relevant results, call search_broader tool.
Do not present results the user didn't ask for.
</instructions>
<data>
{"results": [
  {"title": "MCP Protocol Spec", "relevance": 0.95},
  {"title": "JSON-RPC Overview", "relevance": 0.72}
]}
</data>
<next_action>Call validate_relevance with top 3 results</next_action>
```

## Why This Works

The `<instructions>` block acts as a **prompt gate** — it shapes what the agent does next without cluttering the data payload. The model processes these as distinct semantic units:

- `<instructions>` → behavioral directives (what to do)
- `<data>` → factual content (what to work with)
- `<next_action>` → explicit next step (where to go)

## Anti-Pattern: Inline Instructions

```json
{
  "results": ["..."],
  "note": "Remember to validate these before showing them, and also call the verify tool next, and make sure to cite sources"
}
```

This fails because the model treats `note` as data metadata, not as a behavioral directive. Compliance drops significantly.

## MCP Server Helper

```python
def format_tool_response(data: dict, instructions: str, next_action: str = None) -> str:
    parts = [f"<instructions>\n{instructions}\n</instructions>",
             f"<data>\n{json.dumps(data, indent=2)}\n</data>"]
    if next_action:
        parts.append(f"<next_action>{next_action}</next_action>")
    return "\n".join(parts)
```

---

**Source:** Deep research synthesis; community patterns from MCP server implementations.
