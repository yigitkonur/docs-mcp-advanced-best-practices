# Accept Flexible Formats, Normalize Server-Side

LLMs express the same concept in many formats. Don't force a specific one in the schema — accept a loose string and parse it yourself.

Community experience shows flexible string inputs achieve higher accuracy than union types (`anyOf`), which LLMs frequently mishandle. Union types (`z.union()`, `anyOf` in JSON Schema) are especially problematic — LLMs often pick the wrong branch or produce malformed hybrid output.

## Bad: Union Type for Dates

```typescript
// ❌ LLMs frequently mishandle union types
date: z.union([
  z.string().datetime(),
  z.number().describe("Unix timestamp"),
  z.enum(["today", "yesterday", "last_week"]),
])
```

## Good: Flexible String, Server-Side Parsing

```typescript
// ✅ Higher accuracy — let the LLM express naturally
date: z.string().describe(
  "Date in any format: ISO 8601, Unix timestamp, or relative ('yesterday', 'last week')"
)
```

Then normalize in your handler:

```typescript
import { parseDate } from "chrono-node"; // or your preferred parser

async function handler({ date }: { date: string }) {
  const parsed = parseDate(date) ?? new Date(date);
  if (!parsed || isNaN(parsed.getTime())) {
    return { content: [{ type: "text", text: "Could not parse date. Try ISO 8601 like '2024-01-15'." }] };
  }
  // Use `parsed` — it's always a valid Date
}
```

## Applies Beyond Dates

- **File paths / Identifiers:** Accept loose formats, resolve server-side

---

**Source:** Community patterns from [r/mcp](https://reddit.com/r/mcp); production experience with union type failures
