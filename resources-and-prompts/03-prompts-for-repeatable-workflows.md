# Use MCP Prompts for Repeatable Workflow Entry Points

Prompts are user-controlled templates that combine instructions, parameters, and attached resources into a reusable workflow trigger. They solve the problem of "I keep typing the same complex instruction."

```python
@mcp.prompt
def analyze_bug(
    error_message: str,
    file_path: str,
    severity: str = "medium"
) -> list[dict]:
    """Structured bug analysis workflow with access to the relevant source file."""
    return [
        {
            "role": "user",
            "content": {
                "type": "text",
                "text": (
                    f"Analyze this bug systematically:\n"
                    f"Error: {error_message}\n"
                    f"File: {file_path}\n"
                    f"Severity: {severity}\n\n"
                    f"1. Identify the root cause\n"
                    f"2. Check for related issues in nearby code\n"
                    f"3. Propose a fix with test cases\n"
                    f"4. Assess risk of the fix"
                )
            }
        },
        {
            "role": "user",
            "content": {
                "type": "resource",
                "resource": {
                    "uri": f"file://{file_path}",
                    "mimeType": "text/plain"
                }
            }
        }
    ]
```

**Prompts vs Tools:**
- **Prompts** are user-initiated (the user selects them from a menu)
- **Tools** are model-initiated (the model decides to call them)
- Prompts can embed resources directly, providing data alongside instructions
- Prompts act like "saved workflows" the user can trigger repeatedly

**Real-world uses:**
- Sequential thinking MCP with prompts for architecture design, bug analysis, refactoring
- Test generation prompts that include the source file as a resource
- Code review prompts with embedded style guides

**Current limitation:** Most clients don't fully support prompt template values persistence yet. Claude Desktop supports prompts but can't persist saved parameter values across sessions.

**Source:** u/mettavestor r/mcp - "I integrated prompts in my sequential thinking MCP designed for coding"; MCP blog - "MCP Prompts: Building Workflow Automation"
