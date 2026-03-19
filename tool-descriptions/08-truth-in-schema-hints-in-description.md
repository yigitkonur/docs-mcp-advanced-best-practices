# Truth in the Schema, Hints in the Description

The schema and description serve different roles. The schema carries **machine-enforceable truth**: types, required fields, enums, ranges. The description carries **context the schema cannot express**: side effects, auth scope, rate limits, latency, idempotency, and failure modes.

Don't duplicate what the schema already says. If your enum declares `["draft", "sent", "paid"]`, don't repeat those values in the description. Instead, use that space to tell the model what happens when it picks each one.

```json
{
  "name": "update_invoice_status",
  "description": "Transition an invoice to a new status. Changing to 'sent' triggers an email to the customer (side effect, not reversible). Changing to 'paid' requires payment_id. Rate limited to 10 calls/min per invoice.",
  "inputSchema": {
    "type": "object",
    "required": ["invoice_id", "status"],
    "properties": {
      "invoice_id": {
        "type": "string",
        "description": "UUID of the invoice"
      },
      "status": {
        "type": "string",
        "enum": ["draft", "sent", "paid", "void"]
      },
      "payment_id": {
        "type": "string",
        "description": "Required when status is 'paid'"
      }
    }
  }
}
```

The description adds three things the schema cannot:
1. **Side effect** — `sent` triggers an email
2. **Conditional requirement** — `payment_id` needed for `paid`
3. **Rate limit** — 10 calls/min per invoice

**Anti-pattern:** Descriptions that restate types ("invoice_id is a string") or list enum values already visible in the schema. This wastes tokens and creates drift risk when the schema changes but the description doesn't.

**Key principle:** Schema = what's valid. Description = what's wise.

**Source:** [u/GentoroAI on r/mcp](https://reddit.com/r/mcp/comments/1ooqeqy/)
