# Tokenize PII Before Model Exposure

Never rely on prompt-level instructions to prevent PII leakage. Models don't guarantee compliance with "don't output emails" rules. Instead, replace PII with deterministic tokens before the data reaches the model, and untokenize in the response path.

**Pattern: Bidirectional PII tokenization**

```python
import re
from collections import defaultdict

class PIITokenizer:
    PATTERNS = {
        "EMAIL": r"[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}",
        "PHONE": r"\+?1?[-.\s]?\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}",
        "SSN":   r"\b\d{3}-\d{2}-\d{4}\b",
    }
    def __init__(self):
        self._map, self._reverse = {}, {}
        self._counters = defaultdict(int)

    def tokenize(self, text: str) -> str:
        for pii_type, pattern in self.PATTERNS.items():
            for match in re.finditer(pattern, text):
                val = match.group()
                if val not in self._reverse:
                    self._counters[pii_type] += 1
                    token = f"[{pii_type}_{self._counters[pii_type]}]"
                    self._map[token], self._reverse[val] = val, token
                text = text.replace(val, self._reverse[val])
        return text

    def untokenize(self, text: str) -> str:
        for token, original in self._map.items():
            text = text.replace(token, original)
        return text
```

**Usage:** Call `tokenizer.tokenize(data)` in every tool response before returning to the model. The model sees `[EMAIL_1]` and `[PHONE_1]` instead of real data. After the model responds, call `untokenize()` before displaying to the user.

**Also implement response interceptors:** Scan model completions for PII patterns that leaked through. This is your second line of defense — catch anything the tokenizer missed.

**Source:** [Anthropic — Code Execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp); [u/hasmcp on r/mcp](https://reddit.com/r/mcp)
