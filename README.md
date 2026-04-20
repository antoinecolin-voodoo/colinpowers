# Colinpowers

Custom plugin for **Claude Code** and **Cursor**, based on [Superpowers](https://github.com/obra/superpowers), with tailored development workflow skills:

- **Dual-platform** — skills use Claude Code tool names as canonical vocabulary, with a reference file mapping them to Cursor's native tools. A single session-start hook emits the right JSON shape for whichever host is loading the plugin.
- **Linear integration** — issue tracking via MCP (status updates, backlog search, comments).
- **Branch management** — sub-feature branches off parent feature branches, squash-and-merge.
- **Three-tier model selection** — fast / standard / strong, mapped to the right model IDs on each platform.

## Installation

### Claude Code (local install)

```bash
mkdir -p ~/.claude/plugins/local
cp -R /path/to/colinpowers ~/.claude/plugins/local/colinpowers
```

Restart Claude Code or start a new session. The `SessionStart` hook loads `using-workflow` automatically.

Verify: in a new session, the `using-workflow` content should be injected as `hookSpecificOutput.additionalContext`.

### Cursor (local install)

```bash
mkdir -p ~/.cursor/plugins/local
cp -R /path/to/colinpowers ~/.cursor/plugins/local/colinpowers
```

Restart Cursor (or **Developer: Reload Window**). The plugin should appear under **Settings > Plugins**.

> **Note:** Cursor's plugin loader uses `fs.readdir` with `withFileTypes`, which returns `isDirectory() = false` for symlinks. A symlink will be silently skipped — use a real directory copy.
>
> To sync changes after editing the source repo:
>
> ```bash
> rm -rf ~/.cursor/plugins/local/colinpowers && cp -R /path/to/colinpowers ~/.cursor/plugins/local/colinpowers
> ```

### Team install (Cursor, Teams/Enterprise plan, untested)

Import the GitHub repo as a team marketplace via **Dashboard > Settings > Plugins > Team Marketplaces > Import**.

### MCPs and tools

- The plugin actively uses the Linear MCP. Install it from your host's MCP marketplace and authenticate.
- `ffmpeg` is useful for reading videos attached to Linear tickets: `brew install ffmpeg`.

## Usage

- Type `/using-workflow` in the first prompt of a conversation. (A session-start hook already injects it — this is mostly optional.)
- Works best with a **strong**-tier model (`claude-opus-4-7` on Claude Code, `claude-4.6-opus-high` on Cursor) as the orchestrator; subagents pick smaller tiers.
- Call individual skills directly when useful. `git-branch-workflow` and `requesting-code-review` are the most common stand-alone invocations.
- `qa-product-changelog` generates a QA/product-friendly changelog for a branch (with Linear links).

## Models

Three capability tiers (**fast / standard / strong**), mapped per platform. The authoritative table lives in [`skills/using-workflow/SKILL.md`](skills/using-workflow/SKILL.md#model-selection) — bump model IDs there.

## Skills (14)

| Skill                    | Description                                            |
| ------------------------ | ------------------------------------------------------ |
| `using-workflow`         | Entry point — routing, platform adaptation, models     |
| `brainstorming`          | Socratic design refinement before coding               |
| `writing-plans-lean`     | Reference-first plans — shorter, reviewable            |
| `executing-plans`        | Sequential plan execution with checkpoints             |
| `subagent-development`   | Subagent-per-task with two-stage review                |
| `dispatching-parallel`   | Run multiple agents concurrently                       |
| `git-branch-workflow`    | Branch creation, squash, merge/PR, cleanup             |
| `linear-integration`     | Issue status, backlog search, comments (via Linear MCP)|
| `requesting-code-review` | Pre-merge code review                                  |
| `receiving-code-review`  | Responding to review feedback                          |
| `systematic-debugging`   | 4-phase root cause analysis                            |
| `verification`           | Verify before claiming done                            |
| `qa-product-changelog`   | Branch changelog for QA / product, linked to Linear    |
| `writing-skills`         | Meta-skill for creating new skills                     |

## Platform adaptation

Skill prose uses Claude Code tool names as canonical (`Agent`, `Bash`, `Edit`, `Read`, `Write`, `Grep`, `Glob`, `TodoWrite`, direct `mcp__…__tool` MCP calls). When a skill mentions a tool you don't have in Cursor, translate via [`skills/using-workflow/references/cursor-tools.md`](skills/using-workflow/references/cursor-tools.md):

- `Agent` → `Task`
- `Bash` → `Shell`
- `Edit` → `StrReplace`
- direct MCP call → `CallMcpTool { server, toolName, arguments }`
- …etc.

The LLM usually does this mapping automatically; the reference file is there for ambiguous cases.

## Typical workflow

```
1. User starts task (with or without Linear issue)
2. linear-integration: fetch/search issue → set "In Progress"
3. brainstorming: design conversation → spec
4. git-branch-workflow (start): create sub-feature branch
5. writing-plans-lean: implementation plan
6. subagent-development: implement tasks with review
7. verification: confirm everything works
8. git-branch-workflow (finish): spec cleanup → squash → merge/PR
9. linear-integration: set "In Review"
```
