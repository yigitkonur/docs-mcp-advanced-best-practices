# Use `z.coerce` and `z.preprocess` for Type Safety

LLMs frequently send `"3"` instead of `3`, or `"false"` instead of `false`. Your server crashes with a Zod validation error, the LLM sees a cryptic message, and the user's request fails silently.

**Fix it at the schema level.** Use `z.coerce.number()` for numeric params and `z.preprocess()` for booleans.

## Coerced Number

```typescript
// ❌ Breaks when LLM sends "3"
count: z.number().describe("Number of results")

// ✅ Handles "3", 3, "3.5" gracefully
count: z.coerce.number().describe("Number of results")
```

## Coerced Boolean

The Sequential Thinking server uses this pattern to handle every boolean variant an LLM might send:

```typescript
const coercedBoolean = z.preprocess((val) => {
  if (typeof val === "boolean") return val;
  if (typeof val === "string") {
    if (val.toLowerCase() === "true") return true;
    if (val.toLowerCase() === "false") return false;
  }
  return val;
}, z.boolean());
```

Then use it in your tool schemas:

```typescript
server.tool("search", {
  query: z.string(),
  includeArchived: coercedBoolean.optional().default(false)
    .describe("Include archived results"),
}, async ({ query, includeArchived }) => {
  // includeArchived is always a real boolean here
});
```

## Why This Matters

- LLMs serialize all tool arguments as JSON — type coercion bugs are the #1 cause of silent tool failures
- `z.coerce` is zero-cost at runtime and eliminates an entire class of errors
- The alternative — sending back a validation error — wastes a round-trip and confuses the model

---

**Source:** `modelcontextprotocol/servers` — Sequential Thinking server; GitHub code search findings
