# Shrink Tool Responses as Context Fills Up

A tool that returns full pagination, rich metadata, and verbose field names makes sense early in a session when the context window is mostly empty. That same tool returning the same payload near the end of a long session can push the model into truncation territory, causing it to lose earlier reasoning or system instructions. The fix is to make verbosity a function of how much context remains — not a static contract.

Define three tiers keyed to the fraction of the context window still available. Above 70% remaining: return everything — full records, all fields, pagination metadata. Between 40–70%: switch to summary mode — a count, the top 5 items, and a `detail_available` flag with a hint to call `get_details(id)`. Below 40%: return only the count and a directive to narrow the query.

The implementation requires tracking session token consumption. Store a running `session_tokens` counter and compare against the model's `total_window`.

```python
from dataclasses import dataclass

CONTEXT_THRESHOLDS = {"full": 0.70, "summary": 0.40}

@dataclass
class ContextBudget:
    session_tokens: int
    total_window: int

    @property
    def remaining_fraction(self) -> float:
        return max(0.0, 1.0 - self.session_tokens / self.total_window)

    @property
    def tier(self) -> str:
        r = self.remaining_fraction
        if r >= CONTEXT_THRESHOLDS["full"]:
            return "full"
        if r >= CONTEXT_THRESHOLDS["summary"]:
            return "summary"
        return "minimal"

def tiered_response(records: list[dict], budget: ContextBudget) -> dict:
    tier = budget.tier

    if tier == "full":
        return {"results": records, "total": len(records)}

    if tier == "summary":
        return {
            "total": len(records),
            "sample": records[:5],
            "detail_available": True,
            "hint": f"Showing 5 of {len(records)}. Call get_details(id) for full records.",
        }

    # minimal — only count + directive
    return {
        "total": len(records),
        "hint": "Context low. Narrow your query or call get_details(id) for a specific record.",
    }
```
**Why it matters:** A tool that blindly returns 10,000 rows at turn 40 of a 50-turn session will silently degrade model performance; a tiered response gives the model a graceful off-ramp to request only what it needs.

**Source:** Production patterns from context-aware MCP server design; community reports on context exhaustion from [r/ClaudeAI](https://reddit.com/r/ClaudeAI) and [r/mcp](https://reddit.com/r/mcp)
