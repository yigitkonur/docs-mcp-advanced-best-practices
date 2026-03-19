# Describe Each Enum Value Inline

Don't just list enum values — explain what each one means in the `.describe()` text. The LLM gets semantics, not just labels.

## Measured Impact

| Enum Strategy              | Valid Call Rate |
|----------------------------|---------------|
| Free-form string (no enum) | 60–70%        |
| Enum values only           | ~95%          |
| Enum + inline descriptions | 97%+          |

Enums alone are a massive improvement. Adding inline descriptions of each value pushes accuracy even higher and reduces cases where the model picks the wrong value.

## Bad: Enum Without Context

```typescript
// ❌ LLM knows the options but not when to use each
livecrawl: z.enum(["fallback", "preferred"])
```

## Good: Enum with Inline Descriptions

```typescript
// ✅ LLM understands the semantics of each option
livecrawl: z.enum(["fallback", "preferred"]).describe(
  "Live crawl mode - 'fallback': use live crawling as backup if cached " +
  "version is unavailable, 'preferred': always prioritize live crawling " +
  "over cache (default: 'fallback')"
)
```

The Exa MCP server uses this pattern — without the inline description, LLMs defaulted to `"preferred"` (sounds better), wasting crawl budget. With the description, they correctly default to `"fallback"`.

## More Examples

```typescript
// Log level with behavior descriptions
level: z.enum(["error", "warn", "info", "debug"]).describe(
  "'error': only critical failures, 'warn': errors + potential issues, " +
  "'info': general operational events, 'debug': verbose output for troubleshooting"
)

// Output format with use-case guidance
format: z.enum(["json", "csv", "markdown"]).describe(
  "'json': structured data for programmatic use, 'csv': tabular data " +
  "for spreadsheets, 'markdown': human-readable formatted output"
)
```

---

**Source:** `exa-labs/exa-mcp-server`; GitHub findings; deep research
