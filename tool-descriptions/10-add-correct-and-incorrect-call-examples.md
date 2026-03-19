# Add Correct AND Incorrect Call Examples in Descriptions

Including concrete examples in tool descriptions — both happy-path and common mistakes — measurably improves task completion. Even minor wording changes in descriptions can shift success rates by 25-35%. A well-placed example eliminates an entire class of misuse.

Keep it to one correct example and one edge case or mistake. More than that bloats the prompt without proportional benefit.

```json
{
  "name": "search_orders",
  "description": "Search orders by customer, date range, or status. Returns max 50 results.\n\nExample — search by date range:\n{\"customer_id\": \"cust_123\", \"after\": \"2024-01-01\", \"before\": \"2024-03-01\"}\n\nExample — common mistake (wrong date format):\n{\"customer_id\": \"cust_123\", \"after\": \"Jan 1 2024\"}\nDates must be ISO 8601 (YYYY-MM-DD). Natural language dates will fail.\n\nExample response:\n{\"orders\": [{\"id\": \"ord_456\", \"status\": \"shipped\", \"total\": 99.50}], \"has_more\": true}"
}
```

The example response is equally valuable. When agents see the shape of the output, they can plan downstream tool calls without making a dummy request first. This cuts round-trips and reduces hallucinated assumptions about return formats.

**What to include in examples:**
- One happy-path call with realistic parameter values
- One common mistake with a brief explanation of why it fails
- One example response showing the actual return shape

**What NOT to include:**
- Exhaustive parameter combinations (that's what the schema is for)
- Lengthy prose explanations alongside examples (the example should speak for itself)

**Why it matters:** Models learn calling conventions from examples faster than from prose. A single concrete example outperforms a paragraph of rules — and including a wrong example inoculates against the most frequent failure mode.

**Source:** Anthropic, "Writing Tools for Agents" (25-35% improvement stat); u/gauthierpia, r/AI_Agents; u/GentoroAI, r/mcp
