# Four-Stage Progressive Disclosure Pattern

Stage-wise tool exposure mirrors how a human explores an unfamiliar system. Each stage returns minimal data, keeping token consumption low until the agent truly needs the full definition.

**Interaction flow:**
```
1. discover_categories()
   → ["GitHub", "Slack", "Database", "CRM"]

2. get_category_actions("GitHub")
   → [{"name": "create_pr", "summary": "Create a pull request"},
      {"name": "search_repos", "summary": "Search repositories"},
      {"name": "list_issues", "summary": "List issues with filters"}]

3. get_action_details("GitHub", "create_pr")
   → Full JSON schema with all parameters, types, descriptions

4. execute_action("GitHub", "create_pr", {"repo": "...", "title": "...", "body": "..."})
   → Result
```

**Implementation:**
```python
def handle_discovery(stage, **kwargs):
    if stage == "categories":
        return {"stage": "categories", "options": list_available_services()}

    elif stage == "actions":
        service = kwargs["service"]
        return {"stage": "actions", "service": service,
                "options": list_actions(service)}

    elif stage == "schema":
        return {"stage": "schema",
                "schema": get_action_schema(kwargs["service"], kwargs["action"])}
```

**Token budget at each stage:**
- Stage 1: ~100 tokens (category list)
- Stage 2: ~300-500 tokens (action names + summaries)
- Stage 3: ~500-2000 tokens (full schema for one action)
- Stage 4: Variable (execution result)

**vs. loading everything upfront:** Initial context stays at ~2% of the context window regardless of how many integrations you support.

**Best for:** Multi-tenant SaaS where each tenant enables different integrations. The agent discovers what's available for THIS user, not all possible tools.

**Source:** Klavis AI - "Less is More: 4 design patterns"; Speakeasy blog progressive discovery benchmarks
