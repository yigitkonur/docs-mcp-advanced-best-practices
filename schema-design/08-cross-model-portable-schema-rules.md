# Cross-Model Portable Schema Rules

If your MCP server is accessed by multiple LLM clients (Claude, GPT-4, Gemini), design your input schemas to work across all of them. Each model has different JSON Schema support, and schemas that work perfectly in Claude can fail silently in GPT or cause Gemini to ignore the tool.

## Compatibility Rules (7 Hard Rules)

```typescript
// Rule 1: No oneOf / anyOf / allOf — Gemini ignores tools with these
// ❌ Bad
{ oneOf: [{ type: "string" }, { type: "number" }] }
// ✅ Good — pick the most permissive type, coerce server-side
{ type: "string", description: "Can also be a number (will be converted)" }

// Rule 2: Max 3 levels of nesting — GPT-4 degrades beyond this
// ❌ Bad
{ type: "object", properties: { a: { type: "object", properties: { b: { type: "object", properties: { c: { type: "string" } } } } } } }

// Rule 3: No $ref — most clients don't resolve references in tool schemas
// Inline all definitions

// Rule 4: No format validators beyond date-time and email — inconsistently supported
// ❌ Bad: { type: "string", format: "uri" }  — GPT-4 ignores format
// ✅ Good: describe the format in the description field instead

// Rule 5: No default for required fields — confuses some model implementations
// Only apply defaults to optional fields

// Rule 6: Keep enums under 10 values — larger enums reduce model accuracy
// Split large enums into a separate "mode" tool that narrows the scope

// Rule 7: Use additionalProperties: false — helps all models understand the schema is strict
```

## Compatibility Matrix

| Feature | Claude | GPT-4o | Gemini |
|---|---|---|---|
| `oneOf`/`anyOf` | ✅ | ✅ | ❌ ignored |
| `$ref` references | ✅ | ⚠️ varies | ❌ ignored |
| `format` constraints | ✅ | ⚠️ ignored | ❌ ignored |
| 5+ levels nesting | ✅ | ⚠️ degraded | ❌ breaks |
| Enum > 20 values | ✅ | ⚠️ degraded | ⚠️ degraded |
| `additionalProperties: false` | ✅ | ✅ | ✅ |

## Test Your Schema

```typescript
// Quick cross-model schema validator
function validatePortability(schema: JsonSchema): string[] {
  const issues: string[] = [];
  if (schema.oneOf || schema.anyOf) issues.push("Remove oneOf/anyOf for Gemini compatibility");
  if (schema.$ref) issues.push("Inline $ref for cross-client compatibility");
  if (countNestingDepth(schema) > 3) issues.push("Nesting too deep (>3) for GPT-4 reliability");
  if (Array.isArray(schema.enum) && schema.enum.length > 10) issues.push("Large enum (>10) degrades accuracy");
  return issues;
}
```

**Why it matters:** A schema that crashes Gemini silently drops the tool from that client's context. Cross-model portability rules let you build once and deploy everywhere.

**Source:** Gemini tool use documentation (ai.google.dev); OpenAI function calling guide; community cross-model testing discussions in r/mcp.
