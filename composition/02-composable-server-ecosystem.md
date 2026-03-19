# Design Composable Servers That the LLM Orchestrates

Each MCP server should specialize in one domain. Let the LLM orchestrate multi-server workflows - it's better at reasoning about workflow sequencing than any hardcoded router.

**Example: Threat Modeling Workflow**
```
User: "Analyze the security of our new payment API"

LLM orchestration:
1. Calls github_analyze(repo_url) → gets code structure, dependencies
2. Calls threat_framework(app_description=github_result.description) → gets STRIDE analysis scaffold
3. Uses the STRIDE data to generate a threat report
4. Calls create_jira_tickets(threats=identified_threats) → creates tracking tickets
```

**Server design for composability:**
```python
# Server 1: GitHub Analysis
@tool(description="Analyze a GitHub repository's structure, dependencies, and security-relevant patterns.")
def github_analyze(repo_url: str) -> dict:
    return {
        "description": "Payment processing API using Express.js with Stripe integration",
        "dependencies": [...],
        "identified_components": ["auth", "payment", "webhook_handler"],
        "next_steps": "Use threat_framework() with this description for security analysis."
    }

# Server 2: STRIDE Threat Framework
@tool(description="Get STRIDE threat analysis framework data for an application.")
def threat_framework(app_description: str) -> dict:
    return {
        "stride_categories": {...},
        "context_analysis": analyze_app_context(app_description),
        "report_template": "Generate a threat report using the above framework.",
        "data_for_report": {...}  # Raw data for LLM assembly
    }
```

**Key principles for composability:**
1. Each server exposes **data and scaffolds**, not finished outputs
2. Responses include `next_steps` that reference tools from OTHER servers
3. Return data in formats that other tools can consume as input
4. Don't make servers depend on each other directly - let the LLM chain them

**Source:** [Matt Adams — MCP Server Design Principles](https://matt-adams.co.uk/2025/08/30/mcp-design-principles.html); u/glassBeadCheney on [r/mcp](https://reddit.com/r/mcp)
