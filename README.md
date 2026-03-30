# Colinpowers

Custom Cursor plugin with tailored development workflow skills:

- **Cursor-native** — all instructions target Cursor's tool set (Task, Shell, Read, etc.)
- **Linear integration** — issue tracking via MCP (status updates, backlog search, comments)
- **Branch management** — sub-feature branches off parent feature branches, squash-and-merge
- **Three-tier model selection** — fast / standard / strong mapped to specific models

## Installation

### Local install (development / personal use)

Symlink the repo into Cursor's local plugins directory:

```bash
mkdir -p ~/.cursor/plugins/local
ln -sf /path/to/colinpowers ~/.cursor/plugins/local/colinpowers
```

Then restart Cursor (or run **Developer: Reload Window**).

Verify: your skills should appear in **Settings > Rules** under the "Agent Decides" section.

### Team install (requires Teams/Enterprise plan)

Import the GitHub repo as a team marketplace via **Dashboard > Settings > Plugins > Team Marketplaces > Import**.

## Models

| Alias | Model | Use for |
|-------|-------|---------|
| **fast** | `composer-2-fast` | Mechanical tasks, single-file edits, search/explore |
| **standard** | `claude-4.6-sonnet-medium` | Multi-file implementation, code review, debugging |
| **strong** | `claude-4.6-opus-high` | Architecture, design review, complex debugging, escalation |

## Skills (13)

| Skill | Description |
|-------|-------------|
| `using-workflow` | Entry point — routes to other skills |
| `brainstorming` | Socratic design refinement before coding |
| `writing-plans` | Detailed implementation plans from a design |
| `executing-plans` | Sequential plan execution with checkpoints |
| `subagent-development` | Subagent-per-task with two-stage review |
| `dispatching-parallel` | Run multiple agents concurrently |
| `git-branch-workflow` | Branch creation, squash, merge/PR, cleanup |
| `linear-integration` | Issue status, backlog search, comments |
| `requesting-code-review` | Pre-merge code review |
| `receiving-code-review` | Responding to review feedback |
| `systematic-debugging` | 4-phase root cause analysis |
| `verification` | Verify before claiming done |
| `writing-skills` | Meta-skill for creating new skills |

## Typical workflow

```
1. User starts task (with or without Linear issue)
2. linear-integration: fetch/search issue → set "In Progress"
3. brainstorming: design conversation → spec
4. git-branch-workflow (start): create sub-feature branch
5. writing-plans: implementation plan
6. subagent-development: implement tasks with review
7. verification: confirm everything works
8. git-branch-workflow (finish): spec cleanup → squash → merge/PR
9. linear-integration: set "In Review"
```
