# Add a Response Format Enum to Every Data-Returning Tool

Give the agent control over response verbosity with a `response_format` parameter. This single pattern can cut token usage by 60-70%.

```python
from enum import Enum

class ResponseFormat(Enum):
    DETAILED = "detailed"
    CONCISE = "concise"

@tool
def get_customer(customer_id: str, response_format: ResponseFormat = ResponseFormat.CONCISE):
    data = fetch_customer(customer_id)

    if response_format == ResponseFormat.DETAILED:
        return {
            "name": data.name,
            "email": data.email,
            "recent_transactions": data.transactions,
            "notes": data.notes,
            "internal_id": data.id,
            "thread_ts": data.thread_ts  # for downstream tool calls
        }
    else:
        return {
            "name": data.name,
            "email": data.email,
            "summary": data.summary
        }
```

**Measured impact:** Concise responses use ~72 tokens vs ~206 tokens for detailed ones - a 65% reduction per call. Over a multi-step workflow with 10+ tool calls, this compounds dramatically.

**When the agent should use each mode:**
- **Concise** (default): For browsing, scanning, initial discovery
- **Detailed**: When the agent needs IDs or metadata for follow-up tool calls

This avoids creating two separate tools for the same data, keeping the tool count low.

**Source:** Anthropic Engineering Blog - "Writing effective tools for AI agents"; modelcontextprotocol.info writing effective tools tutorial
