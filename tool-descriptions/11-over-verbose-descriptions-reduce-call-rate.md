# Over-Verbose Descriptions Reduce Tool Call Rate

Tool descriptions over ~200 tokens cause LLMs to call the tool less often, not more. Community evals show a **30–50% reduction in tool invocation rate** when descriptions exceed this threshold — even when the tool is the correct one to use.

The hypothesis: over-long descriptions compete with the model's generative attention. When a description is long enough to read like a mini-essay, the model sometimes "answers" the question using the description's text instead of calling the tool.

## The 200-Token Rule

```typescript
// Measure description token cost before registering
function registerWithLengthCheck(name: string, description: string, schema: z.ZodObject<any>, handler: Function) {
  const tokens = estimateTokens(description);
  if (tokens > 200) {
    console.warn(`⚠️  Tool "${name}" description is ${tokens} tokens (limit: 200). Consider trimming.`);
  }
  server.tool(name, description, schema, handler);
}
```

## Optimal Format: Verb + Object + Key Constraint

```
// ❌ Too long (320 tokens):
"This tool allows you to search through the repository's codebase using semantic or
keyword-based queries. It supports both exact string matching and fuzzy semantic
search using vector embeddings. The results will include file paths, line numbers,
and surrounding context. Use this when you need to find where a particular function,
class, variable, or concept appears in the codebase. You can filter by file type,
directory, or date range. Results are ranked by relevance..."

// ✅ Optimal (~35 tokens):
"Search codebase by keyword or semantic query. Returns file paths, line numbers, and
snippets. Use for finding functions, variables, or concepts. Filter by file type or
directory with optional params."
```

## What Belongs in Schema, Not Description

Move parameter-level guidance into the `description` field of individual parameters rather than the tool description. The model reads parameter descriptions when filling in arguments — that's exactly when you want parameter guidance to appear.

```typescript
{
  query: z.string().describe("Search term. Supports regex if prefixed with 're:'"),
  file_type: z.string().optional().describe("Filter by extension, e.g. '.ts' or '.py'"),
}
```

**Why it matters:** A 400-token description costs more in tokens AND gets called less often. The optimal tool description is short enough to scan and specific enough to select.

**Source:** Community eval results shared in r/mcp (u/Rotemy-x10, 271-upvote thread on tool selection accuracy); Anthropic prompt engineering guide on tool use.
