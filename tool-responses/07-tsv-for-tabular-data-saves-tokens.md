# Use TSV for Tabular Data to Save 30-40% of Tokens

When your tool returns table-shaped data — database query results, list outputs, CSV-like records — use TSV (tab-separated values) instead of JSON. LLMs parse TSV correctly without any extra prompting, and it saves 30-40% of tokens compared to the JSON equivalent.

**JSON array (high token cost):**
```json
[
  {"name": "Alice", "age": 30, "city": "New York", "role": "admin"},
  {"name": "Bob", "age": 25, "city": "San Francisco", "role": "editor"},
  {"name": "Carol", "age": 35, "city": "Chicago", "role": "viewer"}
]
```

Every row repeats every key. For 100 rows with 5 columns, that's 500 redundant key strings.

**TSV equivalent (~40% fewer tokens):**
```
name	age	city	role
Alice	30	New York	admin
Bob	25	San Francisco	editor
Carol	35	Chicago	viewer
```

Headers appear once. No braces, no quotes, no colons, no commas. The model understands this instantly.

**Implementation pattern:**

```typescript
function toTSV(rows: Record<string, unknown>[]): string {
  if (rows.length === 0) return "(no results)";
  const headers = Object.keys(rows[0]);
  const lines = [
    headers.join("\t"),
    ...rows.map(row => headers.map(h => String(row[h] ?? "")).join("\t"))
  ];
  return lines.join("\n");
}

server.registerTool("query_users", {
  inputSchema: { filter: z.string().optional() },
}, async ({ filter }) => {
  const users = await db.query(`SELECT name, age, city, role FROM users`);
  return {
    content: [{
      type: "text",
      text: `Found ${users.length} users:\n\n${toTSV(users)}`
    }]
  };
});
```

**When TSV breaks down:** Nested objects, arrays within cells, or values containing tabs/newlines. For those, fall back to YAML or JSON. But for the vast majority of database results and list outputs, TSV is the right default.

**Source:** pgEdge, "Lessons Learned Writing an MCP Server for PostgreSQL" — discovered significant token savings switching query results from JSON to TSV.
