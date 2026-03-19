# Safe Tool Name Characters Only

Use only `[a-z0-9_]` characters in tool names. Special characters, hyphens, and slashes have caused complete connection failures in production MCP clients — not validation errors, but silent crashes.

## The Bug

Several MCP clients (including early versions of Claude Desktop) crash the entire connection when tool names contain characters outside `[a-zA-Z0-9_]`. The slash character (`/`) is particularly dangerous because some clients try to parse tool names as URI path segments.

Confirmed breaking characters from community reports:
- `/` — crashes Claude Desktop connection silently
- `-` — breaks some tool-router middleware
- `.` — causes issues in OpenAI function-calling adapters
- Spaces — universally rejected, but error messages vary
- Unicode — works in some clients, silent failure in others

## Naming Convention

```typescript
// ❌ DANGEROUS — will crash some clients
const badNames = [
  "read/file",
  "git-status",
  "search.files",
  "get_user's_data",
  "📁-list",
];

// ✅ SAFE — [a-z0-9_] only, underscore_separated
const goodNames = [
  "read_file",
  "git_status",
  "search_files",
  "get_user_data",
  "list_files",
];
```

## Lint Rule (TypeScript)

```typescript
const SAFE_TOOL_NAME = /^[a-z][a-z0-9_]{0,63}$/;

function registerTool(name: string, ...rest: any[]) {
  if (!SAFE_TOOL_NAME.test(name)) {
    throw new Error(
      `Invalid tool name "${name}". Use only lowercase letters, digits, and underscores. ` +
      `Must start with a letter. Max 64 characters.`
    );
  }
  server.tool(name, ...rest);
}
```

Add this check to CI to catch violations before deployment — a broken tool name can take down an entire MCP connection, not just the single tool.

**Why it matters:** Special characters in tool names produce silent failures in the client, not server-side errors. The LLM gets no response, the agent hangs, and the root cause is invisible without client-side logs.

**Source:** r/mcp community issue report (u/sjoti); GitHub issue on modelcontextprotocol/sdk #142; schema-design research synthesis.
