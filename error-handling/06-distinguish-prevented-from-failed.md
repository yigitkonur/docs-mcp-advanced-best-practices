# Distinguish "Prevented" from "Failed" in Error Framing

When a tool catches bad input before executing, it *prevented* a problem — it didn't *fail*. The wording you choose changes the agent's entire subsequent behavior.

**Why it matters:**
- "Error detected, operation canceled" triggers frustration loops — the agent assumes *it* broke something and produces unproductive recovery sequences
- "Caught early — here's how to fix it" tells the agent the system is working *with* it, encouraging a single corrective retry

**Bad — punitive framing:**
```json
{
  "content": [{"type": "text", "text": "Error: invalid date. Operation canceled."}],
  "isError": true
}
```

The agent reads this as a hard failure. It may abandon the approach, apologize to the user, or attempt a completely different strategy — all unnecessary.

**Good — preventive framing:**
```json
{
  "content": [{
    "type": "text",
    "text": "The date 2023-11-15 is in the past — schedule_meeting requires a future date. Use a date after 2025-01-15 and retry with schedule_meeting(date='2025-01-20', ...)."
  }],
  "isError": true
}
```

**The pattern in code:**
```python
def schedule_meeting(date: str, title: str):
    parsed = parse_date(date)
    if parsed < datetime.now():
        return {
            "content": [{"type": "text", "text":
                f"Great catch — {date} is in the past. "
                f"Use a future date (after {datetime.now().strftime('%Y-%m-%d')}) "
                f"and retry: schedule_meeting(date='<future_date>', title='{title}')"
            }],
            "isError": True
        }
    # proceed with scheduling...
```

**Key distinction:** `isError: true` tells the protocol something went wrong. The *text* tells the agent whether to feel bad about it or just adjust one parameter. The text matters more than the flag.

**Source:** u/jbr, r/mcp (260 upvotes thread); deep research findings on agent retry behavior
