# Make Error Messages Educational, Not Technical

An error message IS the documentation at that moment. The model has no other source of truth to figure out what went wrong.

**Bad (real example from Supabase MCP):**
```json
{"error": "Unauthorized"}
```
This stops the model cold. It thinks it lacks permissions and gives up.

**Good:**
```json
{
  "error": "Project ID 'proj_abc123' not found or you lack permissions. To see available projects, use the listProjects() tool.",
  "isError": true
}
```

**Error message checklist:**
1. **Name the specific field** that caused the failure
2. **State the expected format** or valid values
3. **Show an example** of correct input
4. **Suggest a recovery action** with a specific tool name

**Template:**
```python
def format_error(field: str, problem: str, expected: str, recovery_tool: str = None) -> dict:
    msg = f"Parameter '{field}': {problem}. Expected: {expected}."
    if recovery_tool:
        msg += f" Use {recovery_tool} to get valid values."
    return {"content": [{"type": "text", "text": msg}], "isError": True}

# Example usage:
format_error(
    field="start_date",
    problem="Date '2024-07-31' is in the past",
    expected="A future date in YYYY-MM-DD format",
    recovery_tool=None
)
# Output: "Parameter 'start_date': Date '2024-07-31' is in the past. Expected: A future date in YYYY-MM-DD format."
```

**Why it matters:** If a model gets the tool call right 90%+ of the time, and can self-correct from a good error message the other 10%, you don't need exhaustive descriptions covering every edge case upfront. Errors handle the long tail.

**Source:** u/sjoti r/mcp (279 upvotes) - "The Supabase MCP... its response is `{"error": "Unauthorized"}`, which is technically correct but completely unhelpful."
