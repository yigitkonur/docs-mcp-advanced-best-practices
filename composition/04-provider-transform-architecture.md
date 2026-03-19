# Use the Provider + Transform Architecture for Maximum Flexibility

FastMCP 3.0 introduces a composable architecture where Providers source components and Transforms modify their behavior. This is the most flexible way to build complex MCP servers.

**Core concepts:**

| Concept | Description | Example |
|---------|-------------|---------|
| **Component** | Atomic unit (Tool, Resource, Prompt) | A `search_contacts` tool |
| **Provider** | Sources components from anywhere | Decorators, filesystem, OpenAPI spec, remote MCP |
| **Transform** | Modifies Provider behavior | Rename, namespace, filter, gate, version |
| **Composition** | Combine Providers + Transforms | Mount sub-servers, proxy remote tools |

**Provider types:**
```python
# Local functions
@tool
def my_tool(): ...

# From a directory of Python files
admin_provider = FileSystemProvider("./admin_tools", reload=True)

# From an OpenAPI spec
api_provider = OpenAPIProvider("https://api.example.com/openapi.json")

# From a remote MCP server
remote_provider = MCPClientProvider("https://remote-mcp.example.com")

# From instruction files (skills)
skills_provider = SkillsProvider("./skills/")
```

**Transform examples:**
```python
# Namespace all tools from a provider
mcp.mount(github_provider, prefix="github")
# Tools become: github_create_pr, github_list_repos, etc.

# Filter by version
mcp.add_transform(VersionFilter(select="latest"))

# Gate with authentication
mcp.add_transform(AuthGate(tags={"admin"}, scopes={"super-user"}))

# Rename for consistency
mcp.add_transform(RenameTransform({"old_name": "new_name"}))
```

**The Playbook pattern:** Compose Providers, Visibility, Auth, and Session State into multi-step workflows:
1. User authenticates
2. `unlock_admin_mode` updates session state
3. Admin tools become visible (Visibility Transform)
4. Subsequent calls use the newly available tools

This replaces ad-hoc glue code with declarative primitives.

**Source:** [FastMCP 3.0 blog](https://jlowin.dev/blog/fastmcp-3)
