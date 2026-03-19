# Use Semantic Search with Embeddings for 100+ Tools

For very large tool catalogs, replace hierarchical navigation with a single `find_tools` meta-tool that uses embedding similarity to find relevant tools from a natural language query.

```python
from sentence_transformers import SentenceTransformer
import faiss

class SemanticToolRouter:
    def __init__(self, tools):
        self.model = SentenceTransformer('all-MiniLM-L6-v2')
        self.tools = tools
        texts = [f"{t.name}: {t.description}" for t in tools]
        embeddings = self.model.encode(texts)
        self.index = faiss.IndexFlatIP(embeddings.shape[1])
        self.index.add(embeddings)

    def search(self, query: str, top_k: int = 3) -> list:
        emb = self.model.encode([query])
        scores, idx = self.index.search(emb, top_k)
        return [self.tools[i] for i in idx[0]]

@tool
def find_tools(query: str) -> list[dict]:
    """Find tools matching a natural language intent.
    Example: find_tools('list recent HubSpot deals')"""
    matches = router.search(query, top_k=5)
    return [{"id": t.id, "name": t.name, "description": t.description} for t in matches]
```

**When to use semantic vs progressive:**
- **Semantic**: Faster for simple, single-intent queries. ~1.3k initial tokens + ~3k per task.
- **Progressive**: Better for complex multi-step workflows where the agent needs full visibility of available actions. ~2.5k initial + ~2.5k per task.

**Implementation tips:**
- Pre-compute embeddings offline; only query the index at runtime
- Cache recent query-to-tool mappings for 5-10 minutes
- Store embeddings in FAISS (in-memory) or Pinecone (distributed)
- Keep full JSON schemas in a separate KV store, not in the embedding index

**Source:** [Speakeasy — Comparing Progressive Discovery and Semantic Search](https://www.speakeasy.com/blog/100x-token-reduction-dynamic-toolsets); [Klavis AI — Less is More](https://www.klavis.ai/blog/less-is-more-mcp-design-patterns-for-ai-agents)
