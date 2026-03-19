# Return Semantic Identifiers, Not Opaque UUIDs

Models reason in natural language. When you return `"id": "550e8400-e29b-41d4-a716-446655440000"`, the model can't reason about it, can't remember it across tool calls, and will likely hallucinate it when asked to pass it to another tool.

**Bad:**
```json
{
  "project_id": "550e8400-e29b-41d4-a716-446655440000",
  "channel_id": "C04BKGH3L8P",
  "image_url": "https://cdn.example.com/avatars/256px/a7b3c9d1.jpg"
}
```

**Good:**
```json
{
  "project_name": "Q4 Marketing Campaign",
  "project_id": "q4-marketing",
  "channel_name": "#general",
  "profile_image": "Jane's avatar (square, professional headshot)"
}
```

**Practical approach:**
- Always include a human-readable name alongside any ID
- Use slug-style IDs when possible (`q4-marketing` vs UUID)
- Translate internal codes to descriptive labels
- If you must return UUIDs, pair them: `"project": {"id": "550e...", "name": "Q4 Marketing"}`
- For images, describe them textually rather than returning raw pixel-dimension URLs

**Why it matters:** Models retrieve and reason about natural language far better than opaque identifiers. When a subsequent tool call requires passing an ID, the model is more likely to use the correct one if it can associate it with a meaningful name.

**Source:** Anthropic Engineering Blog - "Writing effective tools for AI agents"; modelcontextprotocol.info writing effective tools tutorial
