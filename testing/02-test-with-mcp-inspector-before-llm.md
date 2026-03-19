# Test Tools with MCP Inspector Before Involving an LLM

Don't debug tool behavior through the LLM. Use the MCP Inspector to call tools directly and inspect raw JSON-RPC payloads.

**Quick launch:**
```bash
npx @modelcontextprotocol/inspector@latest
# Opens http://localhost:5173
```

**What to verify with Inspector:**
1. Tool discovery: Does `tools/list` return correct schemas?
2. Input validation: Do bad parameters produce clear errors?
3. Response format: Are responses structured as expected?
4. Error handling: Do failures return `isError: true` with guidance?
5. No stdout pollution: Does the server write only JSON-RPC to stdout?

**The debugging loop:**
```
1. REPRODUCE → Capture exact input that triggers the bug
2. CHECK LOGS → Pull stderr logs for stack traces
3. ISOLATE → Disable all but one tool to narrow scope
4. INSPECTOR TEST → Send crafted JSON-RPC payloads
5. FIX & VERIFY → Run the reproduced case + full test suite
```

**Three layers of testing:**
1. **Unit tests**: Test business logic functions directly (no MCP overhead)
2. **Integration tests**: Spin up the MCP server, call tools via the protocol
3. **Eval tests**: Run multi-step agent workflows with an actual LLM

Start from layer 1 and only involve the LLM (layer 3) after layers 1-2 pass. This saves time and API costs.

**Non-obvious tip:** Temporarily disable all but one tool to see if an issue persists. This isolates whether the problem is tool-specific or systemic.

**Source:** Stainless - "Error Handling And Debugging MCP Servers"; NearForm - "Implementing MCP: Tips, tricks and pitfalls"
