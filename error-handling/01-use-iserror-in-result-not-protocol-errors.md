# Use isError in the Result Object, Not Protocol-Level Errors

MCP has two error paths - and picking the wrong one is a common mistake that kills agent recovery.

**Protocol-level error** (JSON-RPC `error` field): The tool wasn't found, the request was malformed, or the server crashed. The LLM typically never sees this - the client swallows it.

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": { "code": -32001, "message": "Request Timeout" }
}
```

**Tool-call error** (`isError: true` in `result`): The tool was called and ran, but the operation failed. This gets injected into the LLM's context window, enabling the model to reason about and recover from the failure.

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Cannot terminate instance while it is running. Call stop_instance(instance_id='i-abc123') first, then retry."
      }
    ],
    "isError": true
  }
}
```

**The rule:** Use protocol-level errors only for actual protocol failures (bad JSON, unknown method, server crash). For all business logic failures - validation errors, permission issues, resource not found - use `isError: true` in the result so the LLM can see the error and attempt recovery.

**Why it matters:** When you throw a protocol-level error for a business failure, the LLM never sees the error message. It gets a generic "tool call failed" and has no information to fix the problem. With `isError: true`, the error text becomes part of the conversation and the model can adjust its approach.

**Source:** alpic.ai/blog - "Better MCP tool call error responses"; MCP specification - Tools section (modelcontextprotocol.io)
