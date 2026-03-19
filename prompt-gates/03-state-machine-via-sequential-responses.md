# Build State Machines via Sequential Tool Responses

Multi-step workflows don't need an agent framework. Each tool response sets up the next step — turning your MCP server into a lightweight state machine.

## The Pattern

```
Tool 1 (plan) response:
  <state>planning</state>
  <instructions>Prioritize 2024+ sources. Max 5 queries.</instructions>
  <next_tool>search</next_tool>

Tool 2 (search) response:
  <state>searching</state>
  <instructions>Cross-check facts across 2+ sources. Drop confidence < 0.6.</instructions>
  <next_tool>validate</next_tool>

Tool 3 (validate) response:
  <state>complete</state>
  <instructions>Present with confidence scores. Cite sources inline.</instructions>
```

## Why This Works

Each tool response is a **prompt gate** that:
1. Declares current state (so the model knows where it is)
2. Injects step-specific instructions (so behavior adapts per phase)
3. Points to the next tool (so the model doesn't wander)

The MCP server becomes a workflow orchestrator — "flattening the agent back into the model." You get deterministic multi-step behavior without any agent framework.

## Server-Side Helper

```typescript
function buildStateResponse(state: string, data: any,
    instructions: string, nextTool?: string): string {
  return [`<state>${state}</state>`,
    `<instructions>${instructions}</instructions>`,
    `<data>${JSON.stringify(data)}</data>`,
    nextTool ? `<next_tool>${nextTool}</next_tool>` : ""]
    .filter(Boolean).join("\n");
}
```

---

**Source:** u/Biggie_2018, r/mcp — "flattening the agent back into the model"; deep research synthesis.
