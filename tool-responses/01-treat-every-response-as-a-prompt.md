# Treat Every Tool Response as a Prompt to the Model

This is the single most important MCP design insight. Stop treating MCP servers like APIs with better descriptions. Every tool response is injected directly into the model's context and becomes part of its reasoning input.

A raw API response:
```json
{"members": [{"id": "u1", "name": "Jane"}, {"id": "u2", "name": "Bob"}], "total": 25}
```

An MCP-optimized response:
```json
{
  "members": [{"name": "Jane Doe", "activity_score": 92}, {"name": "Bob Smith", "activity_score": 45}],
  "total": 25,
  "summary": "Found 25 active members in the last 30 days. Top contributor: Jane Doe.",
  "next_steps": "Use bulkMessage() to contact these members, or filter by activity_score to focus on top contributors."
}
```

**The key insight:** Developers read docs, experiment, and remember. AI models start fresh every conversation with only tool descriptions to guide them - until they start calling tools. Then tool responses become the primary steering mechanism.

**What to include in responses:**
- A human-readable summary of the result
- Suggested next actions with specific tool names
- Context that helps the model interpret the data
- Pagination hints when results are truncated

**What to avoid:**
- Raw JSON dumps with no context
- Internal IDs without human-readable labels
- Technical metadata the model can't reason about

**Source:** u/sjoti, r/mcp (279 upvotes, 40 comments). One commenter called this "HATEOAS at the language level." Another: "I ended up instructing the model with every MCP response with amazing results."
