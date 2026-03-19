# Feed Evaluation Transcripts Back for Auto-Refactoring

After running evaluation tasks, concatenate the full tool call transcripts and feed them back to Claude. The model can spot description ambiguities, schema inconsistencies, and naming issues that humans miss.

**The workflow:**
```python
# 1. Run eval and capture transcripts
eval_results = []
for task in eval_tasks:
    result = run_eval(task, tools)
    eval_results.append({
        "task": task["description"],
        "success": result["success"],
        "transcript": result["transcript"],
        "tool_calls": result["tool_calls"],
        "tokens": result["tokens_used"]
    })

# 2. Feed transcripts to Claude for analysis
analysis_prompt = f"""
Analyze these MCP tool evaluation transcripts and suggest improvements:

{json.dumps(eval_results, indent=2)}

For each failure or inefficiency, identify:
1. Which tool description was ambiguous
2. What parameter naming caused confusion
3. Where the response format led the model astray
4. Specific text changes to fix each issue

Output as a structured list of changes to make.
"""
```

**What Claude typically catches:**
- Tool descriptions that overlap, causing selection confusion
- Parameter names that don't match how the model thinks about the concept
- Missing examples in descriptions that would prevent common mistakes
- Response formats that bury important information
- Error messages that don't provide actionable recovery guidance

**Iterate until stable:** Repeat the prototype -> evaluate -> refine cycle until held-out test set performance stabilizes. Typically 3-5 rounds are enough.

**Source:** [Anthropic — Writing effective tools for AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents); [modelcontextprotocol.io — writing effective tools](https://modelcontextprotocol.io)
