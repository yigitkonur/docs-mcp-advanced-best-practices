# Add Structured Observability from Day One

MCP servers in production need the same observability as any other service. Don't wait for problems to add logging and metrics.

**Structured logging (JSON to stderr):**
```python
import json
import sys
from datetime import datetime

def log_tool_call(tool_name: str, params: dict, result: dict, duration_ms: float):
    entry = {
        "timestamp": datetime.utcnow().isoformat(),
        "level": "INFO",
        "event": "tool_call",
        "tool": tool_name,
        "params": {k: v for k, v in params.items() if k not in SENSITIVE_FIELDS},
        "success": not result.get("isError", False),
        "duration_ms": duration_ms,
        "tokens_estimate": len(json.dumps(result)) // 4
    }
    print(json.dumps(entry), file=sys.stderr)
```

**Key metrics to emit:**
- `tool_call_count` by tool name (which tools are used most?)
- `tool_call_duration_seconds` histogram (are any tools slow?)
- `tool_error_count` by tool name and error type (which tools fail most?)
- `active_sessions` gauge (how many concurrent users?)
- `response_token_estimate` histogram (which tools produce the largest responses?)

**Health checks:**
```python
@app.get("/health")
def health():
    checks = {
        "database": check_db_connection(),
        "cache": check_redis(),
        "external_api": check_api_reachable()
    }
    healthy = all(checks.values())
    return {"status": "healthy" if healthy else "degraded", "checks": checks}
```

**Non-obvious insight:** Track token estimates per response. A tool that returns 20k tokens on every call is burning context window and money. This metric helps you identify tools that need a `response_format` enum or pagination.

**Source:** modelcontextprotocol.info best practices; Pragmatic Engineer - "Building MCP servers in the real world"
