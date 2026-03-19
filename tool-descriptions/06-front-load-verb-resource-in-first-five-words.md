# Front-Load Verb + Resource in the First Five Words

LLMs skim tool descriptions the same way humans skim headlines — the first few words carry disproportionate weight in tool selection. If your description starts with filler ("This tool is used to…"), the model has already wasted its attention budget before reaching the actual signal.

Structure every description as: **Verb + Resource + key scope**, then stop. Keep total descriptions under ~100 tokens. Past that, you're spending context window on tool selection instead of actual task work.

```json
{
  "name": "search_customers",
  "description": "Search customers by name, email, or account ID. Returns top 20 matches with account status. Use list_customers for unfiltered pagination."
}
```

```json
{
  "name": "create_invoice",
  "description": "Create a draft invoice for a customer. Requires customer_id and at least one line item. Does not send — use send_invoice to deliver."
}
```

**Anti-pattern:**

```json
{
  "name": "search_customers",
  "description": "This tool provides the ability to search through the customer database using various criteria including but not limited to name, email address, and account identifier. It will return a paginated list of results sorted by relevance score with additional metadata about each customer's current account status and tier level."
}
```

The anti-pattern is 50+ tokens before the model learns what the tool actually does. The first version communicates the same information in half the space.

**Key principle:** First 5 words → selection. Next 20 words → parameters. Everything after that → diminishing returns. Budget accordingly.

**Source:** Community best practices; Anthropic prompt engineering guidance on concise tool descriptions
