# Be a Smart Database, Not a Smart Analyst

Your MCP server should provide rich, structured data and let the LLM do the analysis. Don't try to be clever on the server side with keyword matching or opaque scoring algorithms.

**Wrong mental model: "Smart Analyst"**
```python
def analyze_threats(description: str):
    threats = []
    if "payment" in description:  # Brittle keyword matching
        threats.append("Payment fraud")
    if "user" in description:
        threats.append("Identity spoofing")
    return {"threats": threats[:5]}  # Artificial limit for "readability"
```

**Right mental model: "Smart Database"**
```python
def get_threat_framework(app_description: str):
    return {
        "stride_categories": {
            "spoofing": {
                "description": "Identity spoofing attacks",
                "traditional_threats": ["User impersonation", "Credential theft"],
                "ai_ml_threats": ["Deepfake attacks", "Prompt injection"],
                "mitigation_patterns": ["MFA", "Certificate-based auth"],
                "indicators": ["login", "auth", "session", "token", "identity"]
            },
            # ... all categories with COMPLETE data
        },
        "context_analysis": analyze_app_context(app_description),
        "report_template": "Use the above framework data to generate a threat report."
    }
```

**Key principles:**
1. **Information Provider, Not Analyzer** - Supply the framework, let the LLM apply it
2. **Completeness Over Convenience** - Return ALL data with metadata, not truncated slices
3. **Supply scaffolds, not conclusions** - Provide scoring criteria, not final scores
4. **Templates, not reports** - Return `report_template` + raw data, not a finished document

**Why it matters:** The LLM is better at semantic analysis than your keyword matcher. Your server is better at data retrieval and structured output than the LLM. Play to each side's strengths.

**Source:** [Matt Adams — MCP Server Design Principles](https://matt-adams.co.uk/2025/08/30/mcp-design-principles.html)
