# Default to YAML Over JSON for LLM-Consumable Responses

JSON is the developer default, but YAML is the better default for MCP tool responses. It's more readable for LLMs and more token-efficient — no curly braces, no quotes on keys, no commas. The Figma Context MCP server defaults to YAML for exactly this reason.

**JSON response (47 tokens):**
```json
{
  "component": "Button",
  "props": {
    "variant": "primary",
    "size": "large",
    "disabled": false
  },
  "children": ["Submit"]
}
```

**YAML equivalent (~30% fewer tokens):**
```yaml
component: Button
props:
  variant: primary
  size: large
  disabled: false
children:
  - Submit
```

The model processes both equally well, but the YAML version burns fewer context tokens and leaves more room for reasoning.

**Offer a format parameter** so the agent can request JSON when it needs programmatic processing:

```typescript
import yaml from "js-yaml";

server.registerTool("get_design_tokens", {
  inputSchema: {
    file_key: z.string(),
    output_format: z.enum(["yaml", "json"]).default("yaml"),
  },
}, async ({ file_key, output_format }) => {
  const result = await fetchDesignTokens(file_key);
  const formatted = output_format === "json"
    ? JSON.stringify(result, null, 2)
    : yaml.dump(result, { lineWidth: -1 });

  return { content: [{ type: "text", text: formatted }] };
});
```

**When to stick with JSON:** When the response will be piped into another tool that expects JSON input, or when the agent explicitly requests it. Let the agent decide — just make YAML the default.

**Source:** GLips/Figma-Context-MCP (defaults to YAML for all design data responses); community findings on token efficiency of serialization formats.
