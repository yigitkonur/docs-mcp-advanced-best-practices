# Build Circuit Breakers for Agent Loop Detection

Agents can't detect their own loops. From the LLM's perspective, every retry is a fresh attempt with renewed optimism. You must detect and break loops *outside* the LLM's decision-making.

**The problem:**
An agent calls `update_record(id=42, status="active")` → gets a validation error → "fixes" the call → sends the exact same parameters → same error → repeats indefinitely. The LLM genuinely believes each attempt is different.

**Solution — hash-based state tracking:**
```python
import hashlib
from collections import defaultdict
from time import time

class LoopBreaker:
    def __init__(self, max_repeats=3, window_seconds=120):
        self.max_repeats = max_repeats
        self.window = window_seconds
        self.seen: dict[str, list[float]] = defaultdict(list)

    def check(self, tool_name: str, params: dict) -> str | None:
        state = hashlib.sha256(
            f"{tool_name}:{sorted(params.items())}".encode()
        ).hexdigest()[:16]

        now = time()
        # Prune old entries outside the window
        self.seen[state] = [t for t in self.seen[state] if now - t < self.window]
        self.seen[state].append(now)

        if len(self.seen[state]) >= self.max_repeats:
            return (
                f"Loop detected: {tool_name} called {self.max_repeats} times "
                f"with identical parameters in {self.window}s. "
                f"Stop retrying and ask the user for guidance."
            )
        return None

loop_breaker = LoopBreaker()
```

**Wire it into your tool handler:**
```python
def handle_tool_call(tool_name: str, params: dict):
    loop_msg = loop_breaker.check(tool_name, params)
    if loop_msg:
        return {"content": [{"type": "text", "text": loop_msg}], "isError": True}

    return execute_tool(tool_name, params)
```

**What to hash:** Include the full execution state — tool name, all parameters, and any file references. Don't just hash the tool name; `search("foo")` and `search("bar")` are different calls. But `search("foo")` three times in a row is a loop.

**Tuning:** 3 repeats within 2 minutes is a reasonable default. Tighten for fast tools (API lookups), loosen for tools with legitimate retries (file uploads with transient failures).

**Source:** u/Main_Payment_6430, r/AI_Agents; circuit breaker pattern via Pragmatic Engineer
