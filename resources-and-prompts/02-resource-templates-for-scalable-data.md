# Use Resource Templates with URI Patterns for Scalable Data Access

Instead of creating static resources for every data variant, use URI-based resource templates. One template definition serves hundreds of variants.

```typescript
server.registerResource(
  "recipes",
  new ResourceTemplate("file://recipes/{cuisine}", {
    list: undefined,
    complete: {
      cuisine: (value) => CUISINES.filter(c => c.startsWith(value)),
    },
  }),
  { title: "Cuisine-Specific Recipes", description: "Markdown recipes per cuisine" },
  async (uri, vars) => {
    const cuisine = vars.cuisine as string;
    if (!CUISINES.includes(cuisine)) {
      throw new Error(`Unknown cuisine: ${cuisine}. Valid: ${CUISINES.join(', ')}`);
    }
    return {
      contents: [{
        uri: uri.href,
        mimeType: "text/markdown",
        text: formatRecipesAsMarkdown(cuisine)
      }],
    };
  },
);
```

**Key features:**
- **URI pattern matching**: `file://recipes/{cuisine}` resolves dynamically
- **Completions**: Auto-suggest valid values when the user types (supported in VS Code MCP extension and some other clients)
- **Server-side validation**: Reject invalid values with clear errors
- **Single definition**: Handles Italian, Japanese, Mexican, etc. without separate resource registrations

**When resources beat tools for data access:**
- Data is read-only and changes infrequently
- Same data is referenced by multiple tools/prompts
- The data benefits from caching (resources support ETags)
- You want URI-addressable content for cross-server referencing

**When to use tools instead:** When the "read" has parameters beyond a URI path (complex filters, pagination, sorting).

**Source:** [MCP specification — resources](https://modelcontextprotocol.io/specification/2025-11-25/server/resources); [modelcontextprotocol.io](https://modelcontextprotocol.io) resource templates documentation
