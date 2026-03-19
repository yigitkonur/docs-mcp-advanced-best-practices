# Avoid "Not Found" Phrasing — Return What Exists Instead

Returning "X not found" anchors the LLM on failure. Drop the negative prefix entirely and return the available options. The model picks the closest match itself without the discouraging framing.

**Bad — negative anchor first:**
```json
{
  "content": [{"type": "text", "text": "Module 'color' not found. Available modules: fs, http, path, crypto."}],
  "isError": true
}
```

The agent fixates on "not found" and may tell the user the module doesn't exist, or attempt to install it, or hallucinate an alternative — all before considering the list you provided.

**Good — options only:**
```json
{
  "content": [{"type": "text", "text": "Available modules: fs, http, path, crypto. Select one to proceed."}],
  "isError": true
}
```

The agent reads a menu, picks the best fit, and retries. No drama.

**Implementation pattern:**
```python
MODULES = {"fs", "http", "path", "crypto", "stream"}

def load_module(name: str):
    if name not in MODULES:
        available = ", ".join(sorted(MODULES))
        # Don't say "not found" — just show what's there
        return {
            "content": [{"type": "text", "text":
                f"Available modules: {available}. "
                f"Retry load_module(name='<module>') with one of these."
            }],
            "isError": True
        }
    return {"content": [{"type": "text", "text": f"Loaded {name} successfully."}]}
```

**Same principle applies broadly:**
- Users: Don't say "User not found." Say "Matching users: alice, bob, carol."
- Files: Don't say "File missing." Say "Files in directory: config.yaml, schema.json."
- Endpoints: Don't say "Invalid action." Say "Supported actions: create, read, update, delete."

**Why this works:** LLMs are pattern-completion machines. "Not found" primes a failure-recovery pattern. A list of options primes a selection pattern. The selection pattern is far more productive.

**Source:** Community best practices from [r/mcp](https://reddit.com/r/mcp); LLM behavioral observations on negative vs. positive framing
