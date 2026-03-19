# Use Unambiguous Parameter Names

Generic parameter names like `user`, `id`, or `data` force the model to infer meaning from context. Specific names like `user_id`, `project_name`, or `start_date` are self-documenting and reduce hallucination.

**Bad:**
```json
{
  "user": "string",
  "id": "string",
  "type": "string"
}
```

**Good:**
```json
{
  "user_email": "string - the email address of the user to look up",
  "project_id": "string - UUID of the project (use list_projects to find this)",
  "event_type": "string - one of: 'meeting', 'reminder', 'deadline'"
}
```

**Additional tips:**
- Use enums with `minimum`/`maximum` constraints to tighten validation
- Mark fields as `required` to prevent the model from guessing defaults
- Add descriptions to each parameter, not just the tool itself
- Reference other tools in parameter descriptions when values come from them (e.g., "use list_projects to get this ID")

**Why it matters:** Models treat parameter schemas as part of the prompt. Rich schemas with enums and descriptions act as inline documentation that reduces round-trips and retries by ~25%.

**Source:** modelcontextprotocol.info/docs/tutorials/writing-effective-tools/; NearForm - "Implementing MCP: Tips, tricks and pitfalls"
