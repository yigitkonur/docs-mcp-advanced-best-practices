# Prepend Truncation Guidance When Results Are Cut Off

When you paginate or truncate results, don't just silently return partial data. Tell the model what happened and how to get more.

```python
@tool
def search_logs(query: str, limit: int = 50) -> dict:
    results = log_store.search(query, limit=limit + 1)
    truncated = len(results) > limit
    results = results[:limit]

    response = {
        "results": results,
        "count": len(results),
        "total_available": log_store.count(query),
    }

    if truncated:
        response["guidance"] = (
            f"Showing {limit} of {response['total_available']} results. "
            f"Consider filtering by date range (e.g., start_date='2025-01-01') "
            f"or adding more specific terms to narrow results."
        )

    return response
```

**Key details:**
- Place guidance **before** the data in the response so the model reads it first
- Suggest specific parameter values that would reduce the result set
- Include the total count so the model can decide if narrowing is worthwhile
- Claude Code caps tool responses at 25,000 tokens - plan for this limit

**Anti-pattern:** Silently returning the first N results with no indication that more exist. The model will assume the returned data is complete and draw incorrect conclusions.

**Source:** [Anthropic — Writing effective tools for AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents); [modelcontextprotocol.io — writing effective tools](https://modelcontextprotocol.io)
