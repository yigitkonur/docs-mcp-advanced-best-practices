# Log Successful Tool Calls to Build a Learning Database

When your MCP server handles a complex API, record every successful tool call and its parameters. When future calls fail, query this database for a working example to include in the error response.

```python
import json
from pathlib import Path

SUCCESS_LOG = Path("./successful_calls.jsonl")

@tool
def query_api(endpoint: str, params: dict) -> dict:
    try:
        result = api.call(endpoint, params)
        # Log the successful call
        SUCCESS_LOG.open("a").write(json.dumps({
            "endpoint": endpoint,
            "params": params,
            "timestamp": datetime.now().isoformat()
        }) + "\n")
        return result
    except APIError as e:
        # Find a similar successful call
        similar = find_similar_successful_call(endpoint, params)
        error_msg = f"API call failed: {e}"
        if similar:
            error_msg += (
                f"\n\nA similar successful call used these parameters:\n"
                f"{json.dumps(similar['params'], indent=2)}\n"
                f"Try adjusting your parameters to match this pattern."
            )
        return {"content": [{"type": "text", "text": error_msg}], "isError": True}
```

**The insight:** One practitioner reported this reduced API call errors by more than 50%. The model learns from its own (or other sessions') successful patterns.

**Implementation details:**
- Store in a simple JSONL file or SQLite database
- Index by endpoint and key parameter patterns
- Include timestamps to weight recent successes higher
- Consider embedding-based similarity for fuzzy matching

**Privacy note:** Be careful about logging sensitive parameters. Strip PII before storing.

**Source:** u/Simple-Art-2338 r/mcp - "I tried an approach which worked for me... that reduced my error more than 50%"
