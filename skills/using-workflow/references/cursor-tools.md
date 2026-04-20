# Cursor tool mapping

Skills in this plugin use Claude Code tool names as the canonical vocabulary. In Cursor, translate them to the native equivalents below. The LLM usually does this automatically, but this table makes the mapping explicit when a skill is unclear.

| Skill references (Claude Code name) | Cursor equivalent |
|--------------------------------------|--------------------|
| `Agent` (spawn subagent with a prompt) | `Task` |
| `Agent` with `subagent_type` | `Task` — the agent type maps to Cursor's named agents (see `agents/` directory) |
| `Bash` (run shell command) | `Shell` |
| `Edit` (exact-string replacement in file) | `StrReplace` |
| `Write` (create or overwrite file) | `Write` (same) |
| `Read` (read file contents) | `Read` (same) |
| `Grep` (regex content search) | `Grep` (same) or `SemanticSearch` for conceptual search |
| `Glob` (file-name pattern match) | `Glob` (same) |
| `TodoWrite` (task list) | Cursor's built-in plan/task tracker |
| Direct MCP tool call, e.g. `mcp__claude_ai_Linear__get_issue({...})` | `CallMcpTool { server: "user-Linear", toolName: "get_issue", arguments: {...} }` |

## MCP call shape

**Claude Code** exposes each MCP tool as a top-level callable:

```
mcp__claude_ai_Linear__get_issue({ id: "CUP-123" })
```

**Cursor** wraps every MCP call in a single `CallMcpTool` invocation:

```
CallMcpTool {
  server: "user-Linear",
  toolName: "get_issue",
  arguments: { id: "CUP-123" }
}
```

When a skill says "call the `get_issue` tool on the Linear MCP server", use whichever form your platform provides.

## MCP server naming

- **Claude Code**: the Linear MCP prefix in this environment is `mcp__claude_ai_Linear__…`.
- **Cursor**: the Linear MCP server is `user-Linear`.

The `user-Linear` identifier is Cursor-specific; do not type it into a Claude Code MCP call.
