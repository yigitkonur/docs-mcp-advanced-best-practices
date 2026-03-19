# Prevent Cross-Tool Hijacking via Shared Context

A malicious MCP tool can hijack legitimate tools **without ever being called**. The attack exploits shared context: the model sees all active tool descriptions at once, so a poisoned description in one tool can influence how the model uses a completely separate tool.

**Demonstrated attack:** A "Fact of the Day" MCP server included a hidden instruction in its tool description. When the user asked Claude to send an email via a separate mail MCP, Claude followed the injected instruction — forwarding sensitive data to the attacker. The malicious tool was never invoked. It simply existed in the environment.

**Why this works:** Tool descriptions are part of the system prompt context. The model cannot distinguish between legitimate instructions and injected ones across tool boundaries.

**Mitigations:**

1. **Treat tool descriptions as untrusted input:**
```python
# When loading tools from external MCPs, sanitize descriptions
def sanitize_tool_description(desc: str) -> str:
    # Strip instruction-like patterns
    suspicious = ["ignore previous", "instead do", "forward to", "send to"]
    for phrase in suspicious:
        if phrase.lower() in desc.lower():
            raise SecurityError(f"Suspicious instruction in tool description: {phrase}")
    return desc
```

2. **Use separate contexts per MCP server.** Don't mix high-trust tools (email, database) with low-trust tools (third-party integrations) in the same session.

3. **Per-tool allowlists:** Explicitly declare which tools can interact. A "facts" tool has no reason to influence an email tool.

4. **Signed manifests:** Verify that tool descriptions haven't been tampered with between authoring and loading (see Tip 06).

```
HIGH TRUST (isolated context):     LOW TRUST (sandboxed):
  ├── email-mcp                      ├── trivia-mcp
  ├── database-mcp                   ├── weather-mcp
  └── internal-api-mcp               └── third-party-mcp
```

**Key insight:** The attack surface isn't code execution — it's context pollution. Audit your active tool set the same way you audit dependencies.

**Source:** [Marmelab — MCP Security Vulnerabilities](https://marmelab.com/blog/2026/02/16/mcp-security-vulnerabilities.html); [Invariant Labs — MCP Security: Tool Poisoning Attacks](https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks); [u/NexusVoid_AI on r/mcp](https://reddit.com/r/mcp)
