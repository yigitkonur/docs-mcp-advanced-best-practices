# Use Resources as On-Demand Documentation Referenced in Errors

Expose your tool documentation as MCP resources, then reference them in error messages. The model fetches the docs only when it needs help - keeping the initial context lean.

```python
@mcp.resource("docs://tools")
def get_tool_docs() -> str:
    """Complete documentation for all tools, including examples and edge cases."""
    return Path("tool_documentation.md").read_text()

@mcp.tool(description="Search for contacts. See docs://tools for detailed usage.")
def search_contacts(query: str) -> dict:
    if not query.strip():
        return {
            "content": [{
                "type": "text",
                "text": "Query cannot be empty. See docs://tools for usage examples and valid query formats."
            }],
            "isError": True
        }
    # ...
```

**How this plays out:**
1. Initial context has only brief tool descriptions
2. Model calls tool correctly 90% of the time with just the description
3. On failure, the error message references `docs://tools`
4. The model (or client) fetches the resource to get detailed guidance
5. Armed with full docs, the model retries successfully

**Real-world validation:** One practitioner (u/kiedi5 r/mcp) put tool documentation in a markdown file exposed as a resource, then added "see docs://tools for more information" to error messages. "It seems to work really well and LLMs use the tools correctly more often now."

**Important nuance:** Not all clients automatically fetch resources. Some (like Goose) do; others may need the model to explicitly request it. Test with your target client.

**Source:** [u/kiedi5 on r/mcp](https://reddit.com/r/mcp) — "Does anyone use MCP prompts or resources?" thread; u/emicklei confirmed the pattern works with syntax error recovery
