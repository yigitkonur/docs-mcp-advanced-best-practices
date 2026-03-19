# Return Normalized Inputs in Every Response

When your server accepts flexible input formats and normalizes them internally (e.g., "yesterday" → "2024-01-15"), include the normalized values in the response. This teaches the agent the canonical format, improving every subsequent call.

**Without normalized feedback:**
```
Agent: get_events(date="yesterday")       → works
Agent: get_events(date="the day before")  → works (maybe)
Agent: get_events(date="24 hours ago")    → works (maybe)
```
The agent never learns the canonical format. It keeps inventing variations, each one a parsing gamble.

**With normalized feedback:**
```json
{
  "result": { "events": [{"title": "Standup", "time": "09:00"}] },
  "normalized_inputs": {
    "date": "2025-01-15",
    "note": "Interpreted 'yesterday' as 2025-01-15"
  }
}
```
After seeing this once, the agent starts using `"2025-01-15"` format directly.

**Implementation:**
```python
from datetime import datetime, timedelta

def get_events(date: str):
    resolved = resolve_date(date)
    events = db.query_events(resolved)
    return {
        "content": [{"type": "text", "text": json.dumps({
            "events": events,
            "normalized_inputs": {
                "date": resolved.strftime("%Y-%m-%d"),
                "note": f"Interpreted '{date}' as {resolved.strftime('%Y-%m-%d')}"
            }
        })}]
    }

def resolve_date(raw: str) -> datetime:
    shortcuts = {"today": datetime.now(), "yesterday": datetime.now() - timedelta(days=1)}
    return shortcuts.get(raw.lower()) or dateparser.parse(raw)
```

**Same principle for rate-limit errors — return structured retry info:**
```json
{"rate_limited": true, "retry_after_seconds": 30, "reset_at": "2025-01-15T10:05:00Z"}
```

**The principle:** Every response is a training signal. Return the canonical form of what you understood, and the agent converges on clean inputs fast.

**Source:** Arcade.dev, "54 MCP Tool Patterns"; deep research synthesis on agent learning from tool feedback
