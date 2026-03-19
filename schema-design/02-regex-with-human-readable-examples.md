# Pair Regex Validation with Human-Readable Examples

When you use `.regex()` in Zod, the LLM never sees your regex pattern — it only sees the error message on failure. Make that error message a working example the model can copy.

## Bad: Opaque Regex

```typescript
// ❌ LLM gets: "Invalid input" — no idea what format is expected
nodeId: z.string().regex(/^I?\d+[:|-]\d+/)
```

## Good: Regex + Example in Error Message

```typescript
// ✅ LLM gets: "Node ID must be like '1234:5678' or 'I5666:180910;1:10515'"
nodeId: z.string()
  .regex(
    /^I?\d+[:|-]\d+/,
    "Node ID must be like '1234:5678' or 'I5666:180910;1:10515'"
  )
```

The Figma Context MCP server uses this exact pattern — Figma node IDs have a non-obvious format, and providing examples in error messages lets the LLM self-correct on the next attempt.

## Apply This to Any Format Constraint

```typescript
// Hex color
color: z.string()
  .regex(/^#[0-9a-fA-F]{6}$/, "Must be hex color like '#FF5733' or '#00aa99'")

// Semver
version: z.string()
  .regex(/^\d+\.\d+\.\d+$/, "Must be semver like '1.2.3' or '0.10.0'")

// Cron expression
schedule: z.string()
  .regex(/^(\S+\s){4}\S+$/, "Must be cron like '0 9 * * 1' (every Monday 9am)")
```

## Key Insight

The error message is your **teaching interface** with the LLM. When validation fails, the model reads the error, adjusts, and retries. A good example in the error message means the retry almost always succeeds.

---

**Source:** `GLips/Figma-Context-MCP`; GitHub code search findings
