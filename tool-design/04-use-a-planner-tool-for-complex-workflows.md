# Use a Planner Tool to Teach the Model Your Workflow

For MCP servers with multiple tools that must be called in a specific order, create an explicit planner tool that returns the workflow as structured guidance.

```python
@tool(description="Get the recommended workflow plan for the current task. Call this FIRST before using any other visualization tools.")
def create_plan(task_description: str) -> dict:
    return {
        "workflow": [
            {"step": 1, "tool": "load_data", "description": "Load the dataset", "required_params": ["file_path"]},
            {"step": 2, "tool": "analyze_columns", "description": "Understand data types and distributions"},
            {"step": 3, "tool": "create_chart", "description": "Generate the visualization", "required_params": ["chart_type", "x_column", "y_column"]},
            {"step": 4, "tool": "export_dashboard", "description": "Export the final result"}
        ],
        "instructions": "Follow these steps in order. Each tool's response will provide specific guidance for the next step.",
        "important": "Do not skip steps. Step 2 output is required for step 3."
    }
```

**Why this works:** The model sees this tool first and now has a roadmap. Every subsequent tool response reinforces the workflow with further guidance about what comes next.

**Real-world example:** A data visualization MCP server (vizro-mcp by McKinsey) uses this exact pattern. The planner tool returns structured guidance, and every subsequent tool response contains next-step hints. The creator calls this "flattening the agent back into the model."

**Key detail:** Mark the planner tool's description with "Call this FIRST" or similar. The model treats this as a strong signal to invoke it before anything else.

**When to use:**
- Your server has 5+ tools that form a pipeline
- The order of operations matters
- New users (or models) can't intuit the correct sequence

**Source:** u/Biggie_2018 r/mcp - McKinsey vizro-mcp project; u/sjoti r/mcp thread
