# Ask the LLM to Evaluate Your Server's Usability

The LLM is your user. Ask it directly: "How could we improve the usability and tool documentation?" and "How would you design this if there were no constraints?" AI UX is massively underappreciated — and trivially easy to measure by simply asking.

**The feedback loop:**
```
You are using these MCP tools: [paste tool schemas]

1. Which tool descriptions are confusing or ambiguous?
2. Which parameter names don't match how you'd naturally think about the task?
3. Where do you feel uncertain about what a tool does or when to use it?
4. If you could redesign this interface from scratch, what would you change?
5. Here are two versions of a description — which is more pleasant to use?
```

**A/B test descriptions in-context:**
```python
# Present two description variants and ask the model to pick
prompt = """
Version A: "Searches the database for matching records."
Version B: "Find records matching your criteria. Returns up to 50 results
sorted by relevance. Use filters to narrow: status, date_range, owner."

Which description would help you use this tool more effectively? Why?
"""
```

**What this catches that evals miss:**
- Subtle friction: the model "works around" a bad description instead of failing
- Missing context: the model wishes it knew what format a parameter expects
- Naming mismatch: the model's mental model doesn't match your parameter names
- Ambiguous boundaries: the model can't tell when to use tool A vs tool B

**When to run this:**
- After every significant schema change
- When you notice the model using workarounds or extra tool calls
- Before finalizing tool descriptions for production

**Key insight:** The model won't complain unprompted — it adapts silently. You have to explicitly ask for critique. Treat it like a user interview, not a bug report.

**Source:** [u/jimmiebfulton on r/mcp](https://reddit.com/r/mcp) (260 upvotes thread); [u/jbr on r/mcp](https://reddit.com/r/mcp)
