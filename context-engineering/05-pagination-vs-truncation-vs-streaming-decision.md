# Use the Right Data Delivery Strategy for Each Tool Type

How you deliver data from a tool matters as much as what data you deliver. The three common strategies — pagination, truncation, and streaming — each have distinct cost and quality profiles, and choosing the wrong one for the use case is one of the most common sources of avoidable context waste or data loss in MCP servers.

**Pagination** suits lists, database queries, and anything likely to exceed 100 items. The per-page token cost is low and bounded, but it adds 2–3 extra turns as the model requests subsequent pages. No data is lost. **Truncation** suits one-shot responses and logs where the model needs a quick look, not a complete picture. It enforces a hard token ceiling with zero latency overhead, but quality degrades sharply once you exceed roughly 5,000 items — the model never knows what was cut. **Streaming** suits live data and real-time log tailing where freshness matters. Each chunk is cheap, delivery is real-time, but the model receives an incomplete context view until the stream closes, which can cause premature reasoning.

The highest-signal pattern in practice is a **hybrid first-page-plus-summary**: return the first page with a `total_results` count, a `has_more` flag, and a hint string. The model can decide in one turn whether the first page is sufficient or whether it should fetch page 2. This avoids the full cost of eager pagination while giving the model the data it needs to make that decision itself — rather than silently missing records.

```typescript
interface PaginatedResponse<T> {
  results:       T[];
  total_results: number;
  page:          number;
  total_pages:   number;
  has_more:      boolean;
  hint?:         string;   // only included when has_more === true
}

function paginateResults<T>(
  items:    T[],
  page:     number = 1,
  pageSize: number = 20,
): PaginatedResponse<T> {
  const total_pages = Math.ceil(items.length / pageSize);
  const start  = (page - 1) * pageSize;
  const slice  = items.slice(start, start + pageSize);
  const has_more = page < total_pages;

  return {
    results:       slice,
    total_results: items.length,
    page,
    total_pages,
    has_more,
    ...(has_more && {
      hint: `Showing ${slice.length} of ${items.length}. Use page=${page + 1} for more.`,
    }),
  };
}

// Truncation helper — always tell the model what was cut
function truncateWithSignal(text: string, maxChars = 8000): string {
  if (text.length <= maxChars) return text;
  const dropped = text.length - maxChars;
  return text.slice(0, maxChars) +
    `\n\n[...truncated — ${dropped} chars omitted. Use a narrower query for more.]`;
}
```

**Why it matters:** Pagination without a summary forces extra turns; truncation without a signal causes silent data loss; hybrid gives the model full agency over whether to pay the cost of fetching more.

**Source:** Production patterns from LlamaIndex, Anthropic tool-use best practices docs, and r/mcp community discussion on large-result tool design.
