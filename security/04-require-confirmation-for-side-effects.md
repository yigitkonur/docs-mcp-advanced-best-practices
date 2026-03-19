# Always Require Human Confirmation for Destructive Operations

Separate tools into read (auto-approve) and write (require confirmation) tiers. This is especially critical when tool responses contain user-generated content that could include prompt injection.

**Classification:**
```
ALWAYS AUTO-APPROVE (read-only):
  - search_*, list_*, get_*, describe_*
  - Any tool with readOnlyHint: true

ALWAYS REQUIRE CONFIRMATION:
  - delete_*, remove_*, drop_*
  - send_message, post_comment, create_*
  - Any tool that modifies external state
  - Any tool that processes user-generated input
```

**Server-side confirmation pattern:**
```python
@tool
def send_bulk_email(recipients: list[str], subject: str, body: str) -> dict:
    if len(recipients) > 10:
        return {
            "content": [{
                "type": "text",
                "text": (
                    f"About to send email to {len(recipients)} recipients.\n"
                    f"Subject: {subject}\n"
                    f"Please confirm this action with the user before proceeding.\n"
                    f"If confirmed, call send_bulk_email_confirmed(batch_id='...')"
                )
            }],
            "isError": False
        }
    # ... proceed with sending
```

**The escalation principle:** Read tools → always safe. Write tools → confirm if the data comes from external sources. Delete tools → always confirm regardless of source.

**Why not just trust the model?** Because prompt injection can make the model believe a destructive action is what the user wants. A human-in-the-loop is the only reliable defense for irreversible operations.

**Source:** [u/sjoti on r/mcp](https://reddit.com/r/mcp/comments/1lq69b3/) — "Tools that only read? Always allow. Tools that send data, update or delete them? Always have a human in there if it takes external data as input."
