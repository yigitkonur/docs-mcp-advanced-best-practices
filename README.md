# MCP Server Best Practices

**112 battle-tested patterns for building MCP servers that LLMs actually use correctly.**

Synthesized from Anthropic engineering docs, 30+ Reddit threads, GitHub implementations, arXiv papers, and production experience.

`112 patterns` | `15 categories` | `Tested across Claude, GPT-4, Gemini`

> **What is MCP?** The [Model Context Protocol](https://modelcontextprotocol.io/) is an open standard that lets AI assistants (Claude, GPT, Gemini, etc.) connect to external tools and data sources through a unified interface. MCP servers expose tools, resources, and prompts that LLMs call during conversations. Designing these servers well is the difference between an AI that fumbles through 15 tool calls and one that nails the task in 2.

---

## Quick Reference Card

The most impactful decisions when building an MCP server, distilled into one table:

| Decision | Recommendation | Evidence |
|---|---|---|
| **Tool count** | Keep under 40 per server | Performance cliff measured across Claude, GPT-4, Gemini |
| **Description length** | 20-50 words core; XML tags for structure | >100 words partially ignored; Gemini needs <75 tokens |
| **Schema params** | Max 6 top-level, flat only | 98% parse success flat vs <70% nested 12+ params |
| **Enum vs string** | Always use enums for known values | Valid-call rate: 60% (string) -> 95%+ (enum) |
| **Response format** | Structured data + `_next_steps` + metadata | Every response IS a prompt to the LLM |
| **Error format** | `isError:true` + `corrected_call` + `recovery_tool` | Structured errors: 62% -> 89% task completion |
| **Examples in descriptions** | Always include one good + one bad | +25-35% accuracy improvement |
| **Transport** | stdio (local), Streamable HTTP (remote) | SSE is deprecated in MCP spec |
| **Token tax** | ~15% of Claude Code input = tool definitions | Each tool costs 50-150 tokens per message |

---

## Start Here

Pick the path that matches your situation:

### New to MCP
1. Read the [comprehensive master guide](MCP-SERVER-MASTERY.md) for full context
2. Start with [Design Around User Intent, Not API Endpoints](tool-design/01-design-around-intent-not-endpoints.md)
3. Then [Treat Every Tool Response as a Prompt](tool-responses/01-treat-every-response-as-a-prompt.md)
4. Then [Use isError in the Result Object](error-handling/01-use-iserror-in-result-not-protocol-errors.md)

### Experienced MCP Builder
1. [Tool Descriptions Eat ~15% of Your Context Budget](context-engineering/01-tool-description-token-tax.md) -- understand the cost model
2. [Use Meta-Tools for Large Tool Catalogs](progressive-discovery/01-meta-tools-for-large-catalogs.md) -- scale beyond 40 tools
3. [Cross-Model Portable Schema Rules](schema-design/08-cross-model-portable-schema-rules.md) -- multi-model compatibility
4. [Build Circuit Breakers for Agent Loop Detection](error-handling/08-circuit-breakers-for-loop-detection.md) -- production resilience

### Security-Focused
1. [Sanitize User-Generated Content in Responses](security/01-sanitize-user-content-in-responses.md) -- prompt injection defense
2. [Prevent Cross-Tool Hijacking via Shared Context](security/05-cross-tool-hijacking-via-shared-context.md) -- the attack most people miss
3. [Use Delegated Permissions, Not Superuser](security/02-use-delegated-permissions-not-superuser.md) -- least privilege
4. [Zero-Trust Policy Gateway](composition/07-zero-trust-policy-gateway.md) -- enforce policies at the gateway

### Performance-Focused
1. [The Code Execution Pattern Saves 98% of Tool-Related Tokens](context-engineering/02-code-execution-lazy-tool-discovery.md)
2. [Session Pooling for 10x Throughput](session-and-state/04-session-pooling-for-10x-throughput.md)
3. [Transport Choice Makes or Breaks Performance](transport-and-ops/08-transport-performance-benchmarks.md)
4. [Multi-Level Caching for MCP Responses](transport-and-ops/10-multi-level-caching-strategies.md)

---

## Category Explorer

Each pattern is a standalone guide with rationale, code examples, and pitfalls to avoid.

<!-- ====== TOOL DESIGN ====== -->
<details>
<summary><strong>Tool Design</strong> -- Intent-based design, workflow consolidation, planner tools <code>(8 patterns)</code></summary>

| # | Pattern |
|---|---|
| 1 | [Design Around User Intent, Not API Endpoints](tool-design/01-design-around-intent-not-endpoints.md) |
| 2 | [Be a Smart Database, Not a Smart Analyst](tool-design/02-smart-database-not-smart-analyst.md) |
| 3 | [Consolidate Multi-Step Workflows Into Single Atomic Tools](tool-design/03-consolidate-workflows-into-single-tools.md) |
| 4 | [Use a Planner Tool to Teach the Model Your Workflow](tool-design/04-use-a-planner-tool-for-complex-workflows.md) |
| 5 | [Expose a Code Execution Sandbox for Batch Operations](tool-design/05-code-mode-for-batch-operations.md) |
| 6 | [Design Tool Workflows for 3-5 Calls, Not 20+](tool-design/06-design-for-smaller-models.md) |
| 7 | [CRUD: Combined Tool vs Separate Tools Decision](tool-design/07-crud-combined-vs-separate-decision.md) |
| 8 | [The Toolhost/Facade Pattern for Many Related Operations](tool-design/08-toolhost-facade-for-many-operations.md) |

</details>

<!-- ====== TOOL DESCRIPTIONS ====== -->
<details>
<summary><strong>Tool Descriptions</strong> -- XML structure, naming, examples, verbosity optimization <code>(11 patterns)</code></summary>

| # | Pattern |
|---|---|
| 1 | [Use XML Tags to Separate Purpose from Instructions](tool-descriptions/01-use-xml-tags-to-structure-descriptions.md) |
| 2 | [Write Tool Descriptions Like Briefing a New Hire](tool-descriptions/02-write-descriptions-like-briefing-a-new-hire.md) |
| 3 | [Namespace Tools to Prevent Collisions Across Servers](tool-descriptions/03-namespace-tools-to-prevent-collisions.md) |
| 4 | [Use Unambiguous Parameter Names](tool-descriptions/04-use-unambiguous-parameter-names.md) |
| 5 | [Enforce Strict Input Schemas with Enums and Constraints](tool-descriptions/05-enforce-strict-input-schemas.md) |
| 6 | [Front-Load Verb + Resource in the First Five Words](tool-descriptions/06-front-load-verb-resource-in-first-five-words.md) |
| 7 | [Include Exclusionary Guidance -- When NOT to Use a Tool](tool-descriptions/07-include-exclusionary-guidance.md) |
| 8 | [Truth in the Schema, Hints in the Description](tool-descriptions/08-truth-in-schema-hints-in-description.md) |
| 9 | [Use the `instructions` Field as Your Server's skills.md](tool-descriptions/09-use-instructions-field-as-skills-md.md) |
| 10 | [Add Correct AND Incorrect Call Examples](tool-descriptions/10-add-correct-and-incorrect-call-examples.md) |
| 11 | [Over-Verbose Descriptions Reduce Tool Call Rate](tool-descriptions/11-over-verbose-descriptions-reduce-call-rate.md) |

</details>

<!-- ====== TOOL RESPONSES ====== -->
<details>
<summary><strong>Tool Responses</strong> -- Response-as-prompt, format enums, YAML/TSV, annotations <code>(10 patterns)</code></summary>

| # | Pattern |
|---|---|
| 1 | [Treat Every Tool Response as a Prompt to the Model](tool-responses/01-treat-every-response-as-a-prompt.md) |
| 2 | [Add a Response Format Enum to Every Data-Returning Tool](tool-responses/02-add-response-format-enum.md) |
| 3 | [Return Semantic Identifiers, Not Opaque UUIDs](tool-responses/03-return-semantic-identifiers-not-uuids.md) |
| 4 | [Prepend Truncation Guidance When Results Are Cut Off](tool-responses/04-prepend-truncation-guidance.md) |
| 5 | [Include Next-Step Hints in Successful Responses](tool-responses/05-include-next-step-hints-in-success.md) |
| 6 | [Default to YAML Over JSON for LLM-Consumable Responses](tool-responses/06-use-yaml-over-json-for-readability.md) |
| 7 | [Use TSV for Tabular Data to Save 30-40% of Tokens](tool-responses/07-tsv-for-tabular-data-saves-tokens.md) |
| 8 | [Use Content Annotations to Separate Audiences](tool-responses/08-content-annotations-for-audience.md) |
| 9 | [Declare Output Schemas and Return structuredContent](tool-responses/09-output-schemas-with-structured-content.md) |
| 10 | [Put Example Responses in Tool Descriptions](tool-responses/10-put-example-responses-in-descriptions.md) |

</details>

<!-- ====== SCHEMA DESIGN ====== -->
<details>
<summary><strong>Schema Design</strong> -- Type coercion, flat schemas, enums, cross-model portability <code>(8 patterns)</code></summary>

| # | Pattern |
|---|---|
| 1 | [Use `z.coerce` and `z.preprocess` for Type Safety](schema-design/01-use-coerce-and-preprocess-for-type-safety.md) |
| 2 | [Pair Regex Validation with Human-Readable Examples](schema-design/02-regex-with-human-readable-examples.md) |
| 3 | [Keep Schemas Flat, Under 6 Parameters](schema-design/03-keep-schemas-flat-under-six-params.md) |
| 4 | [Accept Flexible Formats, Normalize Server-Side](schema-design/04-accept-flexible-formats-normalize-server-side.md) |
| 5 | [Describe Each Enum Value Inline](schema-design/05-describe-each-enum-value-inline.md) |
| 6 | [Break Tools Over 40 Parameters](schema-design/06-break-tools-over-forty-parameters.md) |
| 7 | [Safe Tool Name Characters Only](schema-design/07-safe-tool-name-characters-only.md) |
| 8 | [Cross-Model Portable Schema Rules](schema-design/08-cross-model-portable-schema-rules.md) |

</details>

<!-- ====== ERROR HANDLING ====== -->
<details>
<summary><strong>Error Handling</strong> -- isError patterns, educational errors, circuit breakers <code>(9 patterns)</code></summary>

| # | Pattern |
|---|---|
| 1 | [Use isError in the Result Object, Not Protocol-Level Errors](error-handling/01-use-iserror-in-result-not-protocol-errors.md) |
| 2 | [Make Error Messages Educational, Not Technical](error-handling/02-make-errors-educational.md) |
| 3 | [Embed Recovery Tool Names Directly in Error Messages](error-handling/03-embed-recovery-tool-names-in-errors.md) |
| 4 | [Include Retry Limits and Fallback Instructions in Errors](error-handling/04-include-retry-limits-in-error-guidance.md) |
| 5 | [Let Errors Handle the Long Tail Instead of Bloating Descriptions](error-handling/05-errors-reduce-description-bloat.md) |
| 6 | [Distinguish "Prevented" from "Failed" in Error Framing](error-handling/06-distinguish-prevented-from-failed.md) |
| 7 | [Avoid "Not Found" Phrasing -- Return What Exists Instead](error-handling/07-avoid-not-found-phrasing.md) |
| 8 | [Build Circuit Breakers for Agent Loop Detection](error-handling/08-circuit-breakers-for-loop-detection.md) |
| 9 | [Return Normalized Inputs in Every Response](error-handling/09-return-normalized-inputs-in-response.md) |

</details>

<!-- ====== SECURITY ====== -->
<details>
<summary><strong>Security</strong> -- Prompt injection defense, RBAC, tool annotations, PII tokenization <code>(8 patterns)</code></summary>

| # | Pattern |
|---|---|
| 1 | [Sanitize User-Generated Content in Tool Responses](security/01-sanitize-user-content-in-responses.md) |
| 2 | [Use Delegated Permissions, Not a Shared Superuser Token](security/02-use-delegated-permissions-not-superuser.md) |
| 3 | [Use Tool Annotations to Signal Safety Properties](security/03-tool-annotations-for-safety-hints.md) |
| 4 | [Always Require Human Confirmation for Destructive Operations](security/04-require-confirmation-for-side-effects.md) |
| 5 | [Prevent Cross-Tool Hijacking via Shared Context](security/05-cross-tool-hijacking-via-shared-context.md) |
| 6 | [Sign and Verify Tool Schemas Before Execution](security/06-sign-and-verify-tool-schemas.md) |
| 7 | [Tokenize PII Before Model Exposure](security/07-tokenize-pii-before-model-exposure.md) |
| 8 | [Use OAuth2 with Granular Scopes for External MCP Servers](security/08-oauth2-scopes-for-external-apis.md) |

</details>

<!-- ====== CONTEXT ENGINEERING ====== -->
<details>
<summary><strong>Context Engineering</strong> -- Token budgets, lazy discovery, tiered verbosity <code>(6 patterns)</code></summary>

| # | Pattern |
|---|---|
| 1 | [Tool Descriptions Eat ~15% of Your Context Budget](context-engineering/01-tool-description-token-tax.md) |
| 2 | [The Code Execution Pattern Saves 98% of Tool-Related Tokens](context-engineering/02-code-execution-lazy-tool-discovery.md) |
| 3 | [Shrink Tool Responses as Context Fills Up](context-engineering/03-tiered-verbosity-by-context-remaining.md) |
| 4 | [Strip ANSI Codes and Progress Bars Before Returning CLI Output](context-engineering/04-strip-cli-decoration-from-responses.md) |
| 5 | [Use the Right Data Delivery Strategy for Each Tool Type](context-engineering/05-pagination-vs-truncation-vs-streaming-decision.md) |
| 6 | [Each LLM Has a Tool Limit Beyond Which Accuracy Collapses](context-engineering/06-the-forty-tool-limit-per-model.md) |

</details>

<!-- ====== PROGRESSIVE DISCOVERY ====== -->
<details>
<summary><strong>Progressive Discovery</strong> -- Meta-tools, semantic search, session unlocking, visibility <code>(8 patterns)</code></summary>

| # | Pattern |
|---|---|
| 1 | [Use Meta-Tools for Large Tool Catalogs (list/describe/execute)](progressive-discovery/01-meta-tools-for-large-catalogs.md) |
| 2 | [Use Semantic Search with Embeddings for 100+ Tools](progressive-discovery/02-semantic-search-for-100-plus-tools.md) |
| 3 | [Session-Based Progressive Tool Unlocking](progressive-discovery/03-session-based-tool-unlocking.md) |
| 4 | [Four-Stage Progressive Disclosure Pattern](progressive-discovery/04-four-stage-progressive-disclosure.md) |
| 5 | [Dynamic Tool List Changes Nuke the KV/Prefix Cache](progressive-discovery/05-dynamic-tool-list-nukes-prefix-cache.md) |
| 6 | [Use `notifications/tools/list_changed` for Dynamic Tooling](progressive-discovery/06-use-tools-changed-notification.md) |
| 7 | [FastMCP Visibility Transforms](progressive-discovery/07-fastmcp-visibility-transforms.md) |
| 8 | [Graceful Fallback for Hidden Tools](progressive-discovery/08-graceful-fallback-for-hidden-tools.md) |

</details>

<!-- ====== AGENTIC PATTERNS ====== -->
<details>
<summary><strong>Agentic Patterns</strong> -- Dependency chains, HATEOAS, guard tools, workflow stages <code>(6 patterns)</code></summary>

| # | Pattern |
|---|---|
| 1 | [Parameter Dependency Chain](agentic-patterns/01-parameter-dependency-chain.md) |
| 2 | [HATEOAS: State-Based Available Actions](agentic-patterns/02-hateoas-state-based-available-actions.md) |
| 3 | [Guard Tools: Precondition Validation via Boolean Params](agentic-patterns/03-guard-tools-precondition-boolean-params.md) |
| 4 | [SERF: Machine-Readable Error Taxonomy for Agents](agentic-patterns/04-serf-machine-readable-error-taxonomy.md) |
| 5 | [Shared Context: Append-Only Event Log for Multi-Agent Coordination](agentic-patterns/05-shared-context-append-only-log.md) |
| 6 | [Server-Enforced Workflow Stages](agentic-patterns/06-server-enforced-workflow-stages.md) |

</details>

<!-- ====== COMPOSITION ====== -->
<details>
<summary><strong>Composition</strong> -- Gateways, proxies, OpenAPI generation, provider-transform <code>(7 patterns)</code></summary>

| # | Pattern |
|---|---|
| 1 | [Use a Meta-Server for Cross-Cutting Concerns](composition/01-meta-server-for-cross-cutting-concerns.md) |
| 2 | [Design Composable Servers That the LLM Orchestrates](composition/02-composable-server-ecosystem.md) |
| 3 | [Wrap Complex APIs in One Tool + Resource Documentation](composition/03-single-tool-with-resource-docs.md) |
| 4 | [Use the Provider + Transform Architecture for Maximum Flexibility](composition/04-provider-transform-architecture.md) |
| 5 | [Generate MCP Servers from OpenAPI Specs](composition/05-openapi-to-mcp-proxy-pattern.md) |
| 6 | [Use a Gateway/Proxy for Multi-Server Orchestration](composition/06-gateway-proxy-for-multi-server.md) |
| 7 | [Zero-Trust Policy Gateway](composition/07-zero-trust-policy-gateway.md) |

</details>

<!-- ====== PROMPT GATES ====== -->
<details>
<summary><strong>Prompt Gates</strong> -- Tool authority, XML separation, state machines <code>(4 patterns)</code></summary>

| # | Pattern |
|---|---|
| 1 | [Tool Responses Have Surprisingly High Authority](prompt-gates/01-tool-responses-have-high-authority.md) |
| 2 | [Use XML Tags to Separate Instructions from Data](prompt-gates/02-xml-separate-instructions-from-data.md) |
| 3 | [Build State Machines via Sequential Tool Responses](prompt-gates/03-state-machine-via-sequential-responses.md) |
| 4 | [Namespace Injected Instructions to Prevent Conflicts](prompt-gates/04-namespace-injected-instructions.md) |

</details>

<!-- ====== RESOURCES AND PROMPTS ====== -->
<details>
<summary><strong>Resources and Prompts</strong> -- On-demand docs, templates, workflow prompts <code>(4 patterns)</code></summary>

| # | Pattern |
|---|---|
| 1 | [Use Resources as On-Demand Documentation Referenced in Errors](resources-and-prompts/01-resources-as-on-demand-documentation.md) |
| 2 | [Use Resource Templates with URI Patterns for Scalable Data Access](resources-and-prompts/02-resource-templates-for-scalable-data.md) |
| 3 | [Use MCP Prompts for Repeatable Workflow Entry Points](resources-and-prompts/03-prompts-for-repeatable-workflows.md) |
| 4 | [When to Use Resources vs Tools](resources-and-prompts/04-when-to-use-resources-vs-tools.md) |

</details>

<!-- ====== SESSION AND STATE ====== -->
<details>
<summary><strong>Session and State</strong> -- Background tasks, session pooling, compaction <code>(5 patterns)</code></summary>

| # | Pattern |
|---|---|
| 1 | [Log Successful Tool Calls to Build a Learning Database](session-and-state/01-log-successful-calls-as-learning-database.md) |
| 2 | [Return Task IDs for Long-Running Operations](session-and-state/02-background-tasks-for-long-operations.md) |
| 3 | [Scope State to Session, Not Global Variables](session-and-state/03-scope-state-to-session-not-global.md) |
| 4 | [Session Pooling for 10x Throughput](session-and-state/04-session-pooling-for-10x-throughput.md) |
| 5 | [Conversation Compaction Priorities](session-and-state/05-conversation-compaction-priorities.md) |

</details>

<!-- ====== TESTING ====== -->
<details>
<summary><strong>Testing</strong> -- Eval-driven dev, MCP Inspector, PromptFoo, transcript refactoring <code>(6 patterns)</code></summary>

| # | Pattern |
|---|---|
| 1 | [Use Evaluation-Driven Development with Realistic Multi-Step Tasks](testing/01-evaluation-driven-development.md) |
| 2 | [Test Tools with MCP Inspector Before Involving an LLM](testing/02-test-with-mcp-inspector-before-llm.md) |
| 3 | [Feed Evaluation Transcripts Back for Auto-Refactoring](testing/03-feed-transcripts-back-for-auto-refactoring.md) |
| 4 | [Ask the LLM to Evaluate Your Server's Usability](testing/04-ask-llm-to-evaluate-server-usability.md) |
| 5 | [Use PromptFoo for Systematic Tool Selection Evals](testing/05-use-promptfoo-for-tool-selection-evals.md) |
| 6 | [Use File Hashing to Prevent Agent Reprocessing Loops](testing/06-file-hashing-prevents-reprocessing.md) |

</details>

<!-- ====== TRANSPORT AND OPS ====== -->
<details>
<summary><strong>Transport and Ops</strong> -- stdio/HTTP transport, observability, caching, K8s deployment <code>(12 patterns)</code></summary>

| # | Pattern |
|---|---|
| 1 | [Keep stdout Pure JSON-RPC -- All Logs to stderr](transport-and-ops/01-keep-stdout-clean.md) |
| 2 | [Use Streamable HTTP, Not Deprecated SSE](transport-and-ops/02-use-streamable-http-not-sse.md) |
| 3 | [Use Hot Reload During Development](transport-and-ops/03-hot-reload-during-development.md) |
| 4 | [Add Structured Observability from Day One](transport-and-ops/04-structured-observability.md) |
| 5 | [Never Call process.exit() or sys.exit() Inside Tool Handlers](transport-and-ops/05-never-process-exit-in-handlers.md) |
| 6 | [Open External Connections Inside Tool Calls, Not at Startup](transport-and-ops/06-open-connections-inside-tool-calls.md) |
| 7 | [Validate Authentication Tokens at Execution Time, Not Startup](transport-and-ops/07-validate-tokens-at-execution-not-startup.md) |
| 8 | [Transport Choice Makes or Breaks MCP Server Performance](transport-and-ops/08-transport-performance-benchmarks.md) |
| 9 | [Token Bucket Rate Limiting per Tool Category](transport-and-ops/09-token-bucket-rate-limiting.md) |
| 10 | [Multi-Level Caching for MCP Responses](transport-and-ops/10-multi-level-caching-strategies.md) |
| 11 | [Health Checks with KPI Targets](transport-and-ops/11-health-checks-kpi-targets.md) |
| 12 | [Kubernetes Deployment for MCP Servers](transport-and-ops/12-kubernetes-mcp-deployment.md) |

</details>

---

## Comprehensive Master Guide

The full guide with extended explanations, code examples, design philosophy, and anti-patterns is available in a single document:

**[MCP-SERVER-MASTERY.md](MCP-SERVER-MASTERY.md)** -- 3,200+ lines covering everything from first principles to Kubernetes deployment.

---

## Key Verified Sources

These are the authoritative sources referenced throughout the patterns:

| Source | What It Covers | Link |
|---|---|---|
| **MCP Specification** | Protocol definition, transport, capabilities | [spec.modelcontextprotocol.io](https://spec.modelcontextprotocol.io/) |
| **Anthropic: Build Effective Agents** | Agent design patterns, tool use best practices | [anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents) |
| **Anthropic: MCP Docs** | Official MCP documentation and tutorials | [modelcontextprotocol.io/docs](https://modelcontextprotocol.io/docs) |
| **Anthropic: Prompt Engineering Guide** | Prompt construction, XML tags, chain-of-thought | [docs.anthropic.com/en/docs/build-with-claude/prompt-engineering](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering) |
| **Stacklok: MCP Security** | Tool poisoning attacks, cross-tool hijacking | [blog.stacklok.com/blog/mcp-security-risks](https://blog.stacklok.com/blog/mcp-security-risks) |
| **Docker: MCP Security Best Practices** | Container-level isolation for MCP servers | [docker.com/blog/securing-mcp-servers](https://www.docker.com/blog/securing-mcp-servers/) |
| **FastMCP** | Python MCP framework with visibility transforms | [github.com/jlowin/fastmcp](https://github.com/jlowin/fastmcp) |
| **MCP Inspector** | Interactive testing and debugging tool | [github.com/modelcontextprotocol/inspector](https://github.com/modelcontextprotocol/inspector) |
| **PromptFoo** | LLM evaluation framework for tool selection testing | [promptfoo.dev](https://www.promptfoo.dev/) |
| **r/ClaudeAI, r/mcp** | Community benchmarks, production war stories | [reddit.com/r/ClaudeAI](https://www.reddit.com/r/ClaudeAI/) |

---

## How This Repo Is Organized

```
.
├── MCP-SERVER-MASTERY.md          # Full guide (start here for deep dives)
├── tool-design/                   # 8 patterns  - Intent-based design, workflow consolidation
├── tool-descriptions/             # 11 patterns - XML structure, naming, examples
├── tool-responses/                # 10 patterns - Response-as-prompt, format enums, YAML/TSV
├── schema-design/                 # 8 patterns  - Type coercion, flat schemas, enums
├── error-handling/                # 9 patterns  - isError, educational errors, circuit breakers
├── security/                      # 8 patterns  - Prompt injection, RBAC, PII tokenization
├── context-engineering/           # 6 patterns  - Token budgets, lazy discovery, verbosity
├── progressive-discovery/         # 8 patterns  - Meta-tools, semantic search, unlocking
├── agentic-patterns/              # 6 patterns  - Dependency chains, HATEOAS, guard tools
├── composition/                   # 7 patterns  - Gateways, proxies, OpenAPI generation
├── prompt-gates/                  # 4 patterns  - Tool authority, XML separation, state machines
├── resources-and-prompts/         # 4 patterns  - On-demand docs, templates, workflows
├── session-and-state/             # 5 patterns  - Background tasks, session pooling
├── testing/                       # 6 patterns  - Eval-driven dev, Inspector, PromptFoo
└── transport-and-ops/             # 12 patterns - stdio/HTTP, observability, caching, K8s
```

Each pattern file is self-contained: it explains the problem, shows the solution with code, and highlights common mistakes.
