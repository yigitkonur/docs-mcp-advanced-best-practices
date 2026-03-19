# Graceful Fallback for Hidden Tools

When you hide a tool via progressive disclosure, a stateful LLM may still "remember" that tool from earlier in the conversation and attempt to call it. Handle this gracefully rather than returning a generic not-found error.

The DarkMatter MCP project coined this the "DarkMatter" problem: tools that exist but are currently invisible can cause agent confusion if the model has prior knowledge of them.

## Detection and Recovery Pattern

```typescript
// Registry of all tools (including currently hidden ones)
const allTools = new Map<string, ToolDefinition>();
const visibleTools = new Set<string>();

// Override the default "tool not found" behavior
server.setToolNotFoundHandler(async (toolName: string, params: unknown) => {
  if (allTools.has(toolName)) {
    // Tool exists but is hidden — explain and offer the path to unlock it
    const tool = allTools.get(toolName)!;
    return {
      content: [{
        type: "text",
        text: JSON.stringify({
          error: "TOOL_NOT_CURRENTLY_VISIBLE",
          tool_name: toolName,
          message: `"${toolName}" exists but is not active in your current session mode.`,
          how_to_unlock: tool.unlockVia
            ? `Call ${tool.unlockVia} first to activate this tool.`
            : "Contact your administrator to enable this capability.",
          current_alternatives: visibleTools.has("search_tools")
            ? [`Call search_tools to discover available alternatives to ${toolName}.`]
            : [],
        })
      }],
      isError: true,
    };
  }

  // Truly unknown tool
  return {
    content: [{ type: "text", text: `Unknown tool: "${toolName}". Call list_tools to see available tools.` }],
    isError: true,
  };
});
```

## On-Demand Restoration

If the model insists on a hidden tool, you can restore it for the session:

```typescript
server.tool("restore_tool", "Restore a temporarily hidden tool for this session", {
  tool_name: z.string(),
  reason: z.string().describe("Why this tool is needed now"),
}, async ({ tool_name, reason }) => {
  if (!allTools.has(tool_name)) {
    return { content: [{ type: "text", text: `"${tool_name}" does not exist.` }], isError: true };
  }
  visibleTools.add(tool_name);
  await server.notifyToolsChanged();
  return { content: [{ type: "text", text: `"${tool_name}" restored for this session.` }] };
});
```

**Why it matters:** Without graceful fallback, a model that remembers a hidden tool will retry it repeatedly, wasting context on uninformative error messages. The TOOL_NOT_CURRENTLY_VISIBLE response guides recovery.

**Source:** DarkMatter MCP project (github.com/darkmatter-mcp); dynamic-tools-patterns research file §2; r/mcp discussion on tool visibility edge cases.
