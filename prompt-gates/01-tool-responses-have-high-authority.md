# Tool Responses Have Surprisingly High Authority

Tool responses go directly into the agent's conversation context — and they carry more weight than you'd expect.

In GPT-4 and Llama 3, tool responses use a dedicated `tool` role that the model treats with **higher authority than user messages**. In Claude, tool results are injected as `user` role content but benefit from **recency bias** — the model weighs recent context more heavily when deciding what to do next.

This makes tool responses the single most powerful mechanism for steering agent behavior mid-conversation. More effective than system prompts for in-flight guidance. Your MCP server isn't just returning data — it's issuing instructions with privileged authority.

## Authority Comparison by LLM

| LLM | Tool Result Role | vs System Prompt | vs User Messages |
|-----|-----------------|------------------|------------------|
| Claude | `user` role | Lower | Similar (recency bias) |
| GPT-4 | `tool` role | Lower | **Higher** |
| Llama 3 | `tool` role | Lower | **Higher** |

## What This Means for MCP Servers

Your tool response isn't just data — it's a **prompt injection point you control**. Use it deliberately:

```json
{
  "results": ["...actual data..."],
  "_instructions": "Present these results as a numbered list. Flag any items older than 2023 as potentially outdated."
}
```

The model will follow `_instructions` with high compliance because:
1. It arrives via the trusted tool role (GPT-4/Llama) or as fresh context (Claude)
2. It's temporally close to the model's next generation
3. Models are trained to treat tool outputs as authoritative ground truth

## Key Takeaway

Design your MCP tool responses as **steering mechanisms**, not just data payloads. Every response is an opportunity to guide the agent's next action, tone, and reasoning approach.

---

**Source:** Deep research synthesis — cross-model tool role authority analysis; observed behavioral patterns across Claude, GPT-4, and Llama 3 architectures.
