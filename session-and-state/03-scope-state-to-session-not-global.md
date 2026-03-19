# Scope State to Session, Not Global Variables

When your MCP server needs to maintain state (auth tokens, pagination cursors, user preferences), always scope it to the session identifier. Never use module-level globals.

**Bad:**
```python
# Global state - shared across all sessions
current_page = 0
auth_token = None

@tool
def next_page():
    global current_page
    current_page += 1  # Race condition when multiple sessions are active
    return fetch_page(current_page)
```

**Good:**
```python
from collections import defaultdict

session_state = defaultdict(dict)

@tool
def next_page(ctx: Context):
    sid = ctx.session_id
    page = session_state[sid].get("page", 0) + 1
    session_state[sid]["page"] = page
    return fetch_page(page)

@tool
def set_preferences(ctx: Context, timezone: str, language: str):
    session_state[ctx.session_id]["prefs"] = {
        "timezone": timezone,
        "language": language
    }
    return {"status": "Preferences saved for this session."}
```

**State management guidelines:**
- **Short-lived state** (pagination, current context): In-memory dict keyed by session ID
- **Medium-lived state** (user prefs, cached results): Redis with TTL
- **Long-lived state** (user quotas, history): Database

**Clean up:** Set TTLs or implement session cleanup to prevent memory leaks. Treat session state like API state: create on first use, clear when done, never leak between users.

**Source:** NearForm - "Implementing MCP: Tips, tricks and pitfalls"; modelcontextprotocol.info best practices
