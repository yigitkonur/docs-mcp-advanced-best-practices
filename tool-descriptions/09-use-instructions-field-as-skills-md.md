# Use the `instructions` Field as Your Server's skills.md

The MCP `initialize` response includes an `instructions` field — a free-form string that clients surface to the model as system-level context. This is the most reliable place to explain your server's overall capabilities, recommended workflows, and inter-tool relationships.

Unlike the spec's `prompts` feature (which many clients silently ignore), `instructions` is consistently read by Claude Desktop, Cursor, and other major MCP clients.

```typescript
const server = new McpServer({
  name: "acme-crm",
  version: "1.0.0",
  instructions: `
# Acme CRM MCP Server

## Available Capabilities
- Customer management (CRUD + search)
- Invoice lifecycle (create → send → pay → void)
- Activity log queries

## Recommended Workflows
1. **Look up a customer** → search_customers → get_customer
2. **Send an invoice** → create_invoice → add_line_items → send_invoice
3. **Check payment status** → get_invoice → get_payment

## Important Constraints
- All write operations require a valid session (call authenticate first)
- Invoice sends are irreversible and trigger customer emails
- Rate limit: 60 requests/min across all tools
  `.trim()
});
```

Treat this like a `skills.md` or `AGENTS.md` — a briefing document the agent reads once at session start to understand the full landscape before making its first tool call.

**Why it matters:** Individual tool descriptions tell the model what each tool does. The `instructions` field tells the model how the tools fit together. Without it, agents discover workflows by trial and error — burning tokens and making avoidable mistakes.

**Source:** u/hasmcp, r/mcp; MotherDuck blog (returns `mcp_server_instructions.md` with full usage guidance via this field)
