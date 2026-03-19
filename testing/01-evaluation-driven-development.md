# Use Evaluation-Driven Development with Realistic Multi-Step Tasks

Don't test MCP tools in isolation. Build an evaluation suite that tests complete user workflows through the agent-tool interaction loop.

**The eval loop:**
```python
import anthropic

def run_eval(task: dict, tools: list) -> dict:
    client = anthropic.Anthropic()
    messages = [{"role": "user", "content": task["prompt"]}]

    while True:
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            system="You are testing MCP tools. Think step by step before each tool call.",
            tools=tools,
            messages=messages
        )

        if response.stop_reason == "end_turn":
            return {
                "success": verify_against_ground_truth(response, task["expected"]),
                "tool_calls": count_tool_calls(messages),
                "tokens_used": response.usage.input_tokens + response.usage.output_tokens,
                "transcript": messages
            }

        # Process tool calls and continue the loop
        for block in response.content:
            if block.type == "tool_use":
                result = execute_mcp_tool(block.name, block.input)
                messages.append({"role": "assistant", "content": response.content})
                messages.append({"role": "user", "content": [{
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": result
                }]})
```

**What to measure beyond accuracy:**
- Total number of tool calls (fewer = better design)
- Token consumption per task
- Error rate and recovery rate
- Time to completion
- Whether the model chose the right tool on the first try

**Pro tip - CoT before tool calls:** Include reasoning/feedback blocks in the system prompt before any tool-call block to trigger chain-of-thought. This surfaces the model's decision-making and helps you spot tool selection confusion.

**Pro tip - auto-refactoring:** Feed the full evaluation transcript back to Claude and ask it to suggest description tweaks, parameter clarifications, and schema fixes based on where the model struggled.

**Source:** [Anthropic — Writing effective tools for AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents); [modelcontextprotocol.io — writing effective tools](https://modelcontextprotocol.io)
