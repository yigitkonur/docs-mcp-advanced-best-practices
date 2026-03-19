# Keep Schemas Flat, Under 6 Parameters

Schema complexity directly impacts tool-call accuracy. Community experience consistently shows flat schemas with 3-6 parameters achieve significantly higher parse success rates than nested schemas. GPT models are especially prone to "self-checking" on nested schemas — they construct a valid input, then second-guess themselves and refuse to call the tool entirely.

## Bad: Nested Object Schema

```typescript
// ❌ Nested — LLMs struggle with this structure
server.tool("search", {
  query: z.string(),
  filters: z.object({
    dateRange: z.object({
      start: z.string(),
      end: z.string(),
    }),
    status: z.enum(["active", "archived"]),
    tags: z.array(z.string()),
  }),
});
```

## Good: Flat Schema

```typescript
// ✅ Flat — same capability, much higher accuracy
server.tool("search", {
  query: z.string(),
  startDate: z.string().optional().describe("Start date (ISO 8601)"),
  endDate: z.string().optional().describe("End date (ISO 8601)"),
  status: z.enum(["active", "archived"]).optional(),
  tags: z.string().optional().describe("Comma-separated tags"),
});
```

## Rules of Thumb

1. **3–6 params** is the sweet spot — high accuracy, enough expressiveness
2. **Never nest 2+ levels** — accuracy drops significantly
3. If you need complexity, **split into multiple tools** rather than one complex schema
4. Treat `z.array()` of primitives as acceptable; `z.array(z.object())` as risky

---

**Source:** Community consensus from [r/mcp](https://reddit.com/r/mcp) discussions on schema complexity
