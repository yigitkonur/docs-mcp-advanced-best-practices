# Use PromptFoo for Systematic Tool Selection Evals

The only way to know if tool descriptions actually work is to measure tool selection accuracy across models. Use PromptFoo or similar frameworks to test: given a user intent, does the model pick the right tool with the right parameters?

**Basic PromptFoo config:**
```yaml
# promptfoo.yaml
prompts:
  - "You have access to these tools: {{tools}}. User request: {{request}}"

providers:
  - openai:gpt-4o
  - anthropic:messages:claude-sonnet-4-20250514
  - ollama:llama3.1

tests:
  # Positive case: correct tool should be selected
  - vars:
      request: "Find all orders from last week"
      tools: "{{tool_schemas}}"
    assert:
      - type: contains
        value: "search_orders"
      - type: contains
        value: "date_range"

  # Negative case: tool should NOT be selected
  - vars:
      request: "What's the weather today?"
      tools: "{{tool_schemas}}"
    assert:
      - type: not-contains
        value: "search_orders"
```

**What to eval:** selection accuracy, parameter correctness, ambiguity handling (two tools could apply), and refusal (tools that shouldn't be selected).

**Run it:**
```bash
npx promptfoo@latest eval
npx promptfoo@latest view  # opens comparison dashboard
```

**Cross-model gotchas:** Claude is conservative (asks before calling), GPT hallucinates param values, smaller models (Llama, Mistral) struggle with >10 tools. Run evals in CI — a tweak that helps Claude can break Llama.

**Source:** [u/AchillesDev on r/mcp](https://reddit.com/r/mcp); [Anthropic — Writing effective tools for AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents); [PromptFoo MCP integration](https://www.promptfoo.dev/docs/integrations/mcp/)
