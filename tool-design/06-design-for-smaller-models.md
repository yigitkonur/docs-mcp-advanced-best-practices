# Design Tool Workflows for 3-5 Calls, Not 20+

Frontier models handle 20+ sequential tool calls, but smaller and open-source models degrade after 5-7 — they forget earlier results, hallucinate parameters, or loop. If your MCP server requires 15 tool calls to complete a common task, you've locked yourself into expensive frontier models.

The fix is to design tools so that the most common user goals complete in 3-5 calls. This doesn't mean fewer tools — it means each tool does more meaningful work per invocation.

**Before: 12+ calls to analyze a repo**
```python
list_repos()              # Call 1
get_repo(id)              # Call 2
list_branches(repo_id)    # Call 3
get_branch(repo_id, "main")  # Call 4
list_commits(branch_id)   # Call 5-6 (pagination)
get_commit(commit_id)     # Call 7-12 (per commit)
```

**After: 2 calls**
```python
@tool(description="Get recent activity summary for a repository. Returns latest commits, active branches, and contributor stats for the specified time range.")
def get_repo_activity(
    repo: str,
    days: int = 7,
    max_commits: int = 20
) -> dict:
    repo_info = api.get_repo(repo)
    branches = api.list_branches(repo)
    commits = api.list_commits(repo, since=days_ago(days), limit=max_commits)
    return {
        "repo": repo_info.name,
        "default_branch": repo_info.default_branch,
        "active_branches": [b.name for b in branches if b.updated_recently(days)],
        "recent_commits": [{"sha": c.sha[:8], "message": c.message, "author": c.author} for c in commits],
        "summary": f"{len(commits)} commits across {len(branches)} branches in the last {days} days."
    }
```

**Why it matters:** Designing for 3-5 calls means your MCP server works with GPT-4o-mini, Haiku, Gemma, and every future small model — not just Claude Opus. It also cuts latency and token costs by 4-10x. If only Sonnet can use your server, you've dramatically limited your audience.

**Source:** [u/sjoti on r/mcp](https://reddit.com/r/mcp/comments/1lq69b3/) — "Design things in a way that tasks only take 3-5 tool calls instead of 20+"
