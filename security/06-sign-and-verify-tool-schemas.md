# Sign and Verify Tool Schemas Before Execution

The vulnerability isn't malicious code — it's malicious instructions in tool descriptions. You can write every line of a tool yourself and still be compromised if an attacker pushes a poisoned description through an external data source, config file, or registry.

**The threat model:** Tool schemas (name, description, parameters) are loaded at runtime. If any part of that pipeline is writable by an attacker — a shared config repo, a package registry, a remote manifest — the tool description becomes an injection vector.

**Mitigation: Schema hashing + verification on load.**

```python
import hashlib
import hmac
import json

SIGNING_SECRET = os.environ["MCP_SCHEMA_SIGNING_SECRET"]

def sign_schema(schema: dict) -> str:
    """Sign a tool schema at build/publish time."""
    canonical = json.dumps(schema, sort_keys=True, separators=(",", ":"))
    return hmac.new(
        SIGNING_SECRET.encode(),
        canonical.encode(),
        hashlib.sha256
    ).hexdigest()

def verify_schema(schema: dict, expected_sig: str) -> bool:
    """Verify schema integrity before registering the tool."""
    actual_sig = sign_schema(schema)
    return hmac.compare_digest(actual_sig, expected_sig)

# At server startup
for tool in loaded_tools:
    if not verify_schema(tool.schema, tool.signature):
        logger.critical(f"Schema tampered: {tool.name}. Refusing to load.")
        raise SchemaIntegrityError(tool.name)
```

**Workflow:**
1. **Build time:** Hash and sign each tool schema. Store signatures in a verified manifest.
2. **Load time:** Before registering any tool, verify its schema against the signed manifest.
3. **Runtime:** Reject tools with missing or mismatched signatures. Log and alert on failures.

**What to include in the signed payload:** Tool name, description, parameter schemas, and any annotation fields (`readOnlyHint`, `destructiveHint`). An attacker flipping `destructiveHint` from `true` to `false` can bypass confirmation gates.

**Source:** u/Additional-Value4345 r/mcp; u/NexusVoid_AI r/mcp
