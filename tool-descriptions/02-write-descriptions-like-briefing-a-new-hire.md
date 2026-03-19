# Write Tool Descriptions Like Briefing a New Hire

The model has zero institutional knowledge. Every conversation starts fresh with only tool descriptions to guide it. Write descriptions as if explaining a tool to a competent new hire on their first day:

- State the tool's **single, clear purpose** explicitly
- Define any domain-specific terminology
- Make implicit conventions explicit (e.g., "dates are in YYYY-MM-DD format")
- Include a brief usage example showing expected input/output
- Specify what the tool does NOT do to prevent misuse

```python
@tool(
    name="search_contacts",
    description="""Search the CRM for contacts matching criteria.

    Returns contact records with name, email, and company.
    Use this when the user asks to find, look up, or search for people.
    Do NOT use this for updating contacts - use update_contact instead.

    Example: search_contacts(query="Jane at Acme") returns matching contacts.
    Supports partial name matching and company name filtering."""
)
```

**Why it matters:** Small, measured refinements to tool descriptions have shown large accuracy gains. Anthropic's Claude Sonnet 3.5 saw significant SWE-bench score improvements from description quality alone.

**Source:** Anthropic Engineering Blog - "Writing effective tools for AI agents" (Sep 2025); modelcontextprotocol.info/docs/tutorials/writing-effective-tools/
