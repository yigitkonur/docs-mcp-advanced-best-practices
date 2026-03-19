# Validate Authentication Tokens at Execution Time, Not Startup

Defer token validation to the moment a tool actually needs it. If you validate at startup, an expired or invalid token prevents the MCP client from even discovering what tools are available — the entire server is dead on arrival.

**Wrong — startup validation (server crashes if token is bad):**
```python
api_token = os.environ["API_TOKEN"]
httpx.get("https://api.example.com/me",
          headers={"Authorization": f"Bearer {api_token}"}).raise_for_status()  # → crash
server = Server("my-server")  # Never reached
```

**Right — validate at execution time:**
```python
server = Server("my-server")

@server.tool("search_tickets")
async def search_tickets(query: str) -> list[TextContent]:
    api_token = os.environ.get("API_TOKEN")
    if not api_token:
        return [TextContent(
            type="text",
            text="API_TOKEN not set. Add it to your environment and restart the server."
        )]

    try:
        async with httpx.AsyncClient() as client:
            resp = await client.get(
                "https://api.example.com/tickets",
                params={"q": query},
                headers={"Authorization": f"Bearer {api_token}"},
            )
            resp.raise_for_status()
            return [TextContent(type="text", text=resp.text)]
    except httpx.HTTPStatusError as e:
        if e.response.status_code == 401:
            return [TextContent(
                type="text",
                text="Token expired or invalid. Re-authenticate at https://example.com/settings/tokens"
            )]
        raise
```

**What this buys you:**
- Tool discovery always works regardless of auth state
- Token errors are **actionable** ("re-authenticate at URL") instead of opaque connection failures
- Tokens that rotate or expire mid-session are handled naturally
- Tools with different auth requirements degrade independently

**Same principle applies to:** API keys, OAuth tokens, service account credentials, SSL certs.

**Source:** [Docker Blog — MCP Server Best Practices](https://www.docker.com/blog/mcp-server-best-practices/)
