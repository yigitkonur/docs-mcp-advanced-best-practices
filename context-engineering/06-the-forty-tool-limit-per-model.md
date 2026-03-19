# Each LLM Has a Tool Limit Beyond Which Accuracy Collapses

Registering more tools does not make a model more capable — past a model-specific threshold it makes it less capable. Performance degradation is not linear; it hits a cliff. Models start forgetting tools exist, hallucinating tool names that were never registered, or calling the wrong tool for the task. The threshold varies significantly by model and is lower than most developers expect.

Community measurement and vendor documentation converge on these per-model profiles:

| Model | Sweet Spot | Hard Degradation | Observed Behaviour |
|---|---|---|---|
| Claude 3.5 / 4 | 20–30 tools | ~50 tools | Ignores tools beyond position 30; accuracy drops ~15% past 128k context |
| GPT-4 / 4.1 | 15–20 tools | 128 (API hard limit) | Hallucinated tool calls appear above 50; latency roughly doubles above 15 |
| Gemini 1.5 Pro | 10 tools | ~100 tools | Optimised for single-call patterns; measurable quality degradation above 10 |

OpenAI explicitly recommends a maximum of 20 tools in their function-calling documentation. u/Brief-Horse-454 summarises the community consensus bluntly: "There is no standard solution. Disable the ones you don't need." The implication is that tool roster management is the developer's responsibility, not something the model handles gracefully on its own.

```typescript
// Tool group registry — expose only tools relevant to the current task
const TOOL_GROUPS: Record<string, string[]> = {
  "code-review":   ["read_file", "search_code", "get_diff", "add_comment"],
  "deployment":    ["run_pipeline", "get_logs", "rollback", "set_env"],
  "data-analysis": ["query_db", "export_csv", "run_notebook", "plot_chart"],
};

function getToolsForTask(allTools: Map<string, Tool>, hint: string): Tool[] {
  for (const [group, names] of Object.entries(TOOL_GROUPS)) {
    if (hint.toLowerCase().includes(group.replace("-", " "))) {
      return names.map(n => allTools.get(n)!).filter(Boolean);
    }
  }
  return [...allTools.values()].slice(0, 20); // safe universal cap
}

// Progressive disclosure — start with 3 always-on tools, expand on request
const ALWAYS_ON = ["help", "search", "read_file"];
const perModelLimit: Record<string, number> = { claude: 30, "gpt-4": 20, gemini: 10 };

function enableTool(active: string[], model: string, name: string): boolean {
  const limit = perModelLimit[model] ?? 20;
  if (active.length >= limit || active.includes(name)) return false;
  active.push(name);
  return true;
}
```

**Why it matters:** Every tool past the model's sweet spot adds token cost and degrades routing accuracy — tool roster hygiene is a first-class correctness concern, not just an optimisation.

**Source:** r/mcp u/Rotemy-x10 (271 upvotes); u/Brief-Horse-454 community consensus; OpenAI function-calling docs (max 20 tools recommendation); Claude model card context notes.
