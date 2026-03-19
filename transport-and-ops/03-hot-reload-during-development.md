# Use Hot Reload During Development

Kill-restart cycles during MCP server development are slow and break your flow. FastMCP supports automatic file watching and server restart.

```bash
# Auto-restarts on file changes
fastmcp dev server.py
```

**What this gives you:**
- File watcher detects changes to your server code
- Server automatically restarts with new code
- No manual kill + restart cycle
- MCP Inspector connects at the same URL

**Development workflow:**
```
1. Start:     fastmcp dev server.py
2. Open:      MCP Inspector at http://localhost:5173
3. Edit:      Modify tool descriptions, add parameters, fix bugs
4. Automatic: Server restarts, Inspector reconnects
5. Test:      Call tools in Inspector to verify changes
6. Repeat
```

**Additional development tools:**
- `fastmcp dev` - Development server with hot reload
- MCP Inspector (`npx @modelcontextprotocol/inspector@latest`) - GUI for testing tool calls
- `claude mcp add <name> <command>` - Quick local registration in Claude Code

**Pro tip:** Keep the Inspector open while iterating on tool descriptions. You can immediately see how schema changes affect the tool discovery payload.

**In FastMCP 3.0:** Tool decorators now return the original Python function, so you can:
```python
@tool
def add(a: int, b: int) -> int:
    return a + b

# Direct call in tests (no MCP overhead)
result = add(2, 3)

# Remote call via MCP
await client.call_tool("add", {"a": 2, "b": 3})
```

**Source:** FastMCP 3.0 blog (jlowin.dev/blog/fastmcp-3); NearForm - "Implementing MCP: Tips, tricks and pitfalls"
