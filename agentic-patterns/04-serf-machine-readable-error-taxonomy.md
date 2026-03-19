# SERF: Machine-Readable Error Taxonomy for Agents

Return errors in a structured, machine-readable format with a fixed taxonomy so the LLM can programmatically decide whether to retry, switch tools, escalate, or stop — rather than parsing free-text error messages.

The SERF framework (Structured Error Recovery Framework) was proposed in arXiv:2603.13417 ("Bridging Protocol and Production: Design Patterns for Deploying AI Agents with Model Context Protocol", Srinivasan, 2026) as a standard for agentic error handling. Its core insight: an agent that reads `"retryable": true, "retry_after_ms": 2000` can act on it reliably. An agent that reads "Please try again in a moment" must interpret natural language under uncertainty.

## SERF Error Structure

```typescript
type SERFCategory =
  | "INVALID_PARAMS"      // Schema/validation failure — do not retry as-is
  | "RESOURCE_NOT_FOUND"  // Target doesn't exist — check ID, don't retry
  | "RESOURCE_EXHAUSTED"  // Quota/rate limit — retry after delay
  | "PERMISSION_DENIED"   // Auth failure — escalate or try different credentials
  | "RATE_LIMITED"        // Explicit rate limit — retry after retry_after_ms
  | "INTERNAL_ERROR";     // Server-side failure — retry with backoff

interface SERFError {
  error_category: SERFCategory;
  message: string;           // Human-readable description
  retryable: boolean;
  retry_after_ms?: number;   // Set for RATE_LIMITED and RESOURCE_EXHAUSTED
  suggested_actions: string[]; // Machine-readable next steps
  param_hints?: Record<string, string>; // For INVALID_PARAMS: field -> fix hint
}

function serfError(category: SERFCategory, message: string, opts: Partial<SERFError> = {}) {
  const defaults: Record<SERFCategory, Partial<SERFError>> = {
    INVALID_PARAMS:     { retryable: false, suggested_actions: ["fix_params_and_retry"] },
    RESOURCE_NOT_FOUND: { retryable: false, suggested_actions: ["search_for_resource", "list_resources"] },
    RESOURCE_EXHAUSTED: { retryable: true,  suggested_actions: ["retry_after_delay", "reduce_request_size"] },
    PERMISSION_DENIED:  { retryable: false, suggested_actions: ["check_permissions", "use_different_credentials"] },
    RATE_LIMITED:       { retryable: true,  suggested_actions: ["wait_and_retry"] },
    INTERNAL_ERROR:     { retryable: true,  suggested_actions: ["retry_with_backoff", "report_issue"] },
  };
  return {
    content: [{ type: "text", text: JSON.stringify({ ...defaults[category], ...opts, error_category: category, message }) }],
    isError: true,
  };
}

// Usage:
return serfError("RATE_LIMITED", "GitHub API rate limit exceeded", {
  retry_after_ms: 60_000,
  suggested_actions: ["wait_60s_then_retry"],
});
```

## Agent Behavior Contract

Document in your server's README:
> Errors always include `retryable` (bool) and `error_category` (SERF taxonomy). Agents should: if `retryable=false`, stop and report to user; if `RATE_LIMITED`, wait `retry_after_ms`; if `INVALID_PARAMS`, inspect `param_hints`.

**Why it matters:** LLMs retry errors inconsistently when errors are natural-language prose. SERF gives agents deterministic retry logic, preventing both premature give-ups and infinite retry loops.

**Source:** [arXiv:2603.13417](https://arxiv.org/abs/2603.13417) — "Bridging Protocol and Production" (Srinivasan, 2026)
