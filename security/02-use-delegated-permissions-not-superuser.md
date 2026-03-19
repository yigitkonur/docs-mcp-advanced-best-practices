# Use Delegated Permissions, Not a Shared Superuser Token

The biggest security mistake in MCP servers is using a single admin API key for all operations. If a prompt injection attack succeeds, it has full access to everything.

**Bad:**
```python
# Single shared credential for all requests
api = ExternalAPI(api_key=os.environ["ADMIN_API_KEY"])

@tool
def get_user_data(user_id: str):
    return api.get(f"/users/{user_id}")  # Can access ANY user
```

**Good:**
```python
@tool
def get_user_data(user_id: str, ctx: Context):
    # Use the requesting user's delegated token
    user_token = ctx.session.auth_token
    api = ExternalAPI(token=user_token)

    # The API itself enforces access control
    return api.get(f"/users/{user_id}")  # Only returns data this user can see
```

**Implementation patterns:**
- **Entra ID / OAuth delegation:** Use the requesting user's OAuth token, not a service account. The underlying application's RBAC naturally limits what can be accessed.
- **Per-user JWT tokens:** Issue scoped tokens at session creation that encode the user's permission level.
- **Row-level security:** If querying a database, filter by the user's tenant/org ID at the query level.

**Why it matters:** This is the strongest defense against prompt injection. Even if an attacker tricks the model into calling tools maliciously, the damage is limited to what that specific user can already access.

**Source:** u/cake97 r/mcp - "We use delegated permissions via entra. Relies on the underlying RBAC in place from the app."; Christian Schneider - "Securing MCP: a defense-first architecture guide"
