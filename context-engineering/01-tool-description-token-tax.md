# Tool Descriptions Eat ~15% of Your Context Budget

Every MCP tool you register injects its full definition — name, description, and input schema — into the system prompt on **every single message**, not only when the tool is invoked. Community measurement in r/ClaudeAI puts MCP tool definitions at roughly 15% of Claude Code's total input tokens in a typical session. A single complex tool like Playwright MCP can push that share to 50% by itself, before you've said a word.

The math compounds fast: each tool definition costs 50–150 tokens depending on schema complexity. Twenty tools at 100 tokens each is 2,000 tokens per message. Across a 30-turn conversation that's 60,000 tokens gone to definitions alone — context you could have used for code, logs, or reasoning. Gemini models are even tighter; keep descriptions under 50–75 tokens there.

Claude Code v1.0.86+ ships a `/context` command that breaks down token usage by source. For raw traffic inspection, `claude-code-proxy` (github.com/seifghazi/claude-code-proxy) intercepts all requests and logs the full payload so you can measure exactly what your tool roster costs.

```typescript
// ❌ Verbose — 140+ tokens for this one tool
server.tool(
  "search_documents",
  {
    description: "Searches through all documents in the repository using full-text search. " +
      "Supports boolean operators, phrase matching, and field-specific queries. " +
      "Returns paginated results with relevance scores and highlighted snippets.",
    inputSchema: { /* ... */ }
  }
);

// ✅ Trim to under 100 tokens — same capability, fraction of the cost
server.tool(
  "search_documents",
  {
    description: "Full-text search across repo docs. Supports boolean, phrase, field queries. Returns paginated results.",
    inputSchema: { /* ... */ }
  }
);
```

```typescript
// Audit your tool token budget before registering
function estimateToolTokens(tools: Tool[]): void {
  for (const tool of tools) {
    const raw = JSON.stringify(tool);
    const estimate = Math.ceil(raw.length / 4); // rough chars-to-tokens
    if (estimate > 100) {
      console.warn(`[token-audit] ${tool.name}: ~${estimate} tokens — consider trimming`);
    }
  }
}
```

**Why it matters:** Tool descriptions are a hidden fixed cost on every turn. Trimming them is one of the highest-ROI context optimisations you can make — it requires no architecture changes and compounds across the entire conversation.

**Source:** u/TheNickmaster21 and u/violet_mango, r/ClaudeAI community measurements; `claude-code-proxy` by github.com/seifghazi.
