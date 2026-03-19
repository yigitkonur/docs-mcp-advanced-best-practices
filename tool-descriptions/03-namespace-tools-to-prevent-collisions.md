# Namespace Tools to Prevent Collisions Across MCP Servers

When multiple MCP servers are connected to the same agent, tool name collisions cause silent failures or unpredictable routing. Namespace every tool with a consistent prefix.

```
asana_search_tasks
asana_create_task
jira_search_issues
jira_create_issue
```

**Naming scheme options:**
- `{service}_{action}` - e.g., `github_create_pr`
- `{service}_{resource}_{action}` - e.g., `asana_projects_search`

Pick one scheme and use it consistently. Different LLMs respond better to different schemes, so test with your target model.

**Anti-pattern:** Generic names like `search`, `create`, `update` that collide the moment you connect a second MCP server.

**Why it matters:** Agents use tool names as the first disambiguation signal. Without namespacing, the model must rely entirely on description text to distinguish between `search` (contacts) and `search` (files), which fails under context pressure.

**Source:** Anthropic Engineering Blog - "Writing effective tools for AI agents"; modelcontextprotocol.info writing effective tools tutorial
