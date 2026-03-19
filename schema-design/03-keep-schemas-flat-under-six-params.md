# Keep Schemas Flat, Under 6 Parameters

Schema complexity directly impacts tool-call accuracy. This isn't theoretical — it's been measured.

## Measured Success Rates

| Schema Shape             | Success Rate |
|--------------------------|-------------|
| Flat, 3–6 params         | ~98%        |
| Flat, 10+ params         | ~85%        |
| Nested 1 level           | ~80%        |
| Nested 2+ levels         | < 70%       |

GPT models are especially prone to "self-checking" on nested schemas — they construct a valid input, then second-guess themselves and refuse to call the tool entirely.

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
2. **Never nest 2+ levels** — accuracy drops below 70%
3. If you need complexity, **split into multiple tools** rather than one complex schema
4. Treat `z.array()` of primitives as acceptable; `z.array(z.object())` as risky

---

**Source:** u/CARUFO, r/mcp; deep research synthesis with measured success rates
