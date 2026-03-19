# Consolidate Multi-Step Workflows Into Single Atomic Tools

When a common user task requires calling 4+ API endpoints in sequence, wrap the entire workflow into one tool. The agent doesn't need to know the internal steps.

**Before (4 separate tools):**
```python
create_project(name, repo)       # Tool 1
add_env_vars(pid, vars)          # Tool 2
create_deployment(pid, branch)   # Tool 3
add_domain(pid, domain)          # Tool 4
```

**After (1 workflow tool):**
```python
@tool(description="Deploy a new project end-to-end. Creates the project, configures environment variables, deploys from the specified branch, and sets up the custom domain.")
def deploy_project(
    repo_url: str,
    domain: str,
    env_vars: dict,
    branch: str = "main"
) -> dict:
    pid = create_project(repo_url)
    add_env_vars(pid, env_vars)
    deployment = create_deployment(pid, branch)
    add_domain(pid, domain)
    return {
        "status": "success",
        "project_id": pid,
        "deployment_url": deployment.url,
        "domain": domain,
        "message": f"Project deployed to {domain} from {branch} branch."
    }
```

**Benefits:**
- **Token savings**: One tool description vs four (3-4x reduction)
- **Fewer failure points**: Server handles the sequencing
- **Friendlier responses**: Return a conversational summary, not raw status codes
- **Simpler agent reasoning**: One decision instead of four

**When NOT to consolidate:**
- When steps are genuinely independent (user might want env vars without deployment)
- When individual steps need human confirmation between them
- When the workflow varies significantly between use cases

**Implementation tip:** Use try/except around each sub-step and return a structured error indicating which stage failed, so the model knows what was already completed.

**Source:** Klavis AI - "Less is More: 4 design patterns for building better MCP servers"
