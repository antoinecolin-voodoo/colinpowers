# Colinpowers

Custom Cursor plugin, based on [Superpowers](https://github.com/obra/superpowers), with tailored development workflow skills:

- **Cursor-native** — all instructions target Cursor's tool set (Task, Shell, Read, etc.)
- **Linear integration** — issue tracking via MCP (status updates, backlog search, comments)
- **Branch management** — sub-feature branches off parent feature branches, squash-and-merge
- **Three-tier model selection** — fast / standard / strong mapped to specific models

## Installation

### Local install (development / personal use)

Copy the repo into Cursor's local plugins directory:

```bash
mkdir -p ~/.cursor/plugins/local
cp -R /path/to/colinpowers ~/.cursor/plugins/local/colinpowers
```

Then restart Cursor (or run **Developer: Reload Window**).

Verify: the plugin should appear in **Settings > Plugins** under the installed section.

> **Note:** Cursor's plugin loader uses `fs.readdir` with `withFileTypes`, which returns `isDirectory() = false` for symlinks. A symlink will be silently skipped — use a real directory copy.
>
> To sync changes from the source repo after editing:
>
> ```bash
> rm -rf ~/.cursor/plugins/local/colinpowers && cp -R /path/to/colinpowers ~/.cursor/plugins/local/colinpowers
> ```

### Team install (requires Teams/Enterprise plan, untested)

Import the GitHub repo as a team marketplace via **Dashboard > Settings > Plugins > Team Marketplaces > Import**.

### MCPs and tools
- The plugin actively uses the Linear MCP. Please install it from Marketplace and AUTHENTICATE.
- FFMPEG can be useful to read videos in Linear tickets. You can install it with `brew install ffmpeg`.

## Usage

- Type `/using-workflow` in the first prompt of a conversation (I think this one is actually optional, there is a session-start hook).
- Works much better with `claude-4.6-opus-high` as initial model (the agent will then use smaller models for subagents).
- You can call individual skills from time to time. I've found that particularly useful for `git-branch-workflow` (sometimes the agent forgets about it) and `requesting-code-review` (sometimes you just want an adversarial code review and not the wole workflow).

## Models


| Alias        | Model                      | Use for                                                    |
| ------------ | -------------------------- | ---------------------------------------------------------- |
| **fast**     | `composer-2-fast`          | Mechanical tasks, single-file edits, search/explore        |
| **standard** | `claude-4.6-sonnet-medium` | Multi-file implementation, code review, debugging          |
| **strong**   | `claude-4.6-opus-high`     | Architecture, design review, complex debugging, escalation |


## Skills (13)


| Skill                    | Description                                 |
| ------------------------ | ------------------------------------------- |
| `using-workflow`         | Entry point — routes to other skills        |
| `brainstorming`          | Socratic design refinement before coding    |
| `writing-plans-lean`     | Reference-first plans — shorter, reviewable |
| `executing-plans`        | Sequential plan execution with checkpoints  |
| `subagent-development`   | Subagent-per-task with two-stage review     |
| `dispatching-parallel`   | Run multiple agents concurrently            |
| `git-branch-workflow`    | Branch creation, squash, merge/PR, cleanup  |
| `linear-integration`     | Issue status, backlog search, comments      |
| `requesting-code-review` | Pre-merge code review                       |
| `receiving-code-review`  | Responding to review feedback               |
| `systematic-debugging`   | 4-phase root cause analysis                 |
| `verification`           | Verify before claiming done                 |
| `writing-skills`         | Meta-skill for creating new skills          |


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

