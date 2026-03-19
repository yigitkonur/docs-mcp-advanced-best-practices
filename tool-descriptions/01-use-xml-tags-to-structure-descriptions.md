# Use XML Tags to Separate Purpose from Instructions in Tool Descriptions

Models parse tool descriptions as their only guide for when and how to use a tool. Structuring descriptions with XML tags significantly improves selection accuracy because models can distinguish the "what it's for" from the "how to use it."

```xml
<usecase>Retrieves member activity for a space, including posts, comments, and last active date. Useful for tracking activity of users.</usecase>
<instructions>Returns members sorted by total activity. Includes last 30 days by default.</instructions>
```

This pattern works because LLMs already have strong XML parsing priors from training data. A flat paragraph describing both purpose and usage gets muddled; tagged sections let the model quickly match intent to the `<usecase>` block and extract parameters from `<instructions>`.

**Why it matters:** In testing, this reduced tool selection errors noticeably compared to free-text descriptions, especially when multiple tools have overlapping capabilities.

**Source:** u/sjoti, r/mcp - "Good MCP design is understanding that every tool response is an opportunity to prompt the model" (279 upvotes). The author reports this pattern "works wonders" for separating when-to-use from how-to-use.
