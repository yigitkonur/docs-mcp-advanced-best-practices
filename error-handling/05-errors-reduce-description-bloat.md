# Let Errors Handle the Long Tail Instead of Bloating Descriptions

If a model gets the tool call right 90%+ of the time, you don't need to cram every edge case into the tool description. Let good error messages handle the remaining 10%.

**The trade-off:**
- **Exhaustive descriptions** cover every case upfront but waste tokens on every call, even when the model would have gotten it right
- **Minimal descriptions + rich errors** keep the initial context lean and only inject detailed guidance when needed

**In practice:**
```python
@tool(description="Search for contacts by name, email, or company.")
def search_contacts(query: str, limit: int = 10):
    """
    Description is intentionally brief. Edge cases are handled by errors.
    """
    if not query.strip():
        return error("Query cannot be empty. Provide a name, email, or company name.")

    if limit > 100:
        return error(
            f"Limit {limit} exceeds maximum of 100. "
            f"Call search_contacts(query='{query}', limit=100) instead."
        )

    results = db.search(query, limit=limit)
    if not results:
        return {
            "content": [{"type": "text", "text":
                f"No contacts found for '{query}'. "
                f"Try a broader search term or check spelling. "
                f"You can also use list_companies() to browse available companies."
            }],
            "isError": False  # Not an error - valid empty result with guidance
        }
    return format_results(results)
```

**Key insight:** This is especially true for complex APIs with many endpoints. Stuffing documentation into descriptions leads to context overload (30+ detailed tools = performance degradation). Error-driven learning keeps the initial footprint small.

**Caveat:** Don't take this too far. Core usage patterns should be in the description. Only edge cases and rare failure modes belong in error messages.

**Source:** [u/sjoti on r/mcp](https://reddit.com/r/mcp/comments/1lq69b3/); [u/Nako_A1 on r/mcp](https://reddit.com/r/mcp/comments/1npfoo9/) — error-driven learning (solution #6)
