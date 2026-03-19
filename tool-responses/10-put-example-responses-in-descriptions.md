# Put Example Responses in Tool Descriptions

Include a concrete example response directly in your tool's description. The agent learns the response shape from the example and can plan subsequent tool calls without making a throwaway exploratory call first. One example in the description saves one round-trip at runtime.

**Without an example — the agent guesses or wastes a call:**
```typescript
server.registerTool("list_users", {
  description: "List users in the workspace",
  inputSchema: { role: z.string().optional() },
}, handler);
```

The agent doesn't know if it gets `{users: [...]}` or `[...]` or `{data: {items: [...]}}` until it calls the tool.

**With an example — the agent knows what to expect:**
```typescript
server.registerTool("list_users", {
  description: `List users in the workspace.

Returns a JSON object with:
- "users": array of {id, name, email, role}
- "total": total count (may exceed returned results)
- "has_more": whether more pages exist

Example response:
{
  "users": [
    {"id": "u_abc1", "name": "Jane Doe", "email": "jane@co.com", "role": "admin"},
    {"id": "u_abc2", "name": "Bob Smith", "email": "bob@co.com", "role": "member"}
  ],
  "total": 47,
  "has_more": true
}

Use "page" param to paginate. Pass user "id" values to get_user_details.`,
  inputSchema: {
    role: z.string().optional().describe("Filter by role: admin, member, viewer"),
    page: z.number().optional().describe("Page number, starts at 1"),
  },
}, handler);
```

**What this unlocks:**
- The agent can write code that destructures the response correctly on the first try
- It knows `has_more` exists and will paginate automatically
- It knows to pass `id` (not `name` or `email`) to downstream tools
- It won't make a dummy call just to discover the response format

**Keep examples minimal** — one or two records, not a full page of results. The agent extrapolates the pattern. A 3-line example teaches the same shape as a 50-line one but costs far fewer description tokens.

**This is not pretty, but it works.** The description gets long, and purists will object. But in practice, agents that see example responses make fewer errors and fewer exploratory calls. The token cost of a slightly longer description is far less than the cost of a wasted tool round-trip.

**Source:** [u/gauthierpia on r/AI_Agents](https://reddit.com/r/AI_Agents) — reported significant reduction in wasted tool calls after adding example responses to descriptions
