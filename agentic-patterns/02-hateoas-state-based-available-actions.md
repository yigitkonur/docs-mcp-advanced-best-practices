# HATEOAS: State-Based Available Actions

Include a dynamic `_available_actions` field in every tool response, computed from the current resource state. The LLM learns what it can do next by reading the response — not by consulting documentation or guessing.

This is the HATEOAS (Hypermedia as the Engine of Application State) pattern adapted for LLM tool responses. Instead of hyperlinks, you return structured action descriptors that describe the next available tool calls.

## Example: Deployment Status Response

```typescript
type Action = { tool: string; description: string; required_params?: string[] };

function getAvailableActions(deployment: Deployment): Action[] {
  if (deployment.status === "running") {
    return [
      { tool: "get_metrics", description: "View live metrics", required_params: ["deployment_id"] },
      { tool: "rollback_deployment", description: "Roll back to previous version", required_params: ["deployment_id"] },
      { tool: "scale_deployment", description: "Adjust instance count", required_params: ["deployment_id", "replicas"] },
    ];
  }
  if (deployment.status === "failed") {
    return [
      { tool: "get_logs", description: "Inspect failure logs", required_params: ["deployment_id"] },
      { tool: "rollback_deployment", description: "Restore last known good version", required_params: ["deployment_id"] },
    ];
  }
  if (deployment.status === "stopped") {
    return [
      { tool: "start_deployment", description: "Start the service", required_params: ["deployment_id"] },
    ];
  }
  return [];
}

server.tool("get_deployment", "Get deployment status", {
  deployment_id: z.string(),
}, async ({ deployment_id }) => {
  const deployment = await fetchDeployment(deployment_id);
  return {
    content: [{
      type: "text",
      text: JSON.stringify({
        ...deployment,
        _available_actions: getAvailableActions(deployment),
      }, null, 2)
    }]
  };
});
```

## Benefits Over Static Documentation

- Actions are **always accurate** — they reflect actual current state, not theoretical capabilities
- The agent never attempts an invalid state transition (e.g., scaling a stopped deployment)
- Reduces need for the agent to call a "what can I do?" discovery tool
- Works without any system-prompt context about the workflow

**Why it matters:** Without HATEOAS, agents frequently call tools in invalid states and must parse error messages to understand what went wrong. With it, valid transitions are self-evident from the data.

**Source:** REST HATEOAS principle adapted for LLM tools; [Nordic APIs — "HATEOAS: The API Design Style That Was Waiting for AI"](https://nordicapis.com/hateoas-the-api-design-style-that-was-waiting-for-ai/); community discussions on [r/mcp](https://reddit.com/r/mcp)
