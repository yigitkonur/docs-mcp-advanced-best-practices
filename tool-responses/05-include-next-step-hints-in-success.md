# Include Next-Step Hints in Successful Responses

Don't just return data - tell the model what it can do next. This is the "HATEOAS at the language level" pattern.

```python
@tool
def create_project(name: str, repo_url: str) -> dict:
    project = api.create_project(name=name, repo=repo_url)
    return {
        "status": "success",
        "project_id": project.id,
        "project_name": name,
        "message": f"Project '{name}' created successfully.",
        "next_steps": [
            f"Use add_environment_variables(project_id='{project.id}') to configure env vars.",
            f"Use create_deployment(project_id='{project.id}', branch='main') to deploy.",
            f"Use add_custom_domain(project_id='{project.id}', domain='...') to set up a domain."
        ]
    }
```

**Why this works:** The model has no memory of your API's workflow. A developer would read your docs and know that after creating a project, the next step is to configure env vars and deploy. The model doesn't know this unless you tell it - and the tool response is the perfect place.

**Guidelines:**
- Keep hints specific: name the exact tool and include required parameter values
- Order hints by likelihood (most common next action first)
- Only include 2-4 hints to avoid overwhelming the context
- Use the actual parameter values from the current response (don't make the model guess)

**Real-world example:** One practitioner built a data visualization MCP server where a "planner tool" returns structured guidance about what order to call visualization tools. Every subsequent tool response reinforces the workflow with further guidance. They call this "flattening the agent back into the model."

**Source:** [u/sjoti on r/mcp](https://reddit.com/r/mcp/comments/1lq69b3/) (279 upvotes); [u/Biggie_2018 on r/mcp](https://reddit.com/r/mcp) — McKinsey [vizro-mcp](https://github.com/mckinsey/vizro)
