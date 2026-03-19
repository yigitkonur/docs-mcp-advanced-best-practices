# Zero-Trust Policy Gateway

Wrap your MCP server in a policy execution gateway that evaluates every tool call against a declarative policy before dispatch. The gateway can validate permissions, enforce constraints, and sign job tickets — all without modifying individual tool handlers.

## Architecture

```
LLM Client → Policy Gateway → Tool Handler
                   ↓
           policy.json (rules)
           HMAC ticket signing
           audit log
```

## Implementation

```typescript
import { createHmac } from "crypto";

interface Policy {
  allowed_tools: string[];
  per_tool: Record<string, {
    max_params?: Record<string, unknown>;
    require_context?: string[];  // Required session fields
    require_role?: string;
  }>;
}

async function loadPolicy(): Promise<Policy> {
  return JSON.parse(await fs.readFile("policy.json", "utf-8"));
}

class ExecutionGateway {
  private policy!: Policy;

  async init() { this.policy = await loadPolicy(); }

  async authorize(toolName: string, params: unknown, context: SessionContext): Promise<void> {
    if (!this.policy.allowed_tools.includes(toolName)) {
      throw new Error(`Tool "${toolName}" is not in the allowed_tools list.`);
    }

    const rule = this.policy.per_tool[toolName];
    if (rule?.require_role && context.role !== rule.require_role) {
      throw new Error(`Tool "${toolName}" requires role "${rule.require_role}". Current: "${context.role}".`);
    }
    if (rule?.require_context) {
      for (const field of rule.require_context) {
        if (!context[field]) throw new Error(`Missing required session context: ${field}`);
      }
    }
  }

  signJobTicket(toolName: string, params: unknown): string {
    const payload = JSON.stringify({ toolName, params, ts: Date.now() });
    return createHmac("sha256", process.env.GATEWAY_SECRET!).update(payload).digest("hex");
  }
}

// Wrap server dispatch
const gateway = new ExecutionGateway();
await gateway.init();

server.use(async (req, next) => {
  await gateway.authorize(req.tool, req.params, req.session);
  const ticket = gateway.signJobTicket(req.tool, req.params);
  auditLog.write({ ticket, ...req.session, tool: req.tool });
  return next();
});
```

## policy.json Example

```json
{
  "allowed_tools": ["read_file", "list_directory", "create_issue", "deploy_staging"],
  "per_tool": {
    "deploy_staging": {
      "require_role": "deployer",
      "require_context": ["project_id", "branch"]
    },
    "create_issue": {
      "require_context": ["github_token"]
    }
  }
}
```

**Why it matters:** A centralized policy gateway decouples authorization logic from tool handlers, allows policy changes without code deployments, and provides a tamper-evident audit trail for all tool invocations.

**Source:** [Stacklok](https://stacklok.com) policy gateway pattern; zero-trust design principles for LLM tool execution
