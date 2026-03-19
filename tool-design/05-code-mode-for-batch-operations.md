# Expose a Code Execution Sandbox for Batch Operations

For data-heavy tasks (batch processing, pagination, custom analytics), expose a single `execute_code` tool that runs LLM-generated Python in a sandbox. This is dramatically more token-efficient than dozens of individual tool calls.

```json
{
  "name": "execute_code",
  "description": "Run Python code in a secure sandbox with access to pre-authenticated API clients (salesforce_client, s3_client). Use for batch operations, data processing, or any task requiring loops/parallelism.",
  "input_schema": {
    "type": "object",
    "properties": {
      "code": {"type": "string", "description": "Python code to execute."},
      "timeout": {"type": "integer", "default": 300, "description": "Execution timeout in seconds."}
    },
    "required": ["code"]
  }
}
```

**Example LLM-generated code:**
```python
from concurrent.futures import ThreadPoolExecutor
import json

def fetch_page(p):
    return salesforce_client.fetch_leads(page=p, limit=1000)

first = fetch_page(1)
total_pages = (first['total_count'] + 999) // 1000

all_leads = []
with ThreadPoolExecutor(max_workers=10) as exe:
    for f in exe.map(fetch_page, range(1, total_pages + 1)):
        all_leads.extend(f['leads'])

s3_url = s3_client.upload(
    data=json.dumps(all_leads),
    filename=f"leads_{len(all_leads)}.json"
)
return {"status": "success", "total": len(all_leads), "s3_url": s3_url}
```

**Security requirements (non-negotiable):**
- Run in isolated containers with `--no-new-privileges`
- Drop all capabilities, mount read-only filesystem
- Enforce CPU/memory limits via cgroups
- Restrict network egress to whitelisted endpoints
- Provide API clients pre-authenticated - never expose raw keys
- Disallow `eval`/`exec` on untrusted strings
- Set hard timeout limits

**When to use:** Batch exports, heavy pagination, custom analytics, file transformations - anywhere the final output is a file/URL rather than a conversational response.

**Source:** [Klavis AI — Less is More: MCP Design Patterns for AI Agents](https://www.klavis.ai/blog/less-is-more-mcp-design-patterns-for-ai-agents)
