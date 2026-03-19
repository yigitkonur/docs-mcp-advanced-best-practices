# Strip ANSI Codes and Progress Bars Before Returning CLI Output

MCP servers that wrap command-line tools inherit everything those tools emit — ANSI color codes, carriage-return-based progress bars, Unicode box-drawing characters, and alignment padding designed for human terminal rendering. None of that is meaningful to an LLM. It is pure token waste, and it is significant.

The Pare MCP project measured real reductions across common CLI tools: `docker build` (multi-stage) drops from 373 to 20 tokens (95%), `git log --stat` over 5 commits from 4,992 to 382 tokens (92%), `npm install` for 487 packages from 241 to 41 tokens (83%), `vitest run` over 28 tests from 196 to 39 tokens (80%), and `cargo build` with 2 errors from 436 to 138 tokens (68%). These are not edge cases — they are the default output of tools developers reach for constantly.

One secondary effect worth noting: the MCP tool definition itself costs tokens on every message. If your CLI-wrapping tool is called fewer than 3–5 times per session, the fixed registration cost can exceed the savings from stripping. For low-frequency tools in long sessions, this balance tips the other way and the stripping still pays off.

```python
import re
import subprocess

# Patterns to strip from CLI output
_ANSI_ESCAPE   = re.compile(r'\x1b\[[0-9;]*[mGKHF]')   # colors, cursor movement
_ANSI_ERASE    = re.compile(r'\x1b\[[0-9]*[JK]')         # screen/line erase
_CR_PROGRESS   = re.compile(r'[^\n]*\r')                  # carriage-return redraws
_SPINNER_CHARS = re.compile(r'[⠋⠙⠹⠸⠼⠴⠦⠧⠇⠏]')           # spinner frames
_BOX_DRAWING   = re.compile(r'[─│┌┐└┘├┤┬┴┼╭╮╯╰]')       # box-drawing unicode

def clean_cli_output(raw: str) -> str:
    s = _ANSI_ESCAPE.sub('', raw)
    s = _ANSI_ERASE.sub('', s)
    s = _CR_PROGRESS.sub('', s)
    s = _SPINNER_CHARS.sub('', s)
    s = _BOX_DRAWING.sub('', s)
    # Collapse runs of blank lines left by stripped progress bars
    s = re.sub(r'\n{3,}', '\n\n', s)
    return s.strip()

def run_cli_tool(cmd: list[str]) -> str:
    result = subprocess.run(
        cmd,
        capture_output=True,
        text=True,
        env={**os.environ, "NO_COLOR": "1", "TERM": "dumb"},  # suppress at source too
    )
    combined = result.stdout + ("\n" + result.stderr if result.stderr else "")
    return clean_cli_output(combined)

# Usage in an MCP tool handler
@server.tool()
async def git_log(n: int = 10) -> str:
    """Recent git log with stats."""
    return run_cli_tool(["git", "log", f"-{n}", "--stat", "--no-color"])
```

```typescript
// TypeScript equivalent — used in Node-based MCP servers
const ANSI_RE = /\x1b\[[0-9;]*[mGKHFJK]/g;
const CR_RE   = /[^\n]*\r/g;

function cleanOutput(raw: string): string {
  return raw.replace(ANSI_RE, '').replace(CR_RE, '').replace(/\n{3,}/g, '\n\n').trim();
}
```

**Why it matters:** ANSI decoration can inflate CLI output by 5–95× in token terms — stripping it is a zero-logic-change optimisation that pays back immediately on every tool call.

**Source:** github.com/Dave-London/Pare (Pare MCP project); r/ClaudeAI community benchmarks (36 upvotes); Anthropic prompt engineering docs on tool response design.
