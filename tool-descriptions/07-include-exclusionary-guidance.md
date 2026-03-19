# Include Exclusionary Guidance — Tell the Model When NOT to Use a Tool

Positive descriptions ("Use this to…") are necessary but insufficient. When your server exposes multiple tools with overlapping domains, the model needs explicit negative routing to avoid misselection.

Add a short exclusionary line at the end of each description. This is especially critical when tools share the same resource type but differ in scope, cost, or side effects.

```json
{
  "name": "get_customer",
  "description": "Fetch a single customer by ID. Returns full profile with contact info and billing history. Do NOT use for searching — use search_customers instead."
}
```

```json
{
  "name": "search_customers",
  "description": "Search customers by name or email. Returns top 20 matches. Do NOT use for bulk export; use export_customers for datasets over 100 records."
}
```

```json
{
  "name": "export_customers",
  "description": "Export all customers matching a filter as CSV. Async — returns a job ID. Do NOT use for single lookups; use get_customer instead."
}
```

Without exclusionary hints, models tend to default to the "biggest" tool — the one that could technically handle every case. This leads to `export_customers` being called when `get_customer` would suffice, wasting time and resources.

**Real-world example:** Figma's MCP server uses `"Do NOT use unless explicitly requested by the user"` on its `depth` parameter to prevent agents from making expensive deep-tree calls by default. The negative signal is what keeps the model from over-fetching.

**Why it matters:** Exclusionary guidance acts as a routing table. Without it, tools with overlapping scope create ambiguity, and ambiguity means the model guesses — often wrong.

**Source:** u/Ok-Birthday-5406, r/mcp; Figma MCP server parameter descriptions
