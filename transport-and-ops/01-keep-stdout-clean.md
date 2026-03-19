# Keep stdout Pure JSON-RPC - All Logs to stderr

The #1 cause of mysterious MCP server failures: a stray `print()` or `console.log()` on stdout corrupts the JSON-RPC message stream.

**The rule:** stdout must contain ONLY valid JSON-RPC messages. Everything else goes to stderr.

**Python:**
```python
import logging
import sys

# Configure all logging to stderr
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s %(levelname)s %(message)s',
    stream=sys.stderr
)

logger = logging.getLogger(__name__)

# NEVER use print() in an MCP server
# Use logger instead
logger.info("Server started")
logger.error("Tool failed", exc_info=True)
```

**Node.js:**
```javascript
const logger = {
    info: (msg, data) => console.error(`[INFO] ${msg}`, data ?? ''),
    error: (msg, err) => console.error(`[ERROR] ${msg}`, err?.stack || err || '')
};

// NEVER use console.log() in an MCP server
// Use console.error() or the logger
logger.info("Server started");
```

**Common traps:**
- Third-party libraries that write to stdout (disable their logging or redirect)
- Debugging `print` statements left in production code
- Framework startup banners (suppress them)
- Health check responses written to stdout instead of HTTP

**How to verify:** Run your server and pipe stdout through `jq`. If `jq` fails to parse, you have stdout pollution.

```bash
node my-server.js 2>/dev/null | jq .
# Should parse cleanly - any error means stdout pollution
```

**Source:** Stainless - "Error Handling And Debugging MCP Servers"; NearForm - "Implementing MCP: Tips, tricks and pitfalls"
