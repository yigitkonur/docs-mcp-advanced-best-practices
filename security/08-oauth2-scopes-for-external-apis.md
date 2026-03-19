# Use OAuth2 with Granular Scopes for External MCP Servers

For external-facing MCP servers, implement OAuth2 with narrowly scoped permissions. Don't accept tokens not explicitly issued for your server. Start with a minimal scope baseline and request higher privileges only when a specific tool requires them.

**Token validation:**
```python
from fastapi import Depends, HTTPException, Security
from fastapi.security import SecurityScopes

async def verify_mcp_token(security_scopes, token = Depends(oauth2_scheme)):
    payload = jwt.decode(token, PUBLIC_KEY, algorithms=["RS256"])
    if payload.get("aud") != "mcp-server-prod":
        raise HTTPException(403, "Token not issued for this MCP server")
    token_scopes = set(payload.get("scope", "").split())
    for scope in security_scopes.scopes:
        if scope not in token_scopes:
            raise HTTPException(403, f"Missing scope: {scope}")
    return payload
```

**Scope design — principle of least privilege:**
```
mcp:read    → search, list, describe    mcp:write  → create, update
mcp:delete  → destructive (rare)        mcp:admin  → config (never auto-granted)
```

**Block SSRF to private IP ranges.** External-facing MCPs must never reach internal networks:
```python
import ipaddress
import socket
import urllib.parse

BLOCKED = ["10.0.0.0/8", "172.16.0.0/12", "192.168.0.0/16",
           "127.0.0.0/8", "169.254.0.0/16"]
BLOCKED_NETS = [ipaddress.ip_network(n) for n in BLOCKED]

def is_safe_url(url: str) -> bool:
    ip = ipaddress.ip_address(socket.gethostbyname(
        urllib.parse.urlparse(url).hostname))
    return not any(ip in net for net in BLOCKED_NETS)
```

**Session security:**
- Generate session IDs with CSPRNG (`uuid4()`), not sequential IDs.
- Bind sessions to user identity; verify on every request.
- Rotate on privilege escalation. TTL: 15 min for elevated scopes, 1 hour for read-only.

**Source:** [u/hasmcp on r/mcp](https://reddit.com/r/mcp); [NCC Group — 5 MCP Security Tips](https://www.nccgroup.com/research/5-mcp-security-tips/); [MCP specification — security](https://modelcontextprotocol.io)
