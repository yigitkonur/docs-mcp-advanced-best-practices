# Guard Tools: Precondition Validation via Boolean Params

Add boolean precondition parameters to destructive tools. The agent must affirm these conditions before the server proceeds. Unlike confirmation dialogs (which just delay), guard params shift **responsibility** — the agent is declaring that preconditions have been verified, and the server can optionally validate the claim.

```typescript
server.tool(
  "delete_environment",
  "Permanently delete a deployment environment. Set tests_verified=true only after running validate_environment. Set backup_confirmed=true only after create_backup completes.",
  {
    environment_id: z.string(),
    tests_verified: z.boolean()
      .describe("Set true only after validate_environment returned no critical errors."),
    backup_confirmed: z.boolean()
      .describe("Set true only after create_backup returned a valid backup_id."),
    backup_id: z.string().optional()
      .describe("The backup_id from create_backup. Required when backup_confirmed=true."),
  },
  async ({ environment_id, tests_verified, backup_confirmed, backup_id }) => {
    if (!tests_verified) {
      return {
        content: [{ type: "text", text: "Blocked: tests_verified is false. Run validate_environment first and confirm no critical errors." }],
        isError: true,
      };
    }
    if (!backup_confirmed || !backup_id) {
      return {
        content: [{ type: "text", text: "Blocked: backup_confirmed is false or backup_id is missing. Run create_backup first." }],
        isError: true,
      };
    }
    // Optionally: cryptographically verify the backup_id is genuine
    const backupValid = await verifyBackupExists(backup_id);
    if (!backupValid) {
      return {
        content: [{ type: "text", text: `Blocked: backup_id "${backup_id}" not found or expired. Re-run create_backup.` }],
        isError: true,
      };
    }

    await deleteEnvironment(environment_id);
    return { content: [{ type: "text", text: `Environment ${environment_id} deleted.` }] };
  }
);
```

## Layered Trust Model

- **Soft guard** — boolean params in schema; agent self-reports; server accepts on good faith
- **Medium guard** — server checks a signed token or ID from a prerequisite tool (see `01-parameter-dependency-chain`)
- **Hard guard** — server independently re-runs the precondition check itself before executing

Choose the layer based on the stakes: a destructive cloud operation warrants a hard guard; a local file operation may only need soft.

**Why it matters:** Boolean guard params force the agent to explicitly acknowledge preconditions rather than rushing through a workflow. They produce audit-log evidence of what the agent claimed was true before a destructive action.

**Source:** Safety patterns discussed in [r/mcp](https://reddit.com/r/mcp); defense-in-depth principles for agentic tool execution
