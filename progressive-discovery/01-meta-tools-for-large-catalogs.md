# Use Meta-Tools for Large Tool Catalogs (list/describe/execute)

When your MCP server has more than ~40 tools, loading all their schemas into the prompt becomes prohibitively expensive. Replace static tool exposure with three meta-tools that let the agent discover tools on demand.

**The three meta-tools:**
```python
@tool
def list_tools(prefix: str = "/") -> list[str]:
    """List available tool categories or tools matching a prefix.
    Example: list_tools('/hubspot/deals/') returns deal-related tools."""
    return tool_registry.list(prefix)

@tool
def describe_tools(tool_id: str) -> dict:
    """Get the full input schema for a specific tool.
    Call this before execute_tool to understand required parameters."""
    return tool_registry.get_schema(tool_id)

@tool
def execute_tool(tool_id: str, arguments: dict) -> dict:
    """Execute a tool by ID with the given arguments.
    Call describe_tools first to get the correct schema."""
    return tool_registry.execute(tool_id, arguments)
```

**Token impact (measured with Claude Sonnet 4):**

| Strategy | 40 tools | 100 tools | 200 tools | 400 tools |
|----------|----------|-----------|-----------|-----------|
| Static (all schemas in prompt) | 43,300 tokens | 128,900 | 261,700 | 405,100 |
| Progressive (meta-tools) | 1,600 tokens | 2,400 | 2,500 | 2,500 |

**Critical design details:**
- **Separate schema retrieval from discovery.** `list_tools` returns only names; `describe_tools` returns the full schema. This prevents sending 100 schemas when the model only needs one.
- **Use hierarchical prefixes** in tool IDs: `/hubspot/deals/create`, `/hubspot/contacts/search`. This makes `list_tools` queries precise.
- The initial context contains only ~1.5-2.5k tokens regardless of total tool count.

**Anti-pattern:** Loading all tool schemas statically into the prompt. Beyond ~200 tools, this exceeds Claude's context window and tasks cannot complete at all.

**Source:** [Speakeasy — Comparing Progressive Discovery and Semantic Search](https://www.speakeasy.com/blog/100x-token-reduction-dynamic-toolsets)
