# Design Around User Intent, Not API Endpoints

The most common MCP anti-pattern is wrapping each API endpoint as its own tool. This forces the model to orchestrate multi-step workflows that a human developer would automate.

**API-centric (bad):**
```
get_members()          → list of member IDs
get_member_activity()  → activity for one member
get_member_posts()     → posts for one member
get_member_comments()  → comments for one member
```
The model must call `get_members`, then loop through results calling 3 tools per member. This eats tokens, is error-prone, and takes 20+ tool calls.

**Intent-centric (good):**
```python
@tool(description="Get activity insights for all members in a space. Returns members sorted by engagement with their posts, comments, and last active date.")
def get_space_activity(space_id: str, days: int = 30, sort_by: str = "total_activity") -> dict:
    members = api.get_members(space_id)
    for m in members:
        m.activity = api.get_activity(m.id, days=days)
        m.posts = api.get_posts(m.id, days=days)
    return {
        "members": sorted(members, key=lambda m: m.activity.total, reverse=True),
        "summary": f"Found {len(members)} members active in the last {days} days.",
        "next_steps": "Use bulk_message(member_ids=[...]) to contact specific members."
    }
```

**One tool call instead of 20+.** The server does the orchestration internally because it knows the API better than the model ever will.

**The "Sable Principle"** (from community): "When designing MCP capabilities, think about what actions the user would want to take, not what API endpoints exist. If the workflow involves Get Trending Tracks + 4 supporting calls - the 4 supporting calls should not be separate tools."

**When to consolidate:** If 3+ API calls always happen together for a common use case, they belong in one tool.

**Source:** u/sjoti r/mcp (279 upvotes); u/glassBeadCheney r/mcp - "Sable Principle" and Toolhost Pattern
