# Server-Enforced Workflow Stages

Maintain a `WorkflowStage` enum on the server and reject tool calls that arrive out of sequence. This enforces multi-step workflows without relying on the agent to follow documented instructions — the server itself gates progress.

This differs from `prompt-gates/03` (which uses response text to *guide* the agent through stages) — here the server *blocks* out-of-order calls with an error, making the wrong path structurally impossible.

## Implementation

```typescript
type WorkflowStage =
  | "initialized"
  | "data_loaded"
  | "analysis_complete"
  | "changes_validated"
  | "committed";

const STAGE_ORDER: WorkflowStage[] = [
  "initialized", "data_loaded", "analysis_complete", "changes_validated", "committed"
];

const sessions = new Map<string, WorkflowStage>();

function assertStage(sessionId: string, required: WorkflowStage, allowedPrevious: WorkflowStage[]) {
  const current = sessions.get(sessionId) ?? "initialized";
  if (!allowedPrevious.includes(current)) {
    const next = STAGE_ORDER[STAGE_ORDER.indexOf(current) + 1];
    throw {
      message: `Stage gate: current stage is "${current}". Must complete "${next}" before calling this tool.`,
      current_stage: current,
      required_stage: required,
      next_required_tool: stageToTool[next],
    };
  }
}

const stageToTool: Record<WorkflowStage, string> = {
  initialized: "load_data",
  data_loaded: "analyze_data",
  analysis_complete: "validate_changes",
  changes_validated: "commit_changes",
  committed: "(workflow complete)",
};

server.tool("commit_changes", "Commit validated changes. Requires validation stage.", {
  session_id: z.string(),
  dry_run: z.boolean().default(false),
}, async ({ session_id, dry_run }) => {
  try {
    assertStage(session_id, "committed", ["changes_validated"]);
  } catch (e: any) {
    return { content: [{ type: "text", text: JSON.stringify(e) }], isError: true };
  }

  const result = await commitChanges(session_id, dry_run);
  sessions.set(session_id, "committed");
  return { content: [{ type: "text", text: `Changes committed. ${result.summary}` }] };
});
```

## Why Stage Gates Matter

Without server-enforced stages, agents routinely skip prerequisite steps -- calling `deploy` before `validate`, or `commit` before `analyze`. The agent may appear confident, but it has no structural reason to follow the correct order. Stage gates eliminate this class of error entirely: an out-of-order call receives a clear error specifying the required next step, making the wrong path structurally impossible.

**Why it matters:** Documentation and prompt guidance are suggestions. Stage gates are enforcement. For consequential workflows (deployments, financial operations, data migrations), server-side stage enforcement is the only reliable safeguard.

**Source:** State machine enforcement patterns; community discussions on [r/mcp](https://reddit.com/r/mcp)
