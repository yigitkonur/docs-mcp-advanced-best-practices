# Use File Hashing to Prevent Agent Reprocessing Loops

Hash the full execution state — not just the action — so the agent can detect "I already downloaded and parsed this exact file 30 seconds ago." This kills the common loop where agents re-download, re-parse, or re-process the same data repeatedly.

**Server-side execution cache:**
```python
import hashlib, time, json

execution_cache: dict[str, dict] = {}
CACHE_TTL = 60  # seconds

def get_cache_key(tool_name: str, params: dict) -> str:
    """Hash tool name + all params including file refs."""
    state = json.dumps({"tool": tool_name, "params": params}, sort_keys=True)
    return hashlib.sha256(state.encode()).hexdigest()

def call_tool_with_cache(tool_name: str, params: dict) -> dict:
    force = params.pop("force_refresh", False)
    key = get_cache_key(tool_name, params)
    if not force and key in execution_cache:
        entry = execution_cache[key]
        age = time.time() - entry["timestamp"]
        if age < CACHE_TTL:
            return {"result": entry["result"], "cached": True,
                    "note": f"Cached {int(age)}s ago. Use force_refresh=true to bypass."}
    result = execute_tool(tool_name, params)
    execution_cache[key] = {"result": result, "timestamp": time.time()}
    return {"result": result, "cached": False}
```

**Hash inputs:** tool name, all parameters, file URLs/paths, filter/query values.
**Don't hash:** timestamps, request IDs, auth tokens, pagination cursors.

**The `force_refresh` escape hatch** — always expose as an optional param:

```json
{
  "name": "download_report",
  "description": "Downloads and parses the report. Cached 60s. Use force_refresh=true for fresh data.",
  "inputSchema": {
    "properties": {
      "url": { "type": "string" },
      "force_refresh": { "type": "boolean", "default": false }
    }
  }
}
```

**Source:** u/Main_Payment_6430, r/AI_Agents
